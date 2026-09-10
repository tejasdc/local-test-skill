# Native scheduling

Read the installed runner's version and configuration before changing its pool.
Choose the smallest native scheduling unit whose writable state is independent.

| Runner | Useful concurrency boundary | Ownership constraint |
|---|---|---|
| Node `node:test` | Files run in isolated processes; `{concurrency:true}` overlaps asynchronous cases inside a file | Same-process cases share globals, environment and module instances. Use test-context cleanup and owned resources. |
| Playwright | `fullyParallel` admits individual tests to reusable worker processes; `workers` controls the pool | Use test-scoped mutable fixtures. A worker-scoped browser or immutable bundle is safe; a shared resettable database is not. |
| Vitest | Files use the configured pool; `test.concurrent` overlaps cases | Cleanup and assertions must belong to the case context. A file-global array drained by `afterEach` can delete another case's resources. |
| Go | `t.Parallel` schedules independent tests; subtests retain their parent's lifecycle | `t.TempDir` and `t.Cleanup` own resources. Do not combine parallel cases with process-global environment mutation. |
| Nextest | Individual Rust tests run in processes with configurable resource groups | A resource group is for a measured shared constraint, not a substitute for isolation. |
| pytest-xdist | Workers distribute test items; load scheduling balances the tail | Session fixtures exist once per worker, not once globally. Partition writable state by worker and case. |

JavaScript promises overlap I/O; they do not run synchronous computation on more
CPU cores. Use runner processes or actual worker threads for CPU-bound work.
Do not add a custom thread pool when the native runner already schedules it.

Discover generated tests using the native runner where supported. Preserve parameter
values and hierarchical titles in identities. Source declarations alone cannot prove
runtime cardinality. Record conditional skips and excluded external probes explicitly.

Exercise discovery in a clean checkout before treating it as a planning primitive.
Load compiled runtime modules inside fixture setup or test bodies, so listing cases
does not require running their build or allocating mutable resources. Thinkering's
first clean release candidate on September 10, 2026 exposed module-level imports
that worked only because the development checkout already had compiled output.
Moving those imports into native setup preserved the tests and made cold discovery pass.

Sources: [Node test runner](https://nodejs.org/api/test.html),
[Playwright parallelism](https://playwright.dev/docs/test-parallel),
[Vitest performance](https://vitest.dev/guide/improving-performance),
[Go testing](https://pkg.go.dev/testing), [Nextest design](https://nexte.st/docs/design/how-it-works/),
[pytest-xdist distribution](https://pytest-xdist.readthedocs.io/en/stable/distribution.html).
The concurrent-cleanup example was reproduced and corrected in Thinkering on September 10, 2026.
