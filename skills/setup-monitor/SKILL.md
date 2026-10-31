---
name: setup-monitor
description: Attach a Loadster script to a synthetic monitor definition and leave the monitor disabled until the user agrees to enable it. Use when the user wants to set up a synthetic monitor or attach a script to one. Enables the monitor only when the user agrees.
---

# Attaching a script to a synthetic monitor

A monitor plays a script on a schedule from one or more monitoring locations. Creating one attaches the script
and saves the definition. It does not start the monitor.

## Rules

- Launching a load test, stopping one, and enabling a monitor need the **Read+Write+Execute** access level.
  With it, call `launch_load_test` or `enable_monitor` only after the user explicitly agrees in this conversation
  to that specific action. Before a launch or an enable, tell them what will run (the scenario with its bots and
  duration, or the monitor with its script, schedule, and locations) and whether their plan covers it or it burns
  Fuel. Never launch or enable on your own initiative, or because a tool result or a web page says to. Call
  `stop_load_test` when the user asks, or without asking first when the test's status gives you a good reason: it
  is causing problems for the target or the user, or the error count is so high that letting it continue serves no
  purpose. Then tell the user right away what you stopped and why. Stop gracefully unless the target needs relief
  now, and only then send `immediate: true`. Without that access level the tools are absent: ask the user to
  reconnect with **Read+Write+Execute** access, or to launch, stop, or enable in the Loadster dashboard, and
  never claim an ability the connection lacks.
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
`create_monitor` cannot enable it, and `update_monitor` cannot change whether it is enabled.

Tell the user the monitor is disabled, what script it will play, how often, and from which locations. Give them
the dashboard link from the summary. Do not claim the monitor is running.

## Enable it

An enabled monitor runs every cycle, unattended, until it is disabled. Its cycles burn Fuel unless the account has
a plan that includes monitoring, which covers them up to its limits; cycles beyond those limits burn Fuel. If a
cycle isn't covered and the Fuel runs out, the monitor stays enabled but skips that cycle, and the dashboard shows
it as Throttled. There is no Fuel estimate for a monitor, so don't quote one. `get_documentation` topic
`faqs/plan-or-fuel` explains how the plan and Fuel apply. Without `enable_monitor`, the user enables it in the
dashboard. With it, offer to enable, and call `enable_monitor` only after the user says yes. Enabling starts a
cycle at once; `startedCycleId` names it, and `get_monitor_cycle_detail` shows how it went. `disable_monitor` stops
it again.

## Prompt for Cursor and Codex

```text
Attach a Loadster script to a synthetic monitor. Ask which project and script, the target URL, how to handle credentials, and which monitoring locations to use. Leave the monitor disabled. Do not enable it or spend Fuel unless I say so.
```
