---
name: setup-scenario
description: Wire bot groups for a first Loadster load test scenario and report the Fuel estimate and launch blockers. Use when the user wants a first scenario or to shape how a script runs under load. Does not launch the test.
---

# Shaping a first load test scenario

A scenario says how many bots run a script, from where, and for how long. The dashboard calls each group of bots
a bot group. The tool calls the same thing a population. This skill sets the shape only: the script, the bots,
the ramp, the duration, and the region. Saving a scenario costs nothing and starts nothing.

## Rules

- Never launch a cloud load test, enable a monitor, or spend Fuel without explicit human confirmation in-product,
  and never claim that you can.
- Prefer an existing project. Call `list_projects`, and call `create_project` only when no project exists or the
  user explicitly asks for one.
- Ask the user for target URLs, how to handle credentials, and regions. Never invent customer facts.

## Before you build

Call `list_projects` and use the project the user names. Call `create_project` only when the list is empty or the
user explicitly asks for a new project. Ask what to name it, and do not invent a name. If `create_project` is not
on the server, ask them to create the project in the dashboard, then call `list_projects` again.

The scenario needs a script in that project. If there isn't one, or it has not been played successfully, follow
`setup-script` first.

Ask for the target URL and how credentials are handled when the script does not already settle them. Ask for the
shape: which script, how many bots, the ramp, the duration, and the region. Do not fill any of that in yourself. Call `list_engines` for the regions and any private engines, and use an id from that list.
Read `get_documentation` for `load-test-scenarios` before choosing stages. A guessed region name is refused, and
so is a stage shorter than the account allows. The refusal names the limit.

Leave iterations, bandwidth, aggression, and cluster scaling alone unless the user asks for them. Starting
smaller than a number they already gave is a suggestion you make out loud, not a silent change.

## Create the scenario

Call `create_scenario` with that shape. If the save is refused, correct the request it names and try again. If a
scenario was saved, change it with `update_scenario` rather than creating a second one to retry.

## Report the estimate and the blockers

Report the Fuel estimate and the launch blockers that `create_scenario` returns, and give the user the dashboard
link from the summary. Do not calculate a Fuel number the summary didn't return. A launch blocker means the user
could not launch this scenario as it stands. It does not undo the save. Tell them about the blocker. Do not
rewrite the scenario to avoid it.

Do not launch the test, and do not tell the user a test is running. The launch button is theirs, in the
dashboard, and that is the explicit confirmation in-product. After they have run it, `results` covers the report.

## Prompt for Cursor and Codex

```text
Set up my first Loadster load test scenario. Ask which project and script to use, how many bots, the ramp, the duration, and the region. Report the Fuel estimate and any launch blockers. Do not launch the test or spend Fuel.
```
