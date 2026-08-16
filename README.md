# claude-toolkit

Personal [Claude Code](https://code.claude.com) plugins. One repo, acting as both the marketplace
and the plugin source.

## Install

```sh
/plugin marketplace add MisterShack/claude-toolkit
/plugin install review-kit@mistershack
```

Then `/reload-plugins` if the install summary asks for it.

To work on it locally without installing:

```sh
claude --plugin-dir ./plugins/review-kit
```

## What's in it

### `review-kit`

Three tools that exist for the same reason: the expensive defects are the ones nobody was looking
for, and both of these look in places a test suite cannot.

| Component | Kind | What it does |
|---|---|---|
| `/review-kit:plan-review` | skill | Attacks a plan document before any of it is built. Findings are `WRONG` / `UNSTATED` / `UNPROVEN`, each quoting the sentence it disputes, ending in `PROCEED` / `REVISE` / `RETHINK`. |
| `design-reviewer` | agent | Reviews a UI against the project's **own** written design system — tokens, scale, component rules, voice. Refuses to review without one, because a review with no reference produces adjectives. |
| `web-accessibility-reviewer` | agent | Drives the running **web** app in a browser and audits the **accessibility tree** — names, keyboard operability, focus on navigation, whether status messages are announced, contrast in both themes, reflow. |

All three are read-only by design. They report; you decide.

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

## Adding to this

The marketplace holds many plugins; `review-kit` is one. A new tool is either a component inside
it (another `agents/*.md` or `skills/*/SKILL.md`) or, if it addresses a different surface, a new
plugin directory plus one entry in `.claude-plugin/marketplace.json`.

Prefer narrow and opinionated over broad and generic. These are useful because they know where to
look; an agent that tries to cover every platform ends up listing criteria instead of finding
defects.

## Origin

Both tools were built and proven against a real project before being generalised here. The
project-specific versions stay in that repo, tuned to its own invariants; these are the portable
cores, meant to be useful in a repo on day one.

## Licence

MIT.
