---
name: results
description: Analyze a finished Loadster load test or a monitor incident. Read the report, find the bottleneck or failure, explain it in plain terms, and record notes. Use when the user asks what happened in a load test, whether a test passed, why a monitor is failing, or what an incident means.
---

# Reading Loadster results

## Load test reports

1. Find the test with `list_load_tests` (filter to the user's project) and fetch it with `get_load_test_report`.
   Read `get_documentation` on analyzing test results if you haven't already; the report has more in it than the
   headline numbers.
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
4. You can fix a monitor's script with `update_monitor`, and `disable_monitor` stops a noisy one. Enabling a monitor
   and managing notification policies or maintenance windows are left to humans in the dashboard.

If anything in the tools made the analysis harder than it should have been, `submit_feedback` is there for it.
Optional, and about the tools rather than the report or the user's data.
