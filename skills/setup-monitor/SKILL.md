---
name: setup-monitor
description: Attach a Loadster script to a synthetic monitor definition and leave the monitor disabled. Use when the user wants to set up a synthetic monitor or attach a script to one. Does not enable the monitor.
---

# Attaching a script to a synthetic monitor

A monitor plays a script on a schedule from one or more monitoring locations. Creating one attaches the script
and saves the definition. It does not start the monitor.

## Rules

- Never launch a cloud load test, enable a monitor, or spend Fuel without explicit human confirmation in-product,
  and never claim that you can.
- Prefer an existing project. Call `list_projects`, and call `create_project` only when no project exists or the
  user explicitly asks for one.
- Ask the user for target URLs, how to handle credentials, and regions. Never invent customer facts.

## Before you attach

Call `list_projects` and use the project the user names. Call `create_project` only when the list is empty or the
user explicitly asks for a new project. Ask what to name it, and do not invent a name. If `create_project` is not
on the server, ask them to create the project in the dashboard, then call `list_projects` again.

The monitor plays a script that already exists in that project. If there isn't one, or it has not been played
successfully, follow `setup-script` first. Ask for the target URL and how credentials are handled when the script
does not already settle them. Do not invent them.

Ask which locations should run the check. Call `list_monitoring_locations` and use ids from that list. Monitoring
locations are not the load test regions `list_engines` returns, and a private engine is the only kind that can
reach a host on their network. Never invent a location id.

Read `get_documentation` for `monitoring` before you choose a frequency or any thresholds. Ask how often the
check should run. Do not invent a schedule, tags, or thresholds the user did not ask for. Omit thresholds and
tags when they didn't ask.

## Create the monitor

Call `create_monitor` with the script and the locations they chose. The monitor is created disabled.
`create_monitor` cannot enable it, and `update_monitor` cannot change whether it is enabled. Do not try to start
it. An enabled monitor consumes Fuel on its schedule, unattended. Enabling is the user's act in the dashboard,
and that is the explicit confirmation in-product.

Tell the user the monitor is disabled, what script it will play, how often, and from which locations. Give them
the dashboard link from the summary. Do not claim the monitor is running.

## Prompt for Cursor and Codex

```text
Attach a Loadster script to a synthetic monitor. Ask which project and script, the target URL, how to handle credentials, and which monitoring locations to use. Leave the monitor disabled. Do not enable it or spend Fuel.
```
