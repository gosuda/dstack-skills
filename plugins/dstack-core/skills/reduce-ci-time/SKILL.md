---
name: reduce-ci-time
description: 'Use when CI wall time, queue wait, or build minutes are too high and the pipeline needs measured tuning or a contract-first rebuild. Not for a red run needing repair: use ci-fix. Not for authoring a pipeline or its gates: use ci-cd.'
---

# Cut CI time from the merge contract

A pipeline costs more wall time than the work behind it, or its build minutes grow every month. This skill shortens one pipeline against the verdicts merge protection, release gates, and readers of a red run need. It derives the change from evidence, tunes a sound structure in place, and rebuilds the job graph when the structure is the cost. A required gate changes only on its own explicit yes. Edits stay in workflow files and build scripts, and version control rolls everything back.

## Derive the contract

Write what the pipeline owes before reading its shape. Read the provider's required-check or branch-protection settings, the release and publish gates, and the failures that actually blocked a merge or release. The workflow files are evidence for the walk below, not the source of the contract.

For each contract item, record what it proves, who needs the verdict, which change or release it must block, and whether it is required, optional, or observational. Also record how soon a contributor must get the first useful signal from an explicit team or repository rule; when no rule names one, put the cheapest independent proof first and measure that signal directly. A check that looks unused, duplicated, or too slow is still contract until its own answer says otherwise.

## Measure the old pipeline

Read per-job and per-step durations from the provider's own data, never from a stopwatch. On GitHub Actions, `gh run view <run-id> --json jobs` returns every job with `startedAt` and `completedAt`, and the jobs API record adds the same timestamps for every step inside the job (docs.github.com/en/rest/actions/workflow-jobs). On GitLab, the jobs API carries duration and queued duration per job (docs.gitlab.com/api/jobs).

Record four numbers before touching anything: total run time split into queue wait and execution, the per-job table, the per-step breakdown of the heaviest jobs, and the time to the first job whose result a contributor can use. Compare like with like: same provider capacity, comparable change, and the most recent accepted run as the rolling baseline. The original run reports the total improvement; a later slowdown cannot pass by still beating that old run.

Find the critical path: the chain of jobs whose durations and queue waits add up to the finish time. The slowest job is not always on it; a quick job that every later job waits behind often is. Name the structure that costs the measured time: a serial chain, a duplicated build, a matrix nobody reads, a cache that never hits, or a job that proves what another job already proves.

## Walk every workflow, job, and step

Walk the whole pipeline once and classify every workflow, job, matrix cell, and step with a one-line reason:

- Essential: it proves a named contract item; name the item it proves and keep it.
- Residue: it proves no contract item; name the kind: duplicate proof, obsolete job, dead matrix cell, or a step kept from a past incident with no owner.

Nothing is kept or cut without its classification. Several essential items may share one build or shard when equivalence is proved from inputs and outputs. A residue candidate that is required, named outside the repository, or cannot be resolved by static inspection is not cut; it joins the boundary list.

## Separate interior from boundary

Interior covers job layout, ordering, caching, sharding, path filters, and fan-out inside the same set of proven properties. Interior changes need no ask, but a path filter is interior only while every contract item still runs for every change it protects.

Each boundary surface needs its own explicit yes or no after it is presented with its class and cost:

- Every required or protected check whose removal, rename, merge, trigger narrowing, or advisory state changes what protects the branch or merge request.
- Every release, publish, deploy, migration, or rollback job.
- Every security, license, provenance, or compliance scan.
- Every check a branch-protection rule, auditor, or other repository names.
- Every secret, token permission, runner permission, self-hosted runner contract, environment approval, or protected variable.
- Every surface static inspection cannot resolve, including reusable workflow calls, generated pipelines, and externally triggered jobs.

A no keeps that surface for good. Silence is never a yes. A broad instruction to do all of it is no answer for a boundary surface.

## Choose tuning or rebuild

Tune in place when the graph proves the contract with a sound shape and the measurements point at one slow step or one bounded mechanism: one cache that can be keyed correctly, one serial test suite that can shard, one duplicated download, or one setup step that can use a pinned binary.

Rebuild the job graph when the shape is the cost: serial stages hide independent proofs, several jobs rebuild the same inputs, matrix cells spend time proving properties nobody consumes, queue wait grows because the graph asks for more runner slots than it needs, or residue dominates the run. Then derive the replacement from the contract: build each distinct configuration once, fan out the independent proofs it feeds, run the cheapest useful proof first, and keep only one artifact for each distinct build output. Add no adapter that preserves the old graph inside the new one, and keep no second pipeline for comfort. The only allowed overlap is one comparison run or reviewable report that proves parity before the old definition is removed.

Evidence that the structure already meets the contract ends the pass with the measurements. Do not manufacture a rebuild to spend a speed request.

## Make the moves in this order

For a tuning case, apply one qualifying move at a time, rerun the pipeline, and record the numbers. For a rebuild, apply the derived graph as one atomic target. In either case, judge the move by the wall time of the whole run, plus queue wait when it adds jobs, and by minutes consumed when that is the cost under repair. Reject a move that speeds one step and slows the run.

Each move names its precondition, its check, and its risk. Skip a move whose precondition is false and check the next one. A cache loses when restore and save cost more than the work it saves; fan-out loses when the new jobs wait for runners.

- Cache dependencies on the lockfile. Precondition: dependency install or restore dominates a job. Key the cache on the lockfile hash (`actions/cache` with `hashFiles('**/package-lock.json')`; GitLab `cache:key: files`), with a fallback key so close matches restore partially. Check: install step minutes, before and after. Risk: a cache that outlives its lockfile restores stale dependencies; the lockfile hash in the key bounds that, and a scheduled key reset clears drift.
- Restore build outputs between runs. Precondition: compile work repeats unchanged across runs. Upload or pull the tool's build cache (Gradle build cache, ccache or sccache archives, the provider's artifact store). Check: compile minutes across two runs of the same tree. Risk: a stale cache hides a source change; key the cache on its inputs, never on the branch name.
- Filter paths. Precondition: jobs run for changes they cannot observe. Keep a shared workflow unfiltered and gate its optional jobs with changed-file detection and job-level `if`, so required jobs still report on every change. Use a workflow-level filter only on a separate workflow whose every job can safely be skipped: `paths` or `paths-ignore` on one event, never both on the same event (GitHub), or `rules:changes` (GitLab), and include the workflow file itself plus generated and fixture directories the job reads. Check: number of jobs for a docs-only and a code-only change, plus the required check results on both. Risk: a filter too narrow skips a needed gate; every filtered-out job must still run for the paths that feed it, and a skipped workflow leaves its required checks unreported. Any change to a required trigger is boundary.
- Fan jobs out against runner limits. Precondition: one job holds serial work with no ordering constraint, or the critical path sits behind queued jobs. Split the work into parallel jobs or a matrix, sized so the runner concurrency quota absorbs them; beyond the quota, fan-out adds queue wait instead of removing it. Check: critical-path minutes and queue wait, before and after. Risk: an exhausted runner quota queues the same pipeline slower than before; measure queue wait next to run time.
- Shard tests from timing data. Precondition: the test job sets the critical path. Split the suite into shards balanced by measured per-suite duration, not by file count, and re-shard when suite timing changes. Check: slowest-shard minutes, before and after. Risk: imbalance concentrates wall time in one shard; the shard timing exposes it.
- Order gates fail-fast. Precondition: cheap checks run after heavy build work. Put lint and type checking first so a broken change stops in minutes; the heavy gates still run and still block. Check: minutes spent on runs that end in a cheap failure. Risk: ordering only, because no gate left the pipeline; a gate that must run first for correctness keeps its place.
- Remove duplicate work between jobs. Precondition: two jobs produce equivalent outputs from the same inputs: same sources and generated inputs, same target platform and architecture, same toolchain, same dependency set, same profile, features, and flags. Build once, upload the artifact, download it downstream, and run a shared check once. Keep one artifact per distinct build configuration; jobs with different outputs keep their own builds. Check: duplicated step minutes across jobs. Risk: an artifact hand-off fails silently, so the downstream job rebuilds or uses nothing; the download step must hard-fail when the artifact is missing, and a present but incompatible artifact is caught only by comparing the full build configuration, not by the download succeeding.
- Cancel superseded runs. Precondition: a run sits abandoned mid-flight when the next push arrives. Key each workflow's group by workflow identity and branch or pull request identity (`concurrency` with `group: ${{ github.workflow }}-${{ github.ref }}` and `cancel-in-progress` on GitHub; auto-cancel redundant pipelines with `interruptible: true` on GitLab), since a branch-only group collides across workflows and one workflow's run cancels the other's. Separate event types whose runs are not substitutes for each other into different groups. Check: aggregate run minutes across a day of pushes. Risk: a needed long run gets canceled, so deployments and release runs stay outside the cancelable group.
- Install pinned tools instead of building them. Precondition: a job compiles a toolchain or tool every run. Install the prebuilt artifact at a pinned version and cache it against that version. Check: setup step minutes, before and after. Risk: a floating version drifts between runs; pin exact versions in the workflow file.

## Prove parity and the delta

Prove every contract item still has a verdict path. For each essential gate, plant one failure that the gate exists to catch and observe the new pipeline turn red on an isolated or reviewable run. Where the provider offers no safe planted-failure path, parity is unproven: restore the last green pipeline and report the missing proof rather than substituting a weaker check. A planted failure that stays green fails parity the same way.

Measure the changed pipeline the same way as the old one on comparable runs: wall time, queue wait, build minutes, first useful signal, and the per-job and per-step tables. Finish the target or revert it to the last green state; a half-old, half-new graph is not a result. Never erase a required validation to make the proof pass, and take no remote, credential, paid, or published change unless the user explicitly asked for that separate act.

## What you hand back

The stated contract, the measured baseline and critical path, the classification table, each boundary surface with its yes or no, and the tuning or rebuild verdict with its reason. Hand back the edited workflow files, the before and after numbers for every applied move from the provider's own report, the parity evidence for every essential gate, and the accepted risks with their bounds. A move whose numbers did not improve is reverted, not explained.
