# Diagnose the critical path

Measure elapsed preparation, build, queueing, setup, body, teardown and collection
separately. Summed case durations are CPU/work totals, not time to release. Preserve
the first failed assertion timestamp separately from case/run completion.

Use one frozen source, dependency set and workload for a concurrency sweep. Record
host CPU topology, quotas, memory, load and competing jobs. Compare medians and the
slow tail. If higher worker counts fail, inspect the failure and profile before
attributing it to RAM or cores. Shared fixture corruption, expensive application
rendering and runner contention require different corrections.

Count simultaneous pools, including independent agent jobs. Per-runner worker limits
multiply when browser, storage and HTTPS suites each launch a pool. Coordinate their
admission in the existing orchestrator while keeping native cases parallel; record
queue time separately and do not invent build dependencies to serialize resources.
Thinkering's September 10 clean-gate probe measured 0.17% CPU idle with overlapping
pools and another release job, whereas its bounded ten-worker sample passed. That
is evidence to coordinate work, not evidence that all tests require fewer workers.

Separate a large-history performance workload from representative accessibility
scans and ordinary functional cases. Retain exact search/edit/persistence behavior
and the large workload's evidence. Reducing data in every case would erase the scale
regression; running full-page axe on thousands of repeated controls may obscure it.

Cache deterministic build outputs first. Bind source files, configuration, compiler,
dependencies, generated inputs and output-affecting environment. Validate outputs
before reuse, remove deleted-source outputs and compare incremental with a clean
build after changing the cache. A cached build is not a current test pass. Never
write through an immutable release's build directory.

Prefer supported incremental compilation over a bespoke language/framework rewrite.
Schedule independent producers together; consumers wait only for their own inputs.
The focused command wrappers must use the same graph, or development remains serial
even when CI is faster.

Sources: [TypeScript incremental](https://www.typescriptlang.org/tsconfig/incremental.html),
[Gradle build cache](https://docs.gradle.org/current/userguide/build_cache.html),
[Bazel test encyclopedia](https://bazel.build/reference/test-encyclopedia).
Thinkering's September 10, 2026 investigation separated a passing 2,000-note search
from a WebKit accessibility-teardown timeout and identified eager drag registration.
