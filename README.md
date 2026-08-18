# claude-toolkit

Personal [Claude Code](https://code.claude.com) plugins. One repo, acting as both the marketplace
and the plugin source.

## Install

```sh
/plugin marketplace add MisterShack/claude-toolkit
/plugin install review-kit@mistershack
/plugin install build-kit@mistershack
/plugin install ship-kit@mistershack
```

Then `/reload-plugins` if the install summary asks for it. A plugin installed after a session began
is not visible to that session — if the agents do not appear, that is why.

To work on it locally without installing:

```sh
claude --plugin-dir ./plugins/review-kit --plugin-dir ./plugins/build-kit --plugin-dir ./plugins/ship-kit
```

## What's in it

### `review-kit`

Five tools that exist for the same reason: the expensive defects are the ones nobody was looking
for, and each of these looks somewhere a test suite cannot.

| Component | Kind | What it does |
|---|---|---|
| `/review-kit:plan-review` | skill | Attacks a plan document before any of it is built. Findings are `WRONG` / `UNSTATED` / `UNPROVEN`, each quoting the sentence it disputes, ending in `PROCEED` / `REVISE` / `RETHINK`. |
| `design-reviewer` | agent | Reviews a UI against the project's **own** written design system — tokens, scale, component rules, voice. Refuses to review without one, because a review with no reference produces adjectives. |
| `web-accessibility-reviewer` | agent | Drives the running **web** app in a browser and audits the **accessibility tree** — names, keyboard operability, focus on navigation, whether status messages are announced, contrast in both themes, reflow. |
| `migration-rehearser` | agent | Builds a database at the *previous* migration, seeds realistic rows, runs the migration forward, and reports what breaks, what is silently lost, and what the database looks like if it dies halfway. |
| `doc-drift-auditor` | agent | Checks what the documentation *claims* against what the code, tests and git history *show*. Findings are `WRONG` / `STALE` / `UNOWNED` / `UNVERIFIABLE`, each quoting the sentence and carrying its evidence. |

All five are read-only by design. They report; you decide.

The two UI agents are complementary rather than overlapping: the accessibility one asks *can
people use this*, the design one asks *does this look considered and consistent*. Neither is
useful without the app running.

The accessibility agent is deliberately scoped to the web — a DOM, a browser accessibility tree
and CSS media queries. Native mobile and desktop toolkits have their own trees and conventions,
and several of its checks are meaningless there, so it says so and stops rather than producing
findings that do not apply. Other surfaces get their own agents as they come up.

#### Why the accessibility one drives the app

It was written after two real defects shipped, each caught by accident when a browser driver
failed to find a field by its label:

- a labelled-field component rendered `<label>` as a *sibling* of its control with no `htmlFor`,
  associating nothing — every form was affected;
- nesting a `<select>` inside its `<label>` folded every `<option>` into the field's accessible
  name, so a timezone picker announced all 306 zone names before saying what the field was for.

Neither is visible in a screenshot. Neither fails a unit test. Both are obvious the moment you
read the accessibility tree — which is why the agent insists on the running app and refuses to
work from source.

#### Why documentation gets its own auditor

Because prose is the only artifact in a repository with no test, no type checker and no compiler,
and it rots in exactly one direction — it is written when a claim is true and never revisited when
it stops being. The failures are specific rather than vague: a phase marked done against an
acceptance criterion it did not meet; a plan header still reading "draft, not started" after three
of its phases had shipped; one fact stated in three documents with three different values, so that
which file you opened decided what you believed.

The rule that makes it useful rather than a proofreading pass: it audits for claims that are
**false**, never for claims that are missing. A thin document costs a reader a question; a wrong
one costs them a decision.

### `build-kit`

Three agents that **write code**, which is the whole difference from `review-kit`. They are split by
stack rather than by seniority, because what makes an implementation agent useful is not being told
to be careful — it is knowing the specific places this stack goes wrong.

| Component | Kind | What it does |
|---|---|---|
| `react-frontend-developer` | agent | React on the web. Reads the project's design system and data layer before writing, keeps testable logic out of components, and carries the traps: nested interactive elements, computed accessible names, service workers that update themselves but not the page, caches that hand back missing fields. |
| `hono-drizzle-backend-developer` | agent | Hono routes and Drizzle schema. Validates every write server-side, checks authorisation in one place, and treats a migration against live data as the dangerous thing it is — including that Drizzle cannot tell a rename from a drop-and-add. |
| `playwright-e2e-author` | agent | End-to-end tests in a real browser. Seeds through the API rather than driving eight forms, selects by role and accessible name, pins the clock and the timezone, and covers offline and service-worker behaviour. Proves each new spec fails before believing it passes. |

The first two hand off to each other rather than overlapping: the frontend agent stops at the API
boundary, the backend agent stops at the client. Shared validation schemas are the seam, and the one
place a change is legitimately cross-cutting. The e2e agent sits across both, and is the only one of
the three whose subject is the seam itself — which is where the defects below actually lived.

#### Why they are stack-specific

A generic "senior frontend developer" prompt produces generic code, because the instructions have
nowhere to bite. Every trap listed in these agents is one that shipped in a real project, passed
code review, and passed a green test suite:

- a whole-card `<Link>` with a second action inside it — an anchor nested in an anchor, which
  behaves differently in every browser;
- `Directions<span class="visually-hidden"> to Paris</span>`, whose accessible name computes as
  `Directionsto Paris`, because name computation collapses the leading space;
- `registerType: 'autoUpdate'`, which updates the service *worker* while the open page keeps the
  JavaScript it loaded — an installed PWA can sit a week behind;
- a read-through cache that stores raw JSON, so an entry written by an older build returns
  `undefined` where the new code checked for `null`;
- a `drizzle-kit` migration that read a column rename as a drop plus an add, which on live data is
  every existing row;
- an in-memory database destroyed by its first transaction, surfacing as a cascade of
  unrelated-looking "no such table" failures.

None of these is a knowledge gap. All of them are ambush, and an agent is worth having exactly to
the extent that it has been ambushed before.

### `ship-kit`

Two agents whose subject is **the deployment, not the repository**. That is the reason they are a
separate plugin rather than more entries in the two above: everything in `review-kit` and
`build-kit` can be answered by reading or running the code, and neither of these can. One asks
whether a change reached production; the other asks why an image that works on a laptop does not
work on the host.

| Component | Kind | What it does |
|---|---|---|
| `release-verifier` | agent | Proves a release is actually live: the deployed artifact contains the commit you think it does, health held *through* the rollover rather than being checked afterwards, and platform configuration has not drifted. Read-only in production, and never rolls back. |
| `deploy-investigator` | agent | Reproduces a deploy failure in the real image locally — free, and risks nothing — then bisects the differences between the working environment and the failing one. May edit the repo; never touches the platform. |

They split along whether the deployment is believed to be working. `release-verifier` runs when it
should be fine and asks for evidence; `deploy-investigator` runs when it is not and asks why.

#### Why "the deploy went green" is not the check

It is the platform's claim about its own job, and it is compatible with every one of these, all of
which have really happened:

- a green deploy of the **wrong commit**, because a build filter or watch path meant the build never
  ran and the platform reported success for redeploying what was already there;
- a platform start command **overriding the image `ENTRYPOINT`**, so the entrypoint script and the
  replication sidecar it launched never ran — everything read as configured, and nothing replicated;
- a **volume mounted at a path the app does not write to**: everything worked, and every deploy threw
  the data away;
- a **window of 5xx during rollover** that nobody saw, because health was checked once, at the end —
  where a deploy that dropped every request for ninety seconds looks identical to one that dropped
  none;
- an **environment variable required at boot** and absent, whose only symptom was a healthcheck
  timeout that named nothing.

The shared property is that the deploy log reports the symptom and never the cause, and that the
cause is usually a difference between two environments rather than a defect in either. Hence the two
hard rules these agents keep: verify the **artifact** rather than the dashboard, and reproduce
**locally in the real image** before changing anything that is running.

## Adding to this

The marketplace holds many plugins; `review-kit` is one. A new tool is either a component inside
it (another `agents/*.md` or `skills/*/SKILL.md`) or, if it addresses a different surface, a new
plugin directory plus one entry in `.claude-plugin/marketplace.json`.

Prefer narrow and opinionated over broad and generic. These are useful because they know where to
look; an agent that tries to cover every platform ends up listing criteria instead of finding
defects.

## Origin

Every one of these was built and proven against a real project before being generalised here —
the reviewers after defects they failed to catch, the builders after defects they caused. The
project-specific versions stay in that repo, tuned to its own invariants; these are the portable
cores, meant to be useful in a repo on day one.

## Licence

MIT.
