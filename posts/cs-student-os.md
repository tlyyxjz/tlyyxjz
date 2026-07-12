# How I Built a Notion OS for My CS Degree (GPA, LeetCode, Job Hunt, Everything)

Last October, I hit a wall. Not the productive kind where you're stuck on a bug and then have a breakthrough at 2am. The bad kind. The kind where you're sitting in the library with three tabs of LeetCode open, a problem set due at midnight, a career fair in two days you haven't prepped for, and you genuinely can't remember if you submitted that OS assignment or just thought about submitting it.

That was me. Brain fried. Notes scattered across Google Docs, Notion, random Markdown files, a physical notebook I hadn't opened since week two. My GPA was fine-ish, but only because I was burning myself out every week just to stay afloat. Something had to change.

I'm not going to pretend I had some elegant epiphany. What actually happened is I spent a weekend rage-building a Notion workspace because I was tired of losing things. That workspace turned into the system I still use today — something I ended up calling CS Student OS. This post is basically me walking through how it works, why I made the choices I made, and how you can build the same thing yourself.

## The problem: everything is happening at the same time

Here's the thing about being a CS student that non-CS people don't get. You're not just taking classes. You're taking classes *and* grinding LeetCode *and* applying to internships *and* building side projects *and* trying to have a social life so you don't go insane. These aren't separate tracks either — they bleed into each other constantly.

A recruiter emails you back about an interview next Tuesday. Cool. But next Tuesday is also when your algorithms midterm is, and you've got a systems project demo the day after, and you haven't touched a LeetCode problem in a week because you were busy with the project. So now you're prepping for the interview at 1am with stale knowledge, bombing easy questions you would've nailed a month ago.

Or this: you finish a problem set, feel good about it, then realize the next one was actually due yesterday. Because the deadline was buried in a syllabus PDF you downloaded once and never opened again.

Or the worst one — you do twenty LeetCode problems in a week, feel like a machine, then three weeks later you can't solve the same problem because you never reviewed any of them. You were "grinding" but not actually learning.

All of this is the same root problem: too many moving parts, no single source of truth. Your brain is not built to track forty deadlines and sixteen companies and a hundred LeetCode problems and five classes simultaneously. So stop trying.

## Why Notion (and not five different apps)

I tried a lot of things before landing on Notion. Trello for tasks. Google Calendar for deadlines. An Excel sheet for LeetCode (yes, really). A Notion doc for notes. Apple Notes for random stuff. Drive folders for assignments. It was a mess, and the mess was the problem — I'd put something in one place and forget to check it, or I'd have the same task duplicated across three apps with three different statuses.

Notion isn't perfect, but it solved the core issue: everything lives in one place, and the databases actually talk to each other. A LeetCode problem can link to the topic I'm weak on. An assignment can link to its course. A job application can link to the project I'm putting on my resume. You can't easily do that in a spreadsheet.

The other reasons, quickly:

- It's free for students. The Education plan is a no-brainer.
- It works on my phone, my laptop, my iPad. I can review flashcards on the bus.
- The database views are genuinely good. Kanban for the job hunt, calendar for deadlines, gallery for projects — same data, different lenses.
- I don't have to context-switch between six apps. One workspace. One search bar.

Okay, enough selling. Let me get into the actual system.

## The architecture: what's in the workspace

I'll walk through the five main pieces. If you want to build this yourself, this is the order I'd build them in.

### 1. Dashboard — the only page I actually open daily

The dashboard is the homepage. Everything else is one or two clicks away from here. The goal is: when I open Notion, I immediately see what matters today, without hunting.

Mine has a "Today" view — assignments due today or tomorrow, interviews scheduled, LeetCode problems queued for review. A "This Week" view for bigger picture stuff: project milestones, midterm dates, goal count for LeetCode problems. A GPA widget showing my current cumulative GPA and per-semester breakdown. And a quick-add section with template buttons for the most common things I create — new assignment, new LeetCode problem, new job app.

I cannot overstate how much the quick-add buttons matter. Friction kills systems. If adding a new assignment requires me to navigate to a database, click new, type four fields, set a relation — I won't do it. I'll just remember the deadline and forget it later. One button, pre-filled template, done.

### 2. Courses database — where the grades live

This is a database where each entry is a course. Fields: course name, course code, semester (a select property — Fall 2025, Spring 2026, etc.), professor, credits, current grade / expected grade, syllabus link, meeting time.

But the real magic is the relation. Each course is **related to** an Assignments database. So when I click into CS 351, I see every assignment, quiz, and exam for that course in one filtered view. No more digging through the syllabus. The assignments database has its own fields — due date, type (homework, exam, project), weight, status, grade received — and each one links back to its course.

The grade field is where Rollup comes in. I'll explain the GPA calculation below, but the short version: every assignment has a score and a weight, the course page rolls those up into a current grade, and the dashboard rolls all courses into a GPA. It updates automatically. I never calculate my GPA by hand anymore.

### 3. LeetCode tracker — the part I actually stuck with

I've tried tracking LeetCode three separate times in my life and always quit after a week. This version stuck, and I think it's because of the review system.

Fields: problem number, title, URL, difficulty (Easy / Medium / Hard), tags (arrays, DP, graphs, trees, two pointers — whatever), status (Solved / Reviewed / Needs review / Skipped), last solved date, next review date, time to solve (rough bucket: under 10 min, 10–30, 30+, gave up), my solution link + notes, confidence (1–5).

The critical field is "Next review date." It's a formula that takes "Last solved date" and adds an interval based on how many times I've reviewed it and my confidence score. If I solved it cold in 8 minutes with high confidence, I don't see it again for two weeks. If I struggled and gave up, I see it tomorrow. This is basically a lightweight spaced repetition system inside Notion, and it's the reason I actually retain what I grind instead of speed-running easy problems and forgetting them by Friday.

I have three views on this database. A **Kanban** by difficulty, for when I want to pick a problem by mood. A **Calendar** by next review date, so I can see what's due for review today. And a **table** grouped by tag, for when I'm targeting a weak area.

### 4. Job hunt pipeline — because applying is a numbers game and you need to track the numbers

Kanban view, columns: **Wishlist → Applied → OA → Phone screen → Onsite → Offer → Rejected**.

Fields per application: company, role (SWE intern, backend, frontend, etc.), status (drives the Kanban), date applied, next step + date (interview scheduled, follow-up, etc.), link to posting, recruiter contact if any, salary/hourly rate for when you actually get offers and need to compare, referral? (Y/N), resume version sent, and notes.

The notes field is huge. After every interview I write a debrief: what they asked, what I nailed, what I fumbled, what I should study before the next round. I used to walk out of interviews and immediately forget half the questions. Writing them down within an hour, while it's fresh, made me noticeably better at interviews over the semester. Patterns emerge — oh, every backend interview asks me about database indexing and I keep fumbling it. Cool, now I know what to study.

I also have a "Dead pipeline" view — applications with no response in 3+ weeks. Not because I'm giving up, but because it's realistic. Most applications ghost you. Seeing that visually keeps me from obsessing over any single one. You send fifty, you hear back from five, you interview with three, you get one offer. That's just how it works, and the numbers in the pipeline make that less depressing somehow.

### 5. Projects — the portfolio that actually stays updated

Every project gets an entry: name, description, tech stack (multi-select), GitHub link, live demo link if it exists, status (in progress / shipped / abandoned), and a "resume-ready" checkbox.

The trick here is the Gallery view with a cover image per project. It looks clean and it forces me to actually write a real description instead of "todo app #47." When a project has a nice cover and a one-paragraph description that reads well, I'm way more likely to put it on my resume. And when it's on my resume, I link it back to the job applications it's relevant to, via a relation.

This is the part of the system that pays off slow. You won't feel it week to week. But when you're three months in and someone asks "what have you built?", you have a real answer instead of a vague "uh, a bunch of stuff on my GitHub, let me find the link real quick."

## The Notion tricks that make it actually work

A workspace is just a bunch of pages until you wire it together. These are the features that turn it from "notes app" into "system."

**Rollups for GPA.** This is the one people ask about most. Set it up like this: Assignments database has a "grade" number property and a "weight" number property. The Courses database has a rollup that pulls in all related assignments' grades, weighted, and averages them into a course grade. The Dashboard can then roll up all courses into a GPA using credits as the weight. Once it's set up, you enter a grade once, in one place, and your GPA updates everywhere. Yes, the formula is a little fiddly to write the first time. Yes, it's worth it. I have not manually computed my GPA in two semesters and I do not miss it.

**Relations for everything.** Assignments relate to courses. LeetCode problems relate to topics. Job applications relate to projects (which one did I mention in this application?). Projects relate to courses (was this a class project or personal?). Once you have relations, you can build filtered views that show, say, "all assignments for CS 351 due this week" without manually filtering every time. The data stays connected, and connected data is what makes a system feel smart instead of like a spreadsheet.

**Formulas for spaced repetition.** My LeetCode "next review date" is a formula. Simplified version: if confidence is 5, add 14 days. If 4, add 7. If 3, add 3. If 2 or below, add 1. Stacked on top of review count, so the interval grows the more times I've seen it. You can get fancier with a real SM-2 algorithm, but honestly the simple version gets you 90% of the benefit. The point isn't algorithmic purity — it's that the problem shows up again before I forget it, and not so often that I'm wasting time on stuff I already know.

**Views, views, views.** Same database, different lens. The assignments database is a calendar when I need to see what's due, a Kanban by status when I'm in execution mode, and a table grouped by course when I'm calculating grades. Don't make three databases — make one and use views. This is the single biggest Notion lesson I learned. I used to duplicate data everywhere and then wonder why nothing matched.

**Template buttons.** This is the most underrated feature in Notion. A template button creates a pre-filled entry with one click. I have one for "new assignment" that pre-fills the status as "not started," links to the current semester, and sets a default weight. One click, type the name, done. Without these, the system collapses because I get lazy about adding things. Friction is the enemy. Every extra click is a chance for me to say "I'll add it later" and then not add it later.

## A week in the life

To make this concrete, here's how I actually used the system during a real week last spring. Monday of week 9. Midterm season. The bad part of the semester.

**Monday morning.** I open the dashboard. Today's view shows: CS 351 problem set due Wednesday (status: in progress), a LeetCode review due today (Medium, binary trees, last reviewed two weeks ago), and a phone screen with a fintech company on Thursday. This week view shows the algorithms midterm on Friday and a project demo for distributed systems on the following Monday.

I do the LeetCode review first because it's quick and gets me warmed up. I get it in twelve minutes, mark confidence at 4, next review bumps to two weeks out. Done. Move on to the pset.

**Tuesday.** Dashboard shows the pset is still due Wednesday. I work on it between classes. I also get an email from a recruiter at another company — I add it to the job pipeline as "applied" with the date. Takes ten seconds via the template button. I forget about it immediately, which is the point. It's tracked. I don't have to hold it in my head.

**Wednesday.** Pset submitted, status flipped to "done," grade field empty until I get it back. I spend the evening prepping for Thursday's phone screen — pulling up the company's entry in the pipeline, reviewing the JD, doing one warm-up problem. After the interview, I open the job app entry and write a debrief in the notes: "Asked about reversing a linked list in pairs, then a system design question about URL shortener. Nailed the first, fumbled the scale part of the second. Study: read up on sharding before next round." Status moves to "phone screen — awaiting decision."

**Thursday night / Friday.** Algorithms midterm. I'd been tracking upcoming exams in the calendar view for two weeks, so this wasn't a surprise — I'd carved out study time instead of cramming. I'd also been doing tagged LeetCode problems (DP, mostly) that I knew were weak areas, and I could see the review queue for those tags shrinking over the week. Take the midterm. Mark it done in the assignments database. Move on. Crash for nine hours.

**Weekend.** The distributed systems demo is Monday. The project entry shows it's "in progress" with the GitHub link, so I open the repo and polish the README, take a screenshot for the cover image, update the description so it reads clean. I also notice three LeetCode problems are due for review tomorrow, so I knock those out Sunday evening while watching something dumb on YouTube. Twenty minutes, no stress.

By the end of the week: one pset submitted, one midterm taken, one interview done and debriefed, one project demo prepped, six LeetCode problems reviewed or solved, one new job application added. Nothing got dropped. I slept like a human being on most nights. That's the system working. Not glamorous, just functional.

## What I'd tell my freshman-year self

You don't need a perfect system. You need one you'll actually use. The version of this workspace I started with was ugly and broken in a dozen ways — half the rollups didn't work, the LeetCode review formula was wrong, I had three databases that should've been one. It didn't matter. It still saved my ass because it was better than the alternative, which was "trust my brain to remember everything."

If you're starting from zero, build the minimum: a dashboard, a courses + assignments database with one relation, and a LeetCode tracker with a review formula. That's maybe two hours of setup, and it'll cover 80% of the value. Add the job pipeline and portfolio when you need them. Don't try to build the whole thing in one weekend — you'll burn out on Notion formulas before you ever use the system.

The point isn't the tool. The point is getting everything out of your head and into one place you trust, so your brain can stop being a filing cabinet and start being a brain. Use it for thinking, not for remembering deadlines.

---

Okay, pitch time — and I'll keep it short because I hate long sales sections too.

Everything I described above is exactly how my workspace is set up. But you don't have to build it from scratch. I packaged the whole thing into **CS Student OS**, a ready-to-use Notion template with all five databases pre-built, the GPA rollup working out of the box, the spaced-repetition formula already wired up, template buttons set, and a dashboard that's ready the second you duplicate it into your workspace.

If you want to skip the two weekends I spent fighting with formulas and just start using the system today, you can grab it here:

**[CS Student OS — Gumroad](https://8837207235283.gumroad.com/l/cs-student-os)** ($12+, pay what you want)

It's the exact workspace I use, cleaned up and documented. You'll spend ten minutes customizing it instead of ten hours building it, and once it's yours you can tweak it however you want — add databases, change the formulas, rework the dashboard, whatever fits your workflow.

Good luck this semester. Go to sleep. Don't burn out.

---

*If you found this useful, the template linked above is the fastest way to put it into practice. And if you have questions about setup or want to share your own version, my DMs are open.*
