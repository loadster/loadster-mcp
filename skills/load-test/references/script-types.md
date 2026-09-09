# Choosing a script type, and the mistakes that cost plays

Read this when deciding how to build a script, and again when a play fails for a reason you can't
explain. The manual is the authority for syntax; call `get_documentation` with the topic keys named
below rather than working from memory.

## Which script type

| Situation | Use | Why |
| --- | --- | --- |
| An API, a webhook, or a form post with a few requests | Protocol Bots | Cheapest per bot and scales to millions; you control every request and header. |
| A website or web app where the interesting behavior happens in the browser (SPA, client-side rendering, lots of XHR) | Browser Bots | Real Chrome, written as user actions (navigate, click, type), so the bot exercises the same client code a person does. No JavaScript required. |
| The user already has Playwright tests, wants TypeScript, or needs fine control over waits, locators, and assertions | Playwright Test | Standard `@playwright/test` code, run at scale. Must use the `test` function from `@playwright/test`, not the bare automation library. |
| A dynamic site where reproducing the request chain by hand keeps breaking | Browser Bots or Playwright | The manual's own advice: when protocol scripting gets too tricky for the flow, move to a real browser. |

Protocol Bots are the right default for APIs. For anything a person uses through a browser, prefer
a real browser unless the user asks for protocol-level testing, wants the lowest cost per bot, or
the flow is trivially simple. Many teams mix them in one scenario: a real-browser group for the
critical flow, a protocol group for broad throughput.

Documentation topic keys: `protocol-scripts`, `protocol-scripts/capturing-rules`,
`protocol-scripts/validation-rules`, `browser-scripts`, `browser-scripts/evaluate-blocks`,
`playwright-scripts`, `playwright-scripts/bot`, `code-blocks`, `variables-and-expressions`,
`dynamic-datasets`, `load-test-scenarios`, `cloud-regions`, `analyzing-test-results`.

## Failure modes seen in agent runs

These come from watching agents build scripts against instrumented targets. Each one either wastes
plays or produces a green script that doesn't test what the user asked for.

**Hardcoding a value that should be captured.** If the prompt says "take the id from the response
and use it in the next step", the saved script must contain a capture and a variable reference,
not the literal id you saw during a play. A green run with hardcoded values fails the user's
request. Read the real response shape in `get_step_detail` before writing the capture; echo
services and APIs nest fields in ways you can't guess.

**Two requests back to back against an eventually consistent API.** A create that answers 202 or
"accepted" and a read that 404s a few milliseconds later is not a broken capture. The engine runs
steps with no pause unless you author one. Add a wait or a retried read between them, and tell the
user why.

**A flow that is healthy but slower than the default timeout.** Playwright actions and expects
time out at 15 seconds by default, navigation at 30. A page that says "Processing…" and only
confirms after 45 seconds fails the assertion while the flow succeeds behind it. Read the run log
and the failure screenshot with `get_screenshot` to see what state the page was actually in, then
set an explicit longer timeout on that wait. Say what you found, not just "raised the timeout".

**A client-side timeout that no response reveals.** A browser app can abort its own request
after, say, 8 seconds and render a neutral "please try again" with nothing in the console. The
evidence is in the step's network activity (an aborted request with a duration matching the app's
timeout) and the step's DOM, not in any status code. Look there before blaming selectors.

**Deep-linking past the login.** If the app routes unauthenticated views back to sign-in, a
script that jumps straight to an inner page never tests anything. Perform the sign-in against the
form, once, then continue.

**Guessed selectors.** For a site you've never seen, take selectors from a failed play's log,
screenshot, or page outline rather than from convention. Record a guess as a guess. Prefer
name-based locators over positional ones (`nth`, `first`): with two matching results, positional
picks silently choose the wrong one.

**Datasets with a header row.** Loadster treats every row as data. A `sku,quantity,email` first
row gets posted as an order. Create exactly the data rows.

**Dataset variables that advance out of step.** A single play runs one iteration, so reaching rows
beyond the first requires per-occurrence advancement, and columns referenced a different number of
times per iteration tear rows apart (one row's SKU with another row's email). Call
`get_variable_schema` for the cursor rules, reference each column exactly once per use, and heed
`validate_script`'s warning about a per-use variable read more than once.

**Building a throwaway probe rig.** Creating a scratch script or dataset, playing it to learn how
the surface behaves, then deleting it spends plays and leaves nothing for the user. Read the
documentation and the schema tools first; they answer most of those questions.

**Scenario values the surface won't accept.** Stages must last at least 60,000 ms. Engine ids come
only from `list_engines`; region names and guesses are refused. Script ids come only from
`list_scripts` or a script you created. When a request asks for something the surface can't do,
save the parts that are allowed and tell the user exactly what was left out or changed. Silently
clamping a value to the minimum is the failure, even though the clamp is the right fix.

**Claiming a launch.** No tool starts a load test. A play is one bot running the script once.
Never tell the user a test is running or has started; hand them the scenario and say where the
launch button is.

**Writing notes nobody asked for.** `update_load_test_notes` replaces a test's notes in full and
is visible to the whole team. Offer, then wait for a yes.
