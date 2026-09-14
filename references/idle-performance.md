# Idle behavior and continuous performance evidence

Separate three claims: application work stops when ineligible; runtime resource use
stays bounded; the real device saves energy. Request counts prove the first, OS
accounting helps with the second, and neither establishes battery use or a browser's
OS suspension behavior. A long test with the wrong observer still proves little.

## Deterministic regressions

Define the settling boundary and permitted ongoing work. Count requests, retries,
content invalidations and expensive projections across idle, hidden, offline,
refused-auth, ownership-handoff and return transitions. Preserve exact local/server
records and pending acknowledgements. A low-cost negative control should demonstrate
that the oracle rejects unwanted work; a quiet status badge is not that oracle.

Timers can be advanced for lifecycle logic, but accelerated clocks do not measure
real CPU, memory growth, browser throttling or energy. Measure no-change work as
well as representative edits; efficient request rates can conceal full-history
rebuilds. Run actual engines/storage for the boundary being claimed.

## Measurement without manufacturing activity

- Record source, dependencies, browser build/flags, workload identity, viewport,
  lifecycle input, host load and sampling settings. Keep builds immutable.
- Sample the browser process group from outside the page. Keep browser, application
  server and controller/collector costs separately attributable; when using cgroups,
  put descendants in the owned group at launch and record its full membership.
- Report CPU seconds per wall-clock interval with explicit normalization, memory
  current/peak/trend and request/byte counts. Process memory includes more than JS
  heap; cumulative elapsed spans are not CPU time. Host package power is not per-app
  battery use. Record throttling/pressure instead of treating contention as app work.
- Avoid frequent page evaluation, synthetic mouse events, idle heartbeats and full
  session recording during idle samples. Prefer external sampling and bounded
  event counters. Compare against a matched blank-browser control and an unchanged
  product baseline; measure collector overhead.
- Keep traces/profiles for short diagnostic windows, with bounded buffers and
  explicit overhead. Capturing a profile after an alert cannot recover an earlier
  transient stack; retain trigger metrics and reproduce with profiling when needed.

Playwright 1.63's Chromium defaults include `--disable-background-timer-throttling`,
`--disable-backgrounding-occluded-windows` and `--disable-renderer-backgrounding`.
Such automation is useful for testing application cleanup independently of browser
mercy. For native lifecycle claims, record and audit those flags, actual visibility
transitions and the browser/OS. Overriding `visibilityState` tests a policy input;
headless WebKit on Linux is not Safari/App Nap. Do not indiscriminately disable all
automation defaults or change the ordinary functional suite for a profiling run.

## Soak and trend testing

Give long tests an independent runner and explicit advisory/release classification.
Reuse the product fixture and build authorities; do not create another sync engine
or put a monitor inside the application. Preserve profiles and databases across
idle and reconnect cycles. Resetting every iteration hides cumulative leaks. Keep
deliberate reload/restart/update cases distinct from continuous-lifetime samples.

Choose duration from the failure mechanism: cover actual retry/heartbeat/update
boundaries in short tests; hours/days expose accumulation and lifecycle drift.
A week-long run needs a reason beyond being longer. Separate long elapsed-time
observations from exclusive latency measurements. On a shared host, record competing
jobs; caps protect the host but throttled samples cannot establish clean latency.

The native supervisor owns cadence, overlap, resource containment and cancellation
(`scheduling-jobs`). The runner writes source-bound progress and terminal receipts;
interruption, missing evidence, skipped admission and collector failure stay visible.
Use bounded retention and deduplicated regression reports. An advisory result must
not enter a release-blocking history implicitly. Promote a reproduced defect into
the relevant short regression; periodic success never replaces required checks.

Sources: Thinkering energy audit and second-opinion request, September 14, 2026.
[Playwright launch defaults](https://github.com/microsoft/playwright/blob/v1.63.0/packages/playwright-core/src/server/chromium/chromiumSwitches.ts),
[tracing overhead](https://playwright.dev/docs/best-practices#use-trace-viewer-to-explore-test-failures),
[cgroup accounting](https://docs.kernel.org/admin-guide/cgroup-v2.html),
[WebKit CPU timeline](https://webkit.org/blog/8993/cpu-timeline-in-web-inspector/),
[Long Tasks specification](https://www.w3.org/TR/longtasks-1/) (polling itself prevents idle).
