---
layout:     post
title:      "The orchestrator already exists: an unattended app rebuild on Gitea Actions"
subtitle:   "14 issues, 15 merged PRs and 98 green tests in one night of unsupervised work — the CI pipeline as agent dispatcher, labels as the only state, a kanban as the view, and local GLM as both developer and judge"
date:       2026-11-05 09:00:00 +0100
author:     "Daniel Vela"
locale:     en
---

In October I rebuilt LiShop, my local-first shopping-list PWA, semi-attended: I wrote the spec, a strong model decomposed it into tasks, and small local agents built each task while I reviewed and committed every result. It worked, but I was the merge gate, the scheduler and the night shift. The obvious question was: what part of that loop is actually irreplaceable?

So I ran the experiment. Same app, fresh repository, same task list — but this time the orchestrator would be the one piece of infrastructure I already trust to run unattended jobs for years: **CI**. No agent framework, no dispatcher daemon, no state files on disk. Gitea Actions, a runner on my workstation, and a set of five small pipelines. Setup took an hour; I launched the first cycle at 22:00 and went to bed. At 05:45 the last merge landed: **14 of 14 tasks done, 15 PRs merged with CI green and QA approval, 98 tests passing, roughly 7h45m of wall time, zero token invoices.**

This post is how it works, what broke, and what stayed human.

## The idea: replace the job payload, not the orchestrator

CI systems already solve the hard parts of unattended work: scheduling, concurrency limits, retries, logs, a web UI, and a place where every execution is auditable. Agent orchestrators reinvent all of that, worse. So the pipelines keep the CI shell and change what the job executes:

| Classic CI | This setup |
|---|---|
| job runs tests | job runs an **agent** that changes code |
| cron job | **dispatcher** pipeline that reads the issue board and spawns work |
| flaky test retry | **QA judge** pipeline that approves or rejects the PR |
| deploy gate | **merge gate** pipeline that squashes when everything is green |
| nightly report | **report** pipeline that pushes progress to a fixed issue |

The state lives where CI can't lose it: **in the forge**. One issue = one branch `feat/issue-N` = one worktree = one MR. Issue labels carry the lifecycle: `backlog → ready → in-progress → review → approved → done`, with `blocked` as the human escalation. There is no local state directory to corrupt; the issue board *is* the database, and every transition is a label change with a comment trail.

## The five pipelines

**`cp` (the worker).** Dispatched with an issue number. It fetches the issue and its comments, prepares a worktree, and runs `pi --print` with a prompt that contains the task, the feedback so far, and the rules. The key trick is session identity: the job runs with `--session-id cp-issue-N`, so **a new job resumes the old conversation**. When QA rejects or CI goes red, the same session wakes up with the failure in front of it — the model remembers what it tried, because the transcript survived in the runner's disk. The job ends by pushing the branch, opening the MR and setting `review`; if the agent instead ends with `NEEDS_HUMAN: <question>`, the pipeline comments the question on the issue and parks the issue as `blocked`.

**`cp-board` (the dispatcher).** A cron pipeline every 20 minutes that replaces my old dispatcher daemon: issues in `ready` whose declared dependencies are `done` get dispatched to `cp`; issues in `review` with a push newer than the last QA verdict get `cp-qa`; issues in `approved` get `cp-merge`. It also syncs the kanban board (more on that below). Concurrency is not configured anywhere — it is the runner's capacity, physically.

**`cp-qa` (the judge).** Runs a second agent — headless, stateless — over the PR diff, the issue's acceptance criteria, and **its own previous verdicts on that issue** (the verdict history is just the issue comments). It can only emit `VEREDICTO: approve` or `VEREDICTO: reject — <numbered reasons>`; the pipeline applies the consequences: approve → `approved`; reject → feedback comment, back to the worker (via a fresh `cp` dispatch that resumes the session), and after two rejected cycles → `blocked` with the reasons.

**`cp-merge` (the gate).** Only touches issues in `approved`. Conflict with main → back to the worker (session resume again, with the conflict comment in context). CI red → back to the worker **with a condensed digest of the actual failure** (the pipeline greps the failing job's log and posts it as a comment — the agent never has to guess why the build broke). CI green → squash merge, delete branch, `done`.

**`cp-report` (the push channel).** Every morning, a comment on a fixed issue: what merged in the last 24h, what is blocked, what is in flight, what is queued. This exists because of a lesson from my first unattended factory: *I stop watching systems that don't show me progress*. The progress comes to me.

There is also `cp-setup` (labels + kanban project bootstrap) and `ci` (plain tests on every push, the hard gate the merge pipeline reads).

## The kanban is a view, with one exception

Gitea projects gave me a board with columns, but nothing moves cards automatically. So the dispatcher syncs the board each cycle from the labels: the card's column follows the issue's status. The **one exception is deliberate**: if a human drags a card into the *Ready* column, that drag is interpreted as intent — the sync promotes the label to `status:ready` instead of fighting it. That is my entire manual interface: **I drag cards to Ready to say "start this", and everything else is automatic.** (Implementation detail worth stealing: Gitea's column-issue endpoints take the issue's *internal* ID, not its index, and the column listing doesn't return the index — you need an id↔number map.)

## What broke, and what it taught

The night produced a list of Gitea-specific lessons that cost real debugging hours at 2 AM:

- **Commit statuses use `.status`, not `.state`.** The GitHub field name crept into my merge gate; Gitea silently returned `null` and the gate would have merged over red CI.
- **Gitea emits a spurious status per workflow per push** (`cp-merge.yml (push)`), which fails when the workflow has required inputs. Those poison the combined status forever — the fix is to only count contexts starting with `ci /`.
- **`pull_request` events never fired** on my instance, so CI now triggers on *push* to any branch and checks out `$GITHUB_SHA` explicitly — which also means the CI workflow file that runs is the one on the worker's branch; infrastructure fixes must be cherry-picked into open branches or the fix never applies.
- **`exit 141` (SIGPIPE) with `pipefail`.** `git diff | head -c 60000` killed the producer once diffs exceeded the cap, and my own failure-digest code did the same with `jq | head`. Limit inside the tool (`[...] | first // empty`), never with `head`.
- **Job containers run with `bash --noprofile`** — the runner's service environment needs the full PATH (node, pi) or every job fails in the first second. And **restarting the runner cancels running jobs**, which leaves issues stuck in `in-progress` unless the dispatcher re-queues them.
- **QA rejection must re-dispatch the worker.** In the old daemon the QA ran synchronously inside the loop; as a pipeline it is asynchronous, and without the explicit re-dispatch the issue just sits in `review` forever.

None of these are agent problems. They are all *interface* problems between two systems that mostly agree — exactly the kind of thing that only shows up in a real overnight run.

## What the system did without me

Three things surprised me, in a good way:

1. **It resolved merge conflicts with main, repeatedly.** My own infrastructure fixes (I couldn't stop improving the pipelines mid-flight) landed on main while workers had branches open; the merge gate bounced them, the sessions merged `origin/main` into their branches and re-pushed, with commit messages explaining what they kept and why.
2. **It adapted its tests to the real CI environment.** A test that passed locally failed in CI (node 24 vs node 26 quirk with `node --test test/`); the digest comment told the worker what failed and the fix came back as an explicit glob, with the commit message stating the reason.
3. **One worker out-engineered me.** The task I gave it was "build the daily report board". It noticed the status-poisoning bug (the same one I had patched hours earlier), wrote a shared jq filter with last-status-per-context semantics, added an acceptance test for it, and wired the merge gate through the filter. Its version was better than my patch, so I adopted it and reverted mine.

## What this does not prove

- **One repo, one night, one run.** No repetitions, no second codebase, no different failure profile.
- **The target was known-good.** The 14 tasks came from an app that already worked; workers rebuilt toward a spec that a previous implementation had already validated. A greenfield spec would stress the planner (the part that writes the issues) much harder than the builders.
- **The judge was not stronger than the builder.** Grok ran out of credits mid-night, so QA fell back to the same local GLM family as the developer. The hard gate was CI; the QA agent was a criteria-checker, not an oracle. With the planned opencode-frontier judge the QA leg gets real teeth, and I intend to measure whether that changes rejection patterns.
- **The tasks were atomic by construction.** Dependencies were hand-written into the issues (`Depends on #N`). A planner that writes those automatically is the untested half.

## Reproducibility

The rebuilt app lives in a private Gitea (`lishop2`, on the tailnet — the old repo with the manual history is `lishop-web`). The pipelines are copies of the templates in my SWE repo, now with every fix from the night backported: `cp.yml`, `cp-board.yml`, `cp-qa.yml`, `cp-merge.yml`, `cp-report.yml`, `cp-setup.yml`, `ci.yml`, plus `deploy/kanban-sync.sh` and its acceptance test. The runner is `act_runner` in host mode (jobs run as my user, with pi, the GPU gateway and the session disk available; label `tesla`, capacity 3). Worker and QA ran on self-hosted GLM-5.3-Flash behind the local gateway; the night's entire LLM bill was zero — which, after reading Revolut's numbers, still feels like the main result.

The boring conclusion is the one I had hoped for: **you don't need a new orchestrator — you need a contract the forge can hold.** Labels for state, issues for memory, CI for legs, kanban for the one thing you still want to touch by hand.
