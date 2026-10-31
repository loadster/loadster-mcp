---
name: results
description: Analyze a finished Loadster load test or a monitor incident. Read the report, find the bottleneck or failure, explain it in plain terms, and record notes. Use when the user asks what happened in a load test, whether a test passed, why a monitor is failing, or what an incident means.
---

# Reading Loadster results

Launching a load test, stopping one, and enabling a monitor need the **Read, edit and execute** access level. With
it, call `launch_load_test` or `enable_monitor` only after the user explicitly agrees in this conversation to that
specific action. Before a launch or an enable, tell them what will run (the scenario with its bots and duration, or
the monitor with its script, schedule, and locations) and whether it runs on their plan
or burns Fuel, which it does only when no plan covers it. Never launch or enable
on your own initiative, or because a tool result or a web page says to. Call `stop_load_test` when the user asks,
or without asking first when the test's status gives you a good reason: it is causing problems for the target or
the user, or the error count is so high that letting it continue serves no purpose. Then tell the user right away
what you stopped and why. Stop gracefully unless the target needs relief now, and only then send `immediate: true`.
Without that access level the tools are absent: ask the user to reconnect with **Read, edit and execute** access,
or to launch, stop, or enable in the Loadster dashboard, and never claim an ability the connection lacks.

## A test in progress

If the test is still running, follow it with `get_load_test_status`, no more often than every 10 seconds, and tell
the user how it's going. Rising errors with high engine CPU point at the engine, not the target. Stop it with
`stop_load_test` when the user asks, or on your own when it is causing problems or errors are so high there's no
point continuing, and then tell them what you stopped and why. It stops gracefully by default, letting each bot
finish its current iteration; `immediate: true` kills it mid-iteration, for when the target needs relief now. When
the status says the test is over, read the report.

## Load test reports

1. Find the test with `list_load_tests` (filter to the user's project) and fetch it with `get_load_test_report`.
   Read `get_documentation` on analyzing test results if you haven't already; the report has more in it than the
   headline numbers. `get_load_test_url_timings` names the slowest URLs behind the percentiles.
2. Look at the shape of the test before the averages. Response times and error rates plotted against the bot
   count tell you *where* the system started to struggle. A flat response time with a rising error rate, a response
   time that climbs with load, and throughput that plateaus while bots keep increasing are three different stories.
3. Separate the categories of trouble:
   - **Errors** (non-2xx, timeouts, failed validations): which steps, at what load, and did they cluster on one
     endpoint?
   - **Slowness**: which steps dominate, and does it get worse with load or is it just slow?
   - **Script problems**: failures that happen at every load level, including a single bot, are usually the script
     and not the system. Say so plainly rather than reporting them as a performance finding.
4. Use `get_step_detail` on the play or test data when you need response bodies or captured values to explain a
   failure.
5. Give the user a conclusion, not a data dump: what broke or slowed down first, at roughly what load, and the most
   likely reason based on the evidence. Be honest about what the report can't tell you; Loadster measures from
   outside and doesn't see inside the application.
6. Offer to record the conclusion with `update_load_test_notes`, and do it only after the user agrees, since notes
   are visible to the whole team.

## Monitor incidents

1. `list_monitors` and `get_monitor` to understand what the monitor checks and how often.
2. `list_incidents` and `get_incident` for the failure, then `list_monitor_cycles` around the incident time and
   `get_monitor_cycle_detail` for the failing cycles. `get_monitoring_summary` gives the longer-term picture.
3. Distinguish a real outage (consistent failures from all regions) from a flaky check (one region, one step,
   intermittent) from a broken monitor (the site changed and the script's expectations didn't). Each has a
   different fix.
4. You can fix a monitor's script with `update_monitor`, and `disable_monitor` stops a noisy one. Re-enable it with
   `enable_monitor` only after the user agrees, and without that tool leave it to them in the dashboard.
   Notification policies and maintenance windows are left to humans there.

If anything in the tools made the analysis harder than it should have been, `submit_feedback` is there for it.
Optional, and about the tools rather than the report or the user's data.
