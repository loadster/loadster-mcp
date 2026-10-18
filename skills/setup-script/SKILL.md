---
name: setup-script
description: Create or import a first Loadster script, then validate, play, and check it. Use when the user wants a Protocol Bot, Browser Bot, or Playwright script, asks for a browser script, or wants to import a JMeter, k6, or Locust test.
---

# Creating a first Loadster script

A script is what one bot does. This skill gets one script into the user's project and proves it with a play. A
play is one bot running the script once. It is not a load test.

## Rules

- Never launch a cloud load test, enable a monitor, or spend Fuel without explicit human confirmation in-product,
  and never claim that you can.
- Prefer an existing project. Call `list_projects`, and call `create_project` only when no project exists or the
  user explicitly asks for one.
- Ask the user for target URLs, how to handle credentials, and regions. Never invent customer facts.

## Choose the type

If the user says "browser script", ask whether they mean a Browser Bot script or a Playwright script. Do not
guess. Otherwise read [the script-type guide](../load-test/references/script-types.md) and let them pick when it
isn't already clear:

- **Protocol Bots** for APIs, simple sites, and high bot counts.
- **Browser Bots** for websites and web apps, written as steps, with no JavaScript required.
- **Playwright** when they already have Playwright tests or want code.

Call `get_documentation` for that type before authoring. Use `list_command_types` and `get_command_schema` for
Protocol and Browser Bot commands, `get_scripting_api` for code blocks and Playwright, and `get_example` when you
need a starting point. The formats have details you will get wrong from memory.

## Pick the project

Call `list_projects` and use the project the user names. Call `create_project` only when the list is empty or the
user explicitly asks for a new project. Ask what to name it, and do not invent a name. If `create_project` is not
on the server, ask them to create the project in the dashboard, then call `list_projects` again.

Ask for the target URL, how credentials should be handled, and which region they want for a later scenario or
monitor, before you write a step. Do not invent a host, username, password, API key, token, or region. If they
do not want credentials stored in the project, say what the script cannot check without them and wait. A script
does not select a load region itself. Keep the region they name for `setup-scenario` or `setup-monitor`.

## Create or import

Validate, then create, then play, then check the play status.

1. **Validate** the commands with `validate_script` and fix every problem it reports before saving.
2. **Create** the script once with `create_script`. To import a JMeter `.jmx`, k6 `.js`, or Locust `.py` file,
   create one script and import into it with `import_script`, then review the action items it returns. An import
   can also create assets, datasets, and a scenario. OpenAPI, Swagger, and Postman are not an import. Read the
   spec and build the script with `create_script`.
3. **Play** with `play_script`.
4. **Check the play status** with `get_play_status` until the play has finished. Use the tool's wait rather than
   sleeping between calls.

Inspect a failing or suspicious step with `get_step_detail`, and use `get_screenshot` for a Browser Bot or
Playwright play. A passing status is not enough when the step didn't do what the user asked. The failure modes in
the script-type guide are worth reading before you guess. Stop a stuck play with `stop_script`.

## Retries

Reuse and edit the script you created. A failed play is a reason to `update_script` and play again, not a reason
to create another script of the same type. Tell the user what you changed between plays and why.

Never change a script's type. `update_script` can replace the type, but you must send the type the script already
has. If the user wants a different type, create a new script and leave the old one. Confirm with the user before
you edit a script you did not create in this conversation.

## Hand off

Summarize the script, what it does, and what the play showed, and give them the dashboard link from the summary.
Point them at `setup-scenario` to shape a load test, or `setup-monitor` to attach the script to a monitor. Do not
launch a test or enable a monitor.

## Prompt for Cursor and Codex

```text
Create my first Loadster script. Ask me the script type (if I say "browser script", ask whether I mean Browser Bot or Playwright), which project to use, the target URL, and how to handle credentials. Validate it, play it, and check the play status. Edit that script if it fails. Do not launch a test, enable a monitor, or spend Fuel.
```
