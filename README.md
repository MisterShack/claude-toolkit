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

Two tools that exist for the same reason: the expensive defects are the ones nobody was looking
for, and both of these look in places a test suite cannot.

| Component | Kind | What it does |
|---|---|---|
| `/review-kit:plan-review` | skill | Attacks a plan document before any of it is built. Findings are `WRONG` / `UNSTATED` / `UNPROVEN`, each quoting the sentence it disputes, ending in `PROCEED` / `REVISE` / `RETHINK`. |
| `accessibility-reviewer` | agent | Drives the running app in a browser and audits the **accessibility tree** — names, keyboard operability, focus on navigation, whether status messages are announced, contrast in both themes, reflow. |

Both are read-only by design. They report; you decide.

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

## Origin

Both tools were built and proven against a real project before being generalised here. The
project-specific versions stay in that repo, tuned to its own invariants; these are the portable
cores, meant to be useful in a repo on day one.

## Licence

MIT.
