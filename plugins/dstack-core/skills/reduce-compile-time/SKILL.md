---
name: reduce-compile-time
description: 'Use when a local or CI build takes too long and its dependency graph, build units, profile, linker, cache, or debug info need rebuilding or retuning from the toolchain timing report. Not for CI pipeline structure and build minutes: use reduce-ci-time.'
---

# Cut build time from the contract

A build costs too much on the clean run or on the edit-compile-test loop. This skill derives what the build owes its consumers, measures where the time goes, and rebuilds the structure that costs it or corrects the setting that costs it. No move changes observable behavior: the same artifacts come out, faster. The proof is the toolchain's own timing report taken the same way before and after, plus the artifact checks that show the contract still holds.

## Write the contract, then look away

State what the build owes and to whom, from consumer evidence rather than from the build files as they grew. The contract names the artifacts each consumer uses (binaries, libraries, test executables, published packages), the guarantees behind each one (reproducibility, supported targets and toolchain versions, the debug information crash triage needs, the optimization level of what ships), and the loop a developer must get (how fast an edit compiles, links, and runs its tests). Sources are who downloads or links each artifact, which checks protect which guarantee, and how developers actually run the loop. Build files record how the graph grew, not what it owes. A guarantee no consumer uses is residue waiting for a classification; a consumer need the graph ignores is a gap to report.

## Measure, then name the cost

Run the toolchain's own timing report first and keep it as the baseline; wall-clock guesses cannot rank work. The build finishes when its longest dependent chain finishes, not when the total work sums, so find the critical path and name the units on it. Each toolchain note below names its timing command and its graph command.

The report decides between two cases, and the case decides the method:

- Settings case: the graph's shape is sound, and a profile, linker, cache, parallelism, or debug info choice costs the time.
- Rebuild case: the graph's shape is the cost, and no flag fixes it.

For a rebuild case, name the structure: one unit everything waits on, a header or module that fans out, generic or macro expansion repeated per unit, a code generator in the hot path, link time dominated by one oversized unit, or a dependency pulled in for one function.

## Classify every part

Walk the dependency graph and the build configuration. Classify each unit, dependency, feature, and setting with a one-line reason:

- Essential: an artifact or a guarantee needs it. Name which.
- Residue: cut it. An unused dependency or feature, a duplicate build of one unit, a re-export layer, or a profile setting kept from a past problem.

Nothing is kept or cut without a classification.

## Separate boundary from interior

Boundary surfaces each need their own explicit yes before they change:

- public API and package or crate names
- feature flags and build options downstream users set
- the minimum supported toolchain version
- supported targets
- artifact formats and symbols other builds link against
- reproducibility and release-profile guarantees

Present each surface with its class. A no keeps it for good; silence is not a yes; what static inspection cannot resolve counts as boundary. Interior surfaces need no ask: how code splits into units inside those surfaces, internal dependencies, dev and test profiles, and the cache and linker choice for local builds.

## Work one target at a time

Take the measured dominant cost first. Apply one move, rerun the same report, keep the before and after; revert a move whose number did not drop. No move removes a check the contract needs; a check that looks removable is a report to the repository owner, not a change. Finish the target with its proof green and commit it, or revert to the last green state; never leave a half-rebuilt graph.

### The settings case

Reach here only when the measurement clears the structure. A cache or a faster linker laid over a rebuild-case structure hides the cost instead of removing it.

- Turn on incremental and shared caches. Precondition: rebuilds recompile unchanged inputs, locally or across machines. Enable the tool's incremental mode and point it at a shared cache keyed on content, not on time. Check: second-run minutes. Risk: a cache keyed too coarsely reuses stale outputs; key on the inputs and let the tool invalidate.
- Switch to a faster linker. Precondition: the report shows the link step dominating, common on incremental rebuilds. Install mold, lld, or wild and point the toolchain at it. Check: link step minutes. Risk: different linker diagnostics hide a warning class; keep the default linker reachable in one flag change.
- Lower the debug info level. Precondition: debug info generation shows up in compile minutes, and the debugging ahead needs line numbers, not variables. Set line-tables-only or level one, or split debug info out of the object files. Check: compile minutes. Risk: a debug session that needed full info finds less; raise the level for that session only.
- Raise parallelism against memory. Precondition: the report shows idle cores. Raise the tool's job count until cores or memory saturate; past memory, swapping and out-of-memory kills lose more than parallelism wins. Check: wall minutes and peak memory. Risk: an out-of-memory kill mid-build on a CI runner; size the job count to the smallest runner.

### The rebuild case

The graph's shape is the cost. Rebuild the structure instead of patching around it: split the blocking unit from its callers' usage, delete the dependency or the fan-out rather than caching over it, and leave no shim unit that re-exports the old shape.

- Rebuild the blocking unit from its callers. Precondition: one unit sits mid-chain and everything else waits on it. Split it by what each caller actually uses, not along the seams the old file grew; a smaller unit rebuilds less and parallelizes more. Migrate every caller to the unit it needs; no re-export shim keeps the old shape alive. Check: critical-path minutes. Risk: a split leaks internals across the boundary; the build's own visibility rules keep the cut honest.
- Delete the generic or macro fan-out. Precondition: the report shows one unit compiling many instances of the same generic or template. Every distinct instantiation compiles separately, so reduce the instantiated type set to what callers use or make the hot path concrete. Check: unit minutes. Risk: a narrowed type set rejects a real caller; the build answers immediately.
- Delete the dependency. Precondition: the graph shows a module or a feature nothing reads, or a dependency pulled for one function. Turn the feature off and drop the dependency, using each note's graph and tidy commands; a one-function use is replaced by writing the function. Check: total units compiled and minutes. Risk: a dependency or a feature still needed breaks the build or a test; both answer loudly.
- Remove the generator from the hot path. Precondition: a code generator reruns every build though its inputs rarely change. Move generation behind an explicit command the developer runs when inputs change, and keep a check that regenerates and compares so stale output fails. Check: generator minutes leaving the critical path. Risk: stale generated output reaching a consumer; the regenerate-and-compare check catches it in the gate.

## Toolchain notes

Each note names the timing command for the measure step, the graph command for the classify walk, and the flags the moves use.

Rust: `cargo build --timings` writes an HTML report under `target/cargo-timings/`; `cargo tree` prints the dependency graph and `cargo tree -d` lists dependencies compiled in more than one version. Set `debug = "line-tables-only"` in the profile, point `rustflags` at `mold` or `lld` with `-C link-arg=-fuse-ld=...`, and run cargo under `RUSTC_WRAPPER=sccache` for a shared compile cache.

C and C++: Ninja records each command's start and end in `.ninja_log`; `ninjatracing <build-dir>/.ninja_log > trace.json` turns that log into a trace for chrome tracing or speedscope. Use `ccache` through CMake's `CMAKE_C_COMPILER_LAUNCHER` and `CMAKE_CXX_COMPILER_LAUNCHER`, compile with `-g1` or split debug info with `-gsplit-dwarf`, and size `ninja -j N` to memory.

Go: time a cold build and a warm one separately (`go clean -cache`, then `time go build`), because the build cache under `GOCACHE` makes the warm number the honest local figure. `go build -debug-actiongraph=<file>` writes the action graph whose dependency chains name the critical path, and `-p n` caps parallel build commands. `go mod graph` and `go mod why -m <module>` expose unneeded modules and `go mod tidy` removes them. Set `CGO_ENABLED=0` when no cgo is required, since the C toolchain then never runs.

TypeScript: `tsc --extendedDiagnostics` prints where the compiler spends its time, and `tsc --generateTrace <dir>` writes a trace that `@typescript/analyze-trace` reads. Turn on `incremental` for `.tsbuildinfo` reuse, split the program with project references, and let the bundler transpile while `tsc --noEmit` checks types.

JVM: `gradle --profile` writes a per-task report under `build/reports/profile/` and `gradle --scan` opens the full timeline; set `org.gradle.parallel=true`, `org.gradle.caching=true`, and `--max-workers` to memory. Maven prints a per-module reactor summary at the end of every build; `mvn -T 1C` builds modules in parallel.

## When the evidence says stop

- A report showing the build meets its contract ends the pass; the rebuild posture applies to broken structure only.
- A verifier gone red on a target: fix the replacement or revert the target to the last green state; never ship red.
- Residue still resolving (a dependency, feature, or re-export path classified cut): delete it before finishing.
- A boundary surface cut without its own yes: restore it and settle the question.
- No remote, paid, credential, or published change: authority stays as the invocation gave it, and publishing an artifact needs its own explicit request.

## What you hand back

The stated contract, the classification ledger with its one-line reasons, the boundary list with each answer, the diff per target, and the timing reports before and after on a clean build and on the common incremental edit. The artifact checks that show the contract holds go with them: the test suite, and binary size or symbol checks where consumers depend on them. One measured baseline per toolchain, and no move kept on faith.
