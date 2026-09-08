---
description: Interactive setup for business-kit. Asks about hosting, domains, app type, brand, database and vendors, then writes .claude/team/PROJECT.md so every business-kit agent configures itself to this specific project instead of a generic default. Run once after installing the plugin, and re-run any time the project's answers change.
---

# onboard — configure business-kit for this project

`business-kit`'s agents (`finance-officer`, `market-strategist`, `platform-engineer`,
`privacy-counsel`, `database-reviewer`, `lead-engineer`, `security-reviewer`, `test-strategist`) are
deliberately written stack-agnostic and product-agnostic — they carry the methodology, not the
facts. This skill fills in the facts, once, so none of them has to ask again mid-review.

The output is a single file, `.claude/team/PROJECT.md`. Every business-kit agent reads it first.

## If `.claude/team/PROJECT.md` already exists

Read it first. Summarise what it currently says in a couple of lines, ask the user whether they
want to update specific sections or redo the whole thing, and only ask questions for what's
changing — do not make someone re-answer everything to fix one field.

## Ask the questions

Use `AskUserQuestion`, batched sensibly (a few related questions per call rather than one at a
time). Cover at least:

1. **What is this product, in one line, and what stage is it at** (idea / actively building, no
   users yet / private beta / launched with real users)? Stage matters a lot downstream —
   `privacy-counsel` and `platform-engineer` both treat "no real user data yet" very differently from
   "shipping to real users."
2. **What kind of app is it** — web app, mobile app (iOS/Android/both), native desktop, a
   backend/API with no first-party client, a browser extension, or some combination? Multi-select.
3. **Where is it hosted, and where is it going to be hosted** (the platform — e.g. a PaaS like
   Railway/Fly/Render/Heroku, a cloud provider directly, Vercel/Netlify/Cloudflare for static or
   edge, self-hosted, or not decided yet)?
4. **Domain(s)**, if any are registered or planned, and whether a public marketing/waitlist page
   exists separately from the product itself.
5. **Database**: which engine (Postgres, MySQL, SQLite, a managed NoSQL store, none yet), and where
   it's hosted or planned to be hosted. Same question for any object/media/file storage if the
   product handles files, images, video or documents.
6. **Brand**: does a brand/design document already exist (`BRAND.md`, `DESIGN.md`, a Figma link,
   etc.)? If not, ask for the core brand colours (even rough hex values or "haven't decided") and the
   one-line tone/voice the product should read in — `market-strategist` uses this to judge whether
   copy and palette choices actually fit, and other agents may point users to it when a design
   document is genuinely missing.
7. **Key vendors/integrations** already chosen or planned: payments, transactional email, any LLM/AI
   API, analytics, print/fulfilment, or anything else that receives or processes user data. This is
   what `platform-engineer` and `privacy-counsel` most need.
8. **What personal data does the product actually handle**, at a category level — names/emails only,
   photos or media of people, precise location, health information, financial information, anything
   about children, or "not sure yet." Be direct that this drives how seriously `privacy-counsel`
   treats the project; getting it wrong in the conservative direction (over-reporting sensitivity) is
   fine, under-reporting is the expensive mistake.
9. **Does the product have any multi-user, sharing or collaboration surface** — can one user's data
   ever be seen, edited or removed by another? `security-reviewer` and `database-reviewer` both need
   this to know whether authorization/tenancy questions apply at all.

Skip a question outright if `.claude/team/PROJECT.md` or another project document already answers it
clearly — confirm rather than re-ask.

It's fine for an answer to be "not decided yet" or "not sure." Write that down as-is rather than
forcing a premature decision — a project agent reading "hosting: not decided yet" should ask before
assuming a stack, not silently pick one.

## Write `.claude/team/PROJECT.md`

Create `.claude/team/` if it doesn't exist. Write (or rewrite) the file with this shape — omit a
section only if the user has genuinely not decided, and say so explicitly rather than leaving it
blank:

```markdown
# Project brief — for business-kit agents

Read by: finance-officer, market-strategist, platform-engineer, privacy-counsel, database-reviewer,
lead-engineer, security-reviewer, test-strategist. Keep this current — re-run `/business-kit:onboard`
when any of this changes.

## What this is

<one-line description> — stage: <idea / building / private beta / launched>

## Platforms

<web / mobile (iOS, Android) / desktop / backend-only / browser extension, etc.>

## Hosting

<platform, or "not decided yet">

## Domains

<domain(s), or "none yet">. Public marketing page: <yes/no, where>.

## Database

Engine: <...>. Hosted: <...>. Media/file storage: <..., or "none">.

## Brand

<link to BRAND.md/DESIGN.md if one exists, otherwise the core colours and one-line voice given here>

## Vendors and integrations

<payments, email, LLM/AI APIs, analytics, fulfilment, anything else that touches user data>

## Data this product handles

<categories, at the level of specificity the user gave — this is what privacy-counsel scopes to>

## Sharing / multi-tenancy

<whether one user's data can be seen or changed by another, and roughly how>

## Revisions

- **<date>** — Created via `/business-kit:onboard`.
```

Use today's date for the revision line. On a re-run that updates existing sections, append a new
revision line describing what changed rather than silently overwriting the history.

## After writing it

Tell the user in the terminal, briefly: the file was written (or updated), which sections are marked
"not decided yet" (if any — those are worth flagging, since agents will ask about them again until
they're filled in), and that any business-kit agent can now be invoked directly and will read this
automatically.

Do not invoke any other business-kit agent as part of this skill — onboarding's job is the file, not
a first review.
