---
name: loadster-load-test
description: Author and verify a Loadster load test end to end. Write a script, validate and play it, build a load test scenario, and hand off for launch. Use when the user wants to load test, stress test, or performance test a site or API with Loadster, or asks for a Loadster script.
---

# Building a load test in Loadster

Loadster runs load tests from scripts. A script describes what one bot does; a scenario says how many bots run it,
from where, and for how long. Your job is to get a verified script and a ready scenario into the user's project.
Launching the full test is the user's job, on purpose, and no tool lets you do it.

## Before you write anything

1. **Confirm authorization.** The target must be a system the user owns or is authorized to test. If that isn't
   clear from context, ask. Loadster's terms require it and you shouldn't assume it.
2. **Pick the project.** Call `list_projects` and use the one the user names. If they're experimenting, suggest a
   separate project so the agent's work doesn't mix with the team's real tests.
3. **Read the documentation for the script type you're about to use.** Call `get_documentation` for the relevant
   topics before authoring. The server's `instructions` say the same thing, and it matters: the script formats have
   details you will get wrong from memory.
4. **Choose the script type.**
   - **Protocol Bots** (HTTP-level) for APIs, simple sites, and high bot counts at low cost.
   - **Browser Bots** (real Chrome, step-based) for websites and web apps where you want realistic client-side
     behavior without writing code.
   - **Playwright Test** scripts (code-first JavaScript) when the user already has Playwright tests or wants full
     control.
   Use `list_command_types` and `get_command_schema` for Protocol and Browser Bot commands, `get_scripting_api` for
   code blocks and Playwright, and `get_example` when you need a starting point.

## Author, validate, play

A saved script is unverified until it has been played. Follow this loop and don't skip steps:

1. **Create or update** the script with `create_script` or `update_script`. Use gentle, realistic think times
   between steps. Use `create_dataset` and `append_dataset_rows` when steps need varied data such as logins or
   search terms, then reference the dataset from the script.
2. **Validate** with `validate_script`. Fix every problem it reports before playing.
3. **Play** with `play_script`. This runs a single bot through the script once. Poll `get_play_status` until it
   finishes.
4. **Inspect the result.** Use `get_step_detail` for each failing or suspicious step (status codes, response bodies,
   captured values, logs), and `get_screenshot` for Browser Bot and Playwright plays. Fix the script and play again
   until every step passes for the right reasons, not just without errors. A 200 from a login page that didn't log in
   is still a failure.
5. **Stop** a stuck play with `stop_script` rather than leaving it running.

Tell the user what you changed between plays and why. Script updates create revisions, so
`list_script_revisions` and `restore_script_revision` can undo a bad edit.

## Build the scenario

1. Call `list_engines` to see which regions and any private engines are available.
2. Create the scenario with `create_scenario`: one or more bot groups, each with a script, a bot count, a ramp
   pattern, and a region. Start smaller than the user's target and suggest ramping up in later runs; the goal of a
   first test is usually to find where the system starts to degrade, not to knock it over.
3. Ask before anything destructive. Deleting scripts, datasets, or scenarios removes the team's work, so confirm
   with the user first even if the client doesn't prompt.

## Hand off

Summarize what you built: the script, what it does, what the play showed, and the scenario's bot groups. Point the
user to the scenario in their Loadster dashboard to launch it. Once the test has run, the `loadster-results` skill
covers reading the report.

Finally, call `submit_feedback` with a short note on what worked and what didn't. It goes to the Loadster team and
is how the tool surface improves.
