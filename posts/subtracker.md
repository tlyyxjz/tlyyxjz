# I Was Bleeding $89 a Month on Subscriptions I Forgot I Had. Here's How I Stopped.

A few months ago, I sat down to look at my bank statement properly for the first time in... I don't even want to admit how long. I was hunting for a specific charge I didn't recognize — turned out to be a legit purchase I'd forgotten about — but what I found instead was worse.

Twelve subscriptions. Twelve.

There was the obvious stuff: Netflix, Spotify, the Adobe Creative Cloud plan I actually need for client work. Those I expected. But then there was the rest of the list. A meditation app I signed up for during a brutal deadline week last March and opened maybe twice. A VPN I bought for an overseas trip and never canceled. A news subscription I started to read one paywalled article. A workout app that seemed like a great idea in January and died by February. A cloud storage upgrade I'd forgotten I already had covered by another service. And — this one genuinely annoyed me — two different AI writing assistants, because I'd started a free trial on one, forgotten about it, then paid for the other one a month later.

Added up, it was $89 a month. Over a thousand dollars a year. On stuff I wasn't using.

The worst part? I consider myself reasonably careful with money. I track my spending. I have a budget. I read personal finance blogs unironically. And I *still* missed this.

## The subscription creep problem

If you're reading this, you probably already know the feeling. Subscriptions have quietly taken over how we pay for basically everything. Software went from something you buy once to something you rent forever. Media, tools, services, even hardware features — all behind monthly payments now.

And here's the thing about monthly payments: they're designed to be invisible. Ten dollars a month sounds like nothing. Twelve ten-dollar charges a month still sounds like nothing, each one individually. But collectively? That's a car payment. That's groceries for a week. That's a chunk of rent.

The numbers back this up, and they're genuinely startling. The average American now spends somewhere around $273 per month on subscription services. That's not a typo. Two hundred and seventy-three dollars, every single month, recurring. And here's the part that actually stings: industry estimates suggest people only get real value out of maybe $100 worth of those subscriptions. The rest is just leakage. Money draining out of accounts for services that have been forgotten, underused, or kept around "just in case I need it someday."

That gap — between $273 spent and $100 actually used — is what people in the industry call "subscription creep." It's the slow accumulation of recurring charges that happens because starting a subscription takes about thirty seconds and a fingerprint, while canceling one takes a support ticket, three confirmation screens, and sometimes an actual phone call. The entire ecosystem is optimized to make paying frictionless and stopping feel like work.

It's not that subscriptions are inherently bad. I happily pay for the ones I use. The problem is the asymmetry. The model makes it effortless to start paying and deliberately friction-filled to stop. Free trials convert automatically at 2am. Cancel buttons get buried five menus deep behind "are you sure?" and "how about a discount instead?" prompts. Some services make you call a phone number during business hours to cancel, like it's 2003 and we're all still using landlines.

So the real question becomes: how do you actually get a handle on this stuff?

## Why the existing solutions didn't work for me

I tried the obvious things first. None of them stuck.

**Bank apps.** I use a major bank, and their app does show transactions. But that's all it shows — a flat list of charges with no real intelligence behind it. There's no "this renews in 3 days" warning. No clear separation of recurring vs. one-time charges. No way to see what's coming up next month before it hits. You can scroll through your history and manually spot the patterns, sure, but that's not a solution, that's homework. And by the time you see the charge in your bank app, the money's already gone. It's a receipt, not a warning.

**Subscription management apps.** This is where Rocket Money and the rest of the category live. And look — they work, sort of. The core idea is solid: connect your accounts, let the app detect recurring charges, get alerts before renewals hit, sometimes even get help canceling. The problem is the business model. Rocket Money charges somewhere around $8 to $12 a month for the premium features, which include the cancellation assistance and renewal alerts — you know, the stuff that's actually useful. The free tier is basically just a read-only list.

There's something deeply ironic about paying a monthly subscription to manage your monthly subscriptions. If the entire goal is to stop bleeding money on recurring charges, adding *another* recurring charge feels like missing the point in a way I can't quite get past. Plus, you're handing your banking credentials to a third-party fintech startup and hoping they stay solvent and security-conscious indefinitely. It works for a lot of people. I just couldn't make peace with it.

**Spreadsheets.** I'm a developer. I've made spreadsheets for everything in my life — workouts, reading lists, coffee bean inventory (don't judge). So naturally I tried tracking subscriptions in Google Sheets. I set up columns, made it look nice, added formulas to calculate monthly and annual totals, even conditional formatting. It worked great for about two weeks. Then I forgot to update it when I started a new trial. Then I forgot again when a renewal came through. Within a month, the sheet was out of date, I'd stopped trusting it, and once you stop trusting a tracker you stop using it — which means it's worse than nothing, because it gives you a false sense of control.

The core issue with all three approaches: none of them *actively interrupt* you. A bank app is reactive. A spreadsheet requires you to remember to maintain it. Even the paid apps mostly just send a passive push notification that's easy to swipe away while you're standing in line for coffee. What I actually wanted was something that would sit there quietly, know my subscriptions, calculate upcoming renewals, and tell me — clearly and in advance — what was about to charge me, so I could make a real decision instead of a reactive one.

## So I built my own

I'm a developer, so my answer to most problems is "I'll just build it myself." Maybe that's not the healthiest instinct, but in this case it actually paid off.

The goals were simple. I wanted a tool that knew all my subscriptions — name, cost, billing cycle, next renewal date, category. Something that showed me my total monthly spend at a glance, broken down by category. Something that warned me before renewals hit, with enough lead time to actually cancel. Something that flagged subscriptions I might want to cancel based on cost, redundancy, or lack of use. And — critically — something that didn't cost me a monthly fee to use.

That last point is the whole ballgame. The entire reason for building this was to stop paying monthly fees for things. So the tracker itself had to be a one-time setup, or free, or self-hosted. I went with local: a Python script running on my own machine, data stored in a local file, no accounts, no cloud, no subscription to manage my subscriptions.

Why local instead of a web service? A few reasons that actually matter. Privacy — my subscription data is surprisingly personal; it reveals what media I consume, what tools I rely on, what I spend on entertainment vs. productivity. I don't need that sitting on someone else's server, getting bundled into some analytics dashboard. Control — I don't want my tracker to disappear because a startup pivoted or ran out of funding, which has happened to me before with a budgeting app I actually liked. Cost — once it's built, it costs nothing to run, forever. And flexibility — if I want to add a weird feature, like exporting renewals to my calendar or integrating with my task manager, I just do it. No API limits, no permission, no waiting on a roadmap.

## The data model

Before writing any code, I had to figure out what I actually needed to track per subscription. After a couple of iterations — I over-engineered the first version, naturally — this is what I landed on.

The core fields are straightforward: a service name, the amount, the billing cycle (monthly, yearly, or weekly — that's enough granularity), the next renewal date, whether it's currently active, and a category. Categories matter way more than you'd expect. Being able to see "I spend $40/month on streaming" versus "I spend $15/month on productivity tools" genuinely changes how you think about where to cut. It turns a pile of charges into a picture of your spending.

I also added a couple of optional fields that turned out to be more useful than the core ones. A notes field for context (things like "only use during NBA season" or "shared with sibling, split the cost"). And a "last used" date that I update manually when I actually open the app or log into the service. That last field is the single most useful thing for making cancellation decisions — if I haven't touched something in 60 days, it's probably time to let it go, no matter how cheap it is. Cheap and unused is still wasted money.

For storage, I went with SQLite. JSON would've worked fine and been simpler to set up, but SQLite gives me proper querying — I can ask "show me all yearly subscriptions renewing in the next 30 days" with a single SQL statement instead of filtering a list in Python. The database is a single file, so it's still trivially portable and easy to back up (I just drop it in my cloud sync folder). Honestly, for a dataset this small — maybe 15 rows — the choice barely matters. Use whatever you're comfortable with. The point is the data model, not the storage engine.

## Renewal date logic, and why it's trickier than it sounds

This is the part that sounds trivial but has real edge cases. The basic idea: given a subscription's next renewal date and billing cycle, figure out when the one after that is. Then compare to today to see how many days until renewal.

Monthly is the sneaky one. "Monthly" doesn't mean "every 30 days" — it means "same day of the month, each month." So if you renew on the 31st of January, your next renewal is the 28th (or 29th) of February, then the 31st of March, and so on. If you naively add 30 days each time, your reminders drift, and once reminders drift, you stop trusting them, and then you're right back to the spreadsheet problem where the tool is worse than nothing.

I handle this by storing the next renewal date explicitly and updating it when a renewal passes, rather than trying to compute everything from an original start date. When a renewal happens (or when I manually mark it as renewed), I add one billing cycle to the next-renewal date. For monthly, that's "same day next month, clamped to the last day of the month if the next month is shorter." For yearly, same logic but adding a year. It's maybe fifteen lines of Python using the `dateutil` library, and it's correct precisely because it doesn't try to be clever.

## The interface and reminders

I went with a simple command-line interface. I know that's not for everyone — most people don't want to live in a terminal — but I do, and it meant I could build the whole thing in an evening instead of a weekend. The commands are straightforward: add a subscription, list them all, see what's renewing this week, get a monthly summary, mark something as used. No UI framework, no CSS, no deployment, no auth.

The reminders were the part I actually cared about, the whole reason for building this. I set up a simple scheduled task (cron on my Mac, Task Scheduler works fine on Windows) that runs the tracker every morning and checks for subscriptions renewing in the next 7 days. If it finds any, it sends me a desktop notification with the name, amount, and days until renewal. Something like: "Notion — $8 — renews in 3 days." That's all I need. Seven days gives me enough time to actually decide and cancel if I want to, instead of finding out the morning of, when it's already too late.

The monthly summary is the other feature I use constantly. Once a month, I get a breakdown: total spend, spend by category, list of everything that renewed that month, and anything I'd marked as unused in the last 30 days. That last category is my cancel list. If something shows up there two months in a row, it's gone. No deliberation. Two strikes and you're out.

## What actually happened when I used it

I've been running this for three months now. Here's what changed.

The first month was mostly discovery — adding everything I'd missed, which took an afternoon of going through bank statements line by line. The total came to $89/month across 12 subscriptions. That was higher than I expected, and somehow lower than the national average, which made me feel simultaneously bad about myself and okay about the broader population.

Month one, I canceled four things immediately: the duplicate AI tool (kept the better one), the meditation app, the VPN I wasn't using, and a stock photo subscription I'd genuinely forgotten existed. That dropped me to around $54/month. Felt good. Felt obvious in retrospect.

Month two, the reminders caught two renewals I would've missed entirely. One was a yearly subscription to a design tool ($120/year, so $10/month equivalent) that I'd forgotten was even on a yearly cycle. The other was a monthly news subscription I'd stopped reading weeks ago but kept meaning to cancel. The news one I killed on the spot — the reminder gave me the prompt I needed. The design tool I kept, because the reminder made me actually stop and check whether I still use it, and I do. That's the thing about reminders: they don't force you to cancel, they force you to *decide*. Most of the time, the decision is the hard part, not the cancellation.

Month three, I was down to $34/month. I'd cut my subscription spending by more than 60%. The remaining ones — Netflix, Spotify, Adobe, Notion, my main cloud storage, a password manager, and a couple of others — are all things I actively use and get real value from. The difference is that now I *know* about them, I've *actively decided* to keep them, and I get a heads-up before every single renewal so I can reconsider without forgetting.

$55 a month doesn't sound like a lot in isolation. I get that. But that's $660 a year. Over a few years, with compounding, it's real money — the kind of money that could go into savings, or a trip, or an emergency fund, or literally anything other than a meditation app I used twice and a VPN I used for one trip.

## If you want to do this yourself

You can absolutely build this from scratch — the core logic is genuinely simple, and if you're comfortable with Python, you could have a working version in a weekend. The data model is small, the renewal math is manageable with a good date library, and a CLI gets you most of the way to something useful. There's no magic here, just a few careful decisions about what to track and how to remind yourself.

But if you'd rather not start from zero — if you'd rather just have the thing that works and spend your evening doing something else — I packaged up the version I've been using into something you can grab and run today. It's called SubTracker. Local first, one-time purchase, no subscription (obviously), no account, no cloud. Your data stays on your machine. You get the full data model, the renewal logic that actually handles edge cases correctly, the reminder system, the monthly summaries, the category breakdowns, the cancel suggestions. And since the code is right there, you can customize it however you want — add fields, change reminder timing, swap the CLI for a web UI, whatever fits your life.

It's pay-what-you-want starting at $5. Which is less than a month of one of the subscriptions you're probably about to cancel. You can grab it here: https://8837207235283.gumroad.com/l/uackeo

Whether you build your own, use mine, or just go manually audit your bank statement tonight — do something. Stop paying for things you forgot you signed up for. Your future self will thank you, and your bank account definitely will.
