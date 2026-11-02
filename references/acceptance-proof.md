# Acceptance proof: the check is the thing the user does

Use this before saying any change is done, verified, tested or working, and whenever the user
says "you said you tested it", "nothing shows up", "how did you verify that", "test the hell out
of it" or "test it overnight". Project testing skills add their entrances; this is the method.

Source: thnkr.ing, 2026-11-02. Four agents called the think-aloud Canvas agent tested. Each had
read a save back from the server or watched the tab that made the change; none watched an
already-open canvas receive an agent's change, and his never did. A study of that project's
failure ledger (about 700 cases, 448 premature done/verified claims) found the same gaps
throughout: a stored record instead of the screen (24), only the acting tab looked at (37), an
isolated copy taken as his path (36), an acknowledgement taken as the effect (26), one action
generalized to all (55), no check at all (57). His words: "How can you come back and say you
tested it when not a single action is reflected on the canvas? What kind of testing mechanism
was that?"

## The oracle

A test oracle is what decides pass or fail (ISO/IEC/IEEE 29119). Write it down before running
anything: the starting state, the actor, the target, the observable result, the deadline. For an
agent action on a page: "the page already open on that board draws the new thread, with its
title, within 2 seconds, without a reload." The audit that missed this wrote its oracle as "a
saved result read back", so it passed a build that never delivered anything.

Label each result with the property it actually established: **saved** (a durable commit with
its identities), **applied** (this client holds it), **presented** (this screen draws it and
the user can act on it). Only presented, on the user's surface, supports "it works".

## Five rules a claim must meet

1. **Observed where the user looks, by something other than the actor**, on a surface that
   existed before the action. Server read-backs, the acting tab, status codes, receipts, "synced",
   exit codes, builds and health checks prove only themselves.
2. **The user's path, hop for hop.** List the hops; name every substitute (isolated copy, mocked
   owner, synthetic input, desktop browser for a phone). A hop nothing crossed is "not checked".
   Run the real component that decides behavior: an agent is checked with the instructions it
   really receives, not a prompt written for the test.
3. **Claimed equals run.** A row per requested action × entrance × size × state, each with its
   own result. One action checked is one action checked.
4. **The states the user lives in**: an already-open page, a second window or device, a follower
   tab of the same browser, reload, a server restart or release with the page left open, offline
   and back, the phone size in WebKit, typing while the change arrives.
5. **It fails first.** Run it against the build that has the bug and see it fail at the named
   assertion; then pass on the fix. Also make sure the check cannot repair what it detects: a
   probe that nudges the page (forces a sync, reloads) must be judged by a window it did not
   nudge.

## When and how much

- **Live, while the user works with you:** one check of the exact reported behavior on the
  user's path, in minutes, plus whatever the release command runs. Say what it did not cover.
- **Unattended (overnight, "test it properly"):** the whole matrix of rule 3 and 4, through the
  real actor (the real agent with the user's kind of words), plus the independent tester.
- **Before and after release:** a machine-run probe for the paths changed, whose pass or fail
  the release command obeys. A sentence typed into a `--checked` field is not that.

Keep the pyramid: fast tests for decisions and contracts where the project allows them, one
end-to-end acceptance check per user journey that matters. The end-to-end check is not optional
because the small tests passed; it is the one that decides "done".

## The independent tester

For a feature the user has reported broken, or any unattended pass, another agent judges the
work. Give it the user's original words, the oracle and rows, and the probe; not the builder's
reasoning. It owns pass/fail and may reject the build. On a fail: keep the counterexample and the
first hop where it diverged; discard the attempt (fresh worktree from main); fix the instruction,
skill or check that let it through when the lesson is reusable; rebuild. Two repair attempts per
failure, then escalate with the evidence (second opinion per the global instructions) rather
than retry blind. The builder may dispute an oracle only through a separately reviewed change.

## What "verified" must carry

The runner, not the agent, writes the record: build or revision, scenario, actor, target ids,
expected and observed result, elapsed time from dispatch, verdict, and where the trace is. Cite
that record in the reply. A missing record, a stale build, a setup error or a pass that does
not reproduce leaves the claim unverified.
