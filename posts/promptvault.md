# I Built a Chrome Extension to Manage 200+ AI Prompts — Here's What Actually Works

Last March I caught myself doing something embarrassing. I had a Notion doc called "good prompts," a TextExpander snippet group I'd abandoned in 2022, roughly forty starred tweets, three Stickies on my desktop, and a Google Doc shared with a coworker who kept editing it without telling me. All of them held prompts I actually used. None of them talked to each other.

The breaking point came on a Tuesday. I was trying to find the prompt I use for cleaning up messy transcripts — the one with the specific "keep contractions, strip filler, don't summarize" instruction I'd spent an afternoon tuning. I knew it was *somewhere*. Twenty minutes of searching later, I'd opened four apps and still couldn't find the version with the right wording. I gave up, rewrote it from memory, and it came out worse.

That night I started sketching what would eventually become a Chrome extension. This post is the honest version of what I learned building it — the architecture that survived, the parts I threw away, and the bugs that almost made me quit.

## Why a browser extension, specifically

I considered a standalone app. I considered a Raycast/Alfred workflow. I even considered just paying for one of those prompt-marketplace subscriptions. None of them solved the actual problem, which is this: **by the time I want a prompt, I'm already inside ChatGPT, or Claude, or Gemini.** The friction isn't storing the prompt. The friction is the round-trip — switch app, find prompt, copy, switch back, paste, reformat because the target box handles newlines weirdly.

A Chrome extension is the only thing that lives in the same process as the page you're typing into. It can read the DOM, it can inject text, it can listen for a keyboard shortcut and respond in under 200ms. A native app can't do that without a browser-native-messaging dance that nobody wants to maintain.

So. Extension it was.

## The architecture, in plain terms

If you've never written one, a Manifest V3 extension is basically four pieces that talk through message-passing:

- **manifest.json** — the config file. Tells Chrome what permissions you need, what scripts to load, where your popup lives.
- **Background service worker** — a JavaScript file that runs in the background, except in MV3 it doesn't really "run in the background," it gets woken up when an event fires and killed when it goes idle. This bit me. More on that later.
- **Content scripts** — JavaScript that gets injected into actual web pages. This is where you touch the DOM of chatgpt.com or claude.ai.
- **Popup** — the little HTML page that shows when you click the toolbar icon. Where users search and manage prompts.

Plus **chrome.storage**, which is the persistence layer. There's `chrome.storage.local` (stays on this machine, ~10MB quota) and `chrome.storage.sync` (synced across your Chrome instances, ~100KB total, 8KB per item). For a prompt manager, sync sounds perfect — until you do the math and realize 100KB is maybe 15-20 long prompts if you're storing metadata too. I use both: sync for the top-level structure and a small "favorites" set, local for the full library.

Here's the manifest I ended up with. Stripped to the relevant parts:

```json
{
  "manifest_version": 3,
  "name": "PromptVault",
  "version": "1.4.2",
  "permissions": ["storage", "activeTab", "contextMenus", "commands", "scripting"],
  "host_permissions": [
    "https://chatgpt.com/*",
    "https://claude.ai/*",
    "https://gemini.google.com/*",
    "https://*.perplexity.ai/*"
  ],
  "background": {
    "service_worker": "background.js",
    "type": "module"
  },
  "action": {
    "default_popup": "popup.html"
  },
  "content_scripts": [
    {
      "matches": ["https://chatgpt.com/*", "https://claude.ai/*", "https://gemini.google.com/*"],
      "js": ["content.js"],
      "run_at": "document_idle"
    }
  ],
  "commands": {
    "open-prompt-search": {
      "suggested_key": { "default": "Ctrl+Shift+P", "mac": "Command+Shift+P" },
      "description": "Search and insert a prompt"
    },
    "insert-last-used": {
      "suggested_key": { "default": "Ctrl+Shift+L", "mac": "Command+Shift+L" },
      "description": "Insert the last used prompt"
    }
  }
}
```

A couple of decisions worth pointing out. I scoped `host_permissions` to the specific AI sites instead of `<all_urls>` — Chrome's review process is friendlier, and users get a cleaner install dialog. I also use `document_idle` for content script injection because injecting at `document_start` means the page's React tree isn't there yet, and at `document_end` you race with hydration. `document_idle` is the boring correct choice.

## Storage: the part I got wrong twice

My first version stored everything in `chrome.storage.sync`. It felt elegant. Open the extension on a different machine, your prompts are just *there*. Then a user reported their prompts were silently disappearing. Turned out they had ~240 prompts and Chrome was rejecting writes past the quota — and `chrome.storage` doesn't throw on quota errors by default in some versions. It just... doesn't save.

The wrapper I use now handles this properly:

```js
// storage.js — shared between background and popup
const SYNC_BUDGET = 90_000; // leave headroom under the 102,400 byte cap
const SYNC_KEY = 'pv_meta'; // small index + favorites only
const LOCAL_KEY = 'pv_library'; // full prompt bodies

export async function savePrompt(prompt) {
  // Always write the full body to local storage.
  const local = await chrome.storage.local.get(LOCAL_KEY);
  const library = local[LOCAL_KEY] || [];
  const idx = library.findIndex(p => p.id === prompt.id);
  if (idx >= 0) library[idx] = prompt;
  else library.push(prompt);
  await chrome.storage.local.set({ [LOCAL_KEY]: library });

  // Mirror a lightweight index to sync so other machines know what exists.
  const meta = (await chrome.storage.sync.get(SYNC_KEY))[SYNC_KEY] || { ids: [], favorites: [] };
  if (!meta.ids.includes(prompt.id)) meta.ids.push(prompt.id);
  try {
    await chrome.storage.sync.set({ [SYNC_KEY]: meta });
  } catch (e) {
    // Quota exceeded — degrade gracefully, local copy is the source of truth.
    console.warn('sync mirror failed, continuing with local only', e);
  }
}

export async function searchPrompts(query) {
  const local = await chrome.storage.local.get(LOCAL_KEY);
  const library = local[LOCAL_KEY] || [];
  if (!query) return library;
  const q = query.toLowerCase();
  return library.filter(p =>
    p.title.toLowerCase().includes(q) ||
    p.tags?.some(t => t.toLowerCase().includes(q)) ||
    p.body.toLowerCase().includes(q)
  );
}
```

The lesson: treat sync as a *hint*, not a guarantee. Local storage is the source of truth. Sync is a convenience that may fail, and your code needs to be okay with that.

## Injecting text into the AI sites — the part nobody warns you about

Here's where things get genuinely annoying. Each AI site handles its input differently, and they change without warning.

ChatGPT uses a `contenteditable` div. Claude uses a `ProseMirror` editor. Gemini uses a `contenteditable` too but with a different structure. You can't just `element.value = text` like it's a textarea. You need to dispatch input events that React (or whatever framework) will actually pick up.

Here's a simplified version of the injection logic:

```js
// content.js
function getEditor() {
  // ChatGPT
  const gpt = document.querySelector('#prompt-textarea');
  if (gpt) return { el: gpt, type: 'gpt' };

  // Claude — ProseMirror
  const claude = document.querySelector('.ProseMirror');
  if (claude) return { el: claude, type: 'claude' };

  // Gemini
  const gemini = document.querySelector('div[contenteditable="true"]');
  if (gemini) return { el: gemini, type: 'gemini' };

  return null;
}

function insertText(text) {
  const editor = getEditor();
  if (!editor) {
    showToast('No supported input found on this page');
    return;
  }
  const { el, type } = editor;

  el.focus();

  if (type === 'gpt' || type === 'gemini') {
    // execCommand is deprecated but still the most reliable way to
    // get React-controlled contenteditables to register the change.
    document.execCommand('insertText', false, text);
  } else if (type === 'claude') {
    // ProseMirror needs the proper input event or it eats newlines.
    const dt = new DataTransfer();
    dt.setData('text/plain', text);
    const ev = new InputEvent('beforeinput', {
      inputType: 'insertFromPaste',
      data: text,
      dataTransfer: dt,
      bubbles: true,
      cancelable: true,
    });
    el.dispatchEvent(ev);
  }

  // Nudge the framework to update its internal state.
  el.dispatchEvent(new Event('input', { bubbles: true }));
}

chrome.runtime.onMessage.addListener((msg) => {
  if (msg.type === 'insert-prompt') insertText(msg.text);
});
```

That `execCommand` call will make some people roll their eyes — it's deprecated, sure. But I tried `document.execCommand`, manual `Selection`/`Range` manipulation, `ClipboardEvent` with `DataTransfer`, and `InputEvent('insertText', ...)`. The deprecated API was the only one that worked consistently across all three sites. Sometimes the boring answer is the right one.

The Claude path took me an entire weekend. ProseMirror strips newlines from anything except `insertFromPaste` events with a proper `DataTransfer`. I found that buried in a GitHub issue from 2022. You're welcome.

## Keyboard shortcuts and the right-click menu

These are the two insertion paths users actually use. The popup is mostly for editing.

Shortcuts come through the `chrome.commands` API in the background worker:

```js
// background.js
chrome.commands.onCommand.addListener(async (command) => {
  if (command === 'insert-last-used') {
    const { lastPromptId } = await chrome.storage.local.get('lastPromptId');
    if (!lastPromptId) return;
    const { pv_library = [] } = await chrome.storage.local.get('pv_library');
    const prompt = pv_library.find(p => p.id === lastPromptId);
    if (!prompt) return;
    const [tab] = await chrome.tabs.query({ active: true, currentWindow: true });
    chrome.tabs.sendMessage(tab.id, { type: 'insert-prompt', text: prompt.body });
  }

  if (command === 'open-prompt-search') {
    chrome.action.openPopup?.(); // only works if user invoked via toolbar
    // Fallback: send a message to content script to open an in-page search overlay.
    const [tab] = await chrome.tabs.query({ active: true, currentWindow: true });
    chrome.tabs.sendMessage(tab.id, { type: 'open-overlay' });
  }
});
```

One gotcha: `chrome.action.openPopup()` only works if the user invoked the extension via the toolbar icon within the last few seconds. So for the keyboard shortcut, I built an in-page overlay that the content script renders. That's the thing that actually gets used 90% of the time.

The right-click menu is set up once, on install:

```js
chrome.runtime.onInstalled.addListener(() => {
  chrome.contextMenus.create({
    id: 'insert-prompt',
    title: 'Insert PromptVault prompt',
    contexts: ['editable']
  });
});

chrome.contextMenus.onClicked.addListener(async (info, tab) => {
  if (info.menuItemId === 'insert-prompt') {
    chrome.tabs.sendMessage(tab.id, { type: 'open-overlay' });
  }
});
```

I scope it to `editable` contexts so it only shows up when the user right-clicks inside an input. Don't be the extension that adds a context menu item to every image on the internet.

## The four bugs that almost ended me

**1. Content Security Policy on claude.ai.** Claude's CSP blocks inline scripts and remote imports. My first overlay injected a `<style>` tag with inline content — worked on ChatGPT, got silently blocked on Claude. Fix: ship all CSS as a separate file declared in the manifest, never inline.

**2. The service worker dying mid-operation.** This is the big MV3 headache. Service workers get killed after ~30 seconds of inactivity. I had a flow where the popup sent a message to the background, the background fetched something, and by the time the response came back the worker was gone. Fix: don't use the background as a middleman for things the popup can do directly. Move storage reads into the popup. The background should be dumb.

**3. Cross-origin XHR from content scripts.** I wanted to fetch prompt suggestions from a server. Content scripts inherit the page's origin, not the extension's, so requests to my API got blocked by CORS. Fix: do the fetch in the background worker (which has extension origin) and pass the result back. Or just don't have a server. I went with option two.

**4. Gemini changed their DOM.** One morning insertions stopped working on gemini.google.com. They'd swapped the `contenteditable` structure. No changelog, no announcement. Fix: query selectors should be as loose as possible — `div[contenteditable="true"]` beats `div.ql-editor.contenteditable`. And set up an automated test that opens each site once a day and tries an insertion. I learned that one the hard way.

## Performance: the numbers that mattered

Early on, opening the popup took about 800ms on a library of 200 prompts. That's because I was rendering every prompt into the DOM immediately and attaching event listeners to each one. Classic mistake.

Two changes got it to ~120ms:

- **Virtualize the list.** Only render the ~30 prompts visible in the scroll viewport. Use `IntersectionObserver` to load more as the user scrolls.
- **Index the search.** I build a simple in-memory trigram index on first load and cache it. Searching 200 prompts went from ~60ms (filter + lowercase) to ~4ms. Overkill for 200 prompts, but it scales to thousands without complaint.

The other win was lazy-loading the editor. The popup has two views: a list and a full editor. The editor pulls in a markdown renderer and a syntax highlighter. Don't load those until the user clicks "edit." Dynamic `import()` inside the click handler. Cuts initial popup weight by 40KB.

## What I'd tell someone starting this today

Don't reach for a framework in the popup. I tried React first; the bundle was 180KB before I'd written a line of my own code, and the popup mount added 200ms. Vanilla JS with a tiny render function is plenty for a UI this small. The fewer moving parts, the easier it is to debug when Chrome ships an update that breaks something.

Test on all three sites every week. They drift. Set up a Playwright script that loads each site, injects a test string, and screenshots the result. Five minutes to write, saves you a one-star review.

Expect the service worker lifecycle to surprise you. Anything stateful in the background will eventually be lost. If you find yourself storing state in a background variable, you've made a mistake — it should be in `chrome.storage`.

And be honest in your permissions. I considered requesting `<all_urls>` so the extension would work "anywhere." It would have. It also would have tanked my install rate because users (rightly) don't trust that. Scope to the sites you actually support. You can always add more later.

---

Building this thing took me about six weeks of evenings, and it's now the small tool I use more than any other. If you want to skip the six weeks — or just want a polished version with cloud sync, team sharing, and prompt versioning that I never managed to open-source properly — I packaged the whole thing up as **PromptVault**.

It's $15, one-time, no subscription. The same architecture I described above, plus the bits I didn't have room to cover here: a proper settings UI, CSV import/export, shared team collections via a link, and the in-page overlay that works across ChatGPT, Claude, Gemini, and Perplexity.

👉 **[PromptVault on Gumroad — $15](https://8837207235283.gumroad.com/l/eaqoat)**

If you found this writeup useful, the product is basically this post plus a year of polish. And if you'd rather build your own — the architecture above is real and it works. Either way, stop pasting prompts from a Notion doc. It's 2026.
