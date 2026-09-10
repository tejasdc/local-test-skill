# Long-running verification

Prefer the execution tool's tracked process when it can retain the runner for
the whole suite. If that runner is terminated while its app server stays alive,
record the partial run as interrupted, preserve its failures, and stop only the
server processes belonging to that run before restarting.

On Linux, a detached process session can retain a test runner beyond its launching
tool call. Use a temporary shell script in the workspace's sanctioned script
directory. It must run the exact test command, capture its exit status immediately,
and write that status to a run-specific file. For example, inside the script:

```bash
npm run test:e2e -- --reporter=line,json
test_exit_status=$?
printf '%s\n' "$test_exit_status" > /absolute/run/path/exit-status
exit "$test_exit_status"
```

Launch that script with `setsid nohup bash /absolute/script/path`, redirect all
three standard streams, and background it. Preserve its PID and log under the
same run-specific directory. `setsid` separates it from the launching process
group; `nohup` ignores hangup. Use a fresh run directory to avoid stale receipts.

Verify the PID and advancing log in a **subsequent tool call**. A PID printed by
the launch command alone does not prove the process survived shell cleanup.
Keep actively polling process state, log progress and the exit-status file; a
detached process does not excuse ending the user turn before verification finishes.
Honor the host's wakeup policy if available and keep user updates within the
normal communication cadence.

Completion requires the exit-status receipt and the test runner's final report.
Treat a missing receipt or unfinished report as interrupted/unknown even if all
cases printed so far passed. Investigate reported failures normally; never use
detachment or retries to conceal them. Cancellation must target this run's known
PID/process group, never a broad name pattern shared with other agents.

Source: chess-with-friends reliability repair, 2026-09-10. A tracked suite ended
with SIGTERM at 45/62 while its local Worker remained alive. A plain background
launch then returned a PID but left an empty log and no live process. A detached
session with subsequent-call PID verification retained the replacement runner.
The termination source was not established; this is a process-lifetime recovery
pattern, not a claim that Wrangler caused those terminations.
