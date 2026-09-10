---
name: local-test
description: Design, add, repair or speed up automated tests. Choose behavioral oracles, isolated fixtures, native parallel execution and verification scope. Use for unit, API, storage, integration and browser tests; repository testing skills supply commands and fixtures.
---

# Tests that provide useful feedback

Read the repository's testing instructions and the nearest relevant test before choosing a runner or fixture. Extend the existing system unless a measured limitation requires a change. For browser interaction or screenshot claims, also use browser-verification. CSS-only changes still need browser evidence for their rendered requirement: inspect the affected composition and responsive behavior, not only current CSS constants. See references/mobile-layout-incident.md for the failure this prevents.

## Choose the proof

State the behavior and a relevant wrong result the test must reject. Select the cheapest boundary that can prove it: pure decision, native component, storage/API, browser interaction, cross-feature journey, performance workload or external probe. Size describes resources; scope describes behavior; cadence describes when the result is needed.

Use real implementations at the boundary claimed and controlled substitutes elsewhere. A mocked HTTP response proves client handling, not server persistence. A saved label proves acknowledgment, not database durability. Use supported setup shortcuts only when setup itself is outside the behavior; never seed the result being asserted.

Derive expected values independently of the implementation. Assert public behavior or a documented invariant. Source-shape assertions are appropriate when source shape is the contract, such as a static validator. They do not substitute for runtime behavior. Assertions should survive a harmless internal refactor.

Keep the smallest fixture that crosses the relevant boundary. Retain large fixtures when scale is the subject. Separate functional, performance and accessibility results when one obscures another. Keep at least one linked journey when integration between the extracted behaviors matters.

## Own state and time

Make each schedulable case independently construct its preconditions. It owns writable databases, files, origins, storage partitions, contexts, environment overrides, clocks and outputs. Intentionally related clients may share one case's state. Separate cases may share immutable assets or a runner-owned service only when mutable state is partitioned and lifecycle ownership is explicit.

Register cleanup immediately after allocation and await it after success, failure and cancellation. Stop only owned processes. Use allocated ports and run-specific output paths. Never kill an arbitrary process occupying a port or reset a database outside the fixture's ownership.

Wait for the event, assertion, transaction or process that establishes readiness. Avoid incidental sleeps and global network-idle assumptions. Use barriers or held promises to control races. Delays are legitimate when time, timeout or cancellation is the behavior. Fake clocks only at a boundary whose behavior they preserve.

Use realistic input when input behavior matters. A helper that waits for every save between keystrokes cannot prove continuous typing is safe. Prefer native eventual assertions over custom polling loops; assert final state as well as completion signals.

## Run independent work together

Use the native runner's process/thread/worker support. Concurrent promises overlap asynchronous work but do not parallelize synchronous JavaScript. Test case isolation is a prerequisite for parallel scheduling, not a reason to keep the suite serial forever.

Retain ownership of long jobs through the execution tool's tracked process, logs and final native report. Verify survival after launch; actively monitor until completion. Missing completion evidence means interrupted/unknown. See references/long-running-verification.md for the established host process-lifetime recovery pattern.

Measure preparation, execution, teardown, queueing, first useful failure and the slowest cases separately. Increase workers against representative workloads; consider resource cost and long tails, not only the number of tests. Fix shared state before adding retries or serial groups. Declare a real non-isolatable resource exception explicitly and narrowly.

Reuse immutable builds when their complete inputs match. Do not confuse cached build outputs with a passing test result for a new candidate. Preserve native failure evidence; retries do not turn a reproducible relevant failure into confidence.

## Select verification and assess quality

During changes, run the affected behavior and observed failures first. Keep cheap broad checks when selecting them would cost more. Use integration, platform and release coverage for the actual regression surface and repository policy. Broad expensive coverage may run periodically only when it is not required to establish the current change's behavior.

Investigate the first failure while independent cases finish. Do not rerun passing checks without a relevant change or unresolved concern. A required failure or missing result stays visible. Report which source and environment were tested and which behavior remains untested.

For new regressions, demonstrate that the relevant bad behavior is rejected when practical. Use a bounded negative control when the oracle is uncertain; do not require mutation tooling for every tiny change. Consolidate tests only after comparing trigger, result, failure boundary and platform coverage. Slowness, low assertion count and similar names are not evidence of uselessness.

Use the repository's metadata, fixtures and evidence contract. Tests declare identity, behavior and resources; policy determines release requirements. Unknown impact cannot silently exempt itself.

## Conditional references

- Native runner and language concurrency: references/native-concurrency.md.
- Storage, service and process ownership: references/resource-fixtures.md.
- Timing, profiling and build reuse: references/performance-diagnosis.md.
- Long runner lifetime and completion receipts: references/long-running-verification.md (preserved).
- Rendered CSS requirements and composition: references/mobile-layout-incident.md (preserved).
- Browser input, engine coverage and visual evidence: browser-verification.
