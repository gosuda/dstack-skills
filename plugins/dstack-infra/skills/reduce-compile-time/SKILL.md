---
name: reduce-compile-time
description: 'Use when a local or CI build takes too long and the build profile, dependency graph, linker, cache, or debug info settings need changing from the tool timing report. Not for CI pipeline structure and build minutes: use reduce-ci-time.'
---

# Cut build time from a profile

A compile dominates the day or the CI bill. This skill reads the toolchain's own timing report, finds the dependency chain that sets the floor, and changes config, dependencies, and flags until the number drops. Edits stay in build configuration, dependency lists, and linker settings; no source behavior changes. Apply one move, rerun the timing report, and keep the before and after; revert a move whose number did not drop.

## The core, any toolchain

- Measure with the tool's own report. Every toolchain note below names its timing command; run it first and keep the output as the baseline. Wall-clock guesses cannot rank work.
- Find the critical path of the dependency graph. The build finishes when its longest dependent chain finishes, not when the total work sums. Name the units on that chain; they are the first candidates for every move below. Each note names its graph command.
- Split the wide unit. Precondition: one unit sits mid-chain and everything else waits for it. Split it along its existing internal boundaries; a smaller unit rebuilds less and parallelizes more. Check: chain minutes in the timing report. Risk: a split leaks internals across the boundary; the existing module lines keep the cut honest.
- Cut generic or template instantiation. Precondition: the report shows one unit compiling many instances of the same generic or template. Every distinct instantiation compiles separately, so fewer instantiated types or generic-free implementations shrink the unit. Check: unit minutes. Risk: a narrowed type set rejects a real caller; the build answers immediately.
- Remove unused dependencies and features. Precondition: the graph shows modules or feature flags nothing reads. Drop the dependency and turn the feature off, using each note's graph and tidy commands. Check: total units compiled and minutes. Risk: a dependency or feature still needed breaks the build or a test; both answer loudly.
- Turn on incremental and shared caches. Precondition: rebuilds recompile unchanged inputs, locally or across machines. Enable the tool's incremental mode and point it at a shared cache keyed on content, not on time. Check: second-run minutes. Risk: a cache keyed too coarsely reuses stale outputs; key on the inputs and let the tool invalidate.
- Switch to a faster linker. Precondition: the report shows the link step dominating, common on incremental rebuilds. Install mold, lld, or wild and point the toolchain at it. Check: link step minutes. Risk: different linker diagnostics hide a warning class; keep the default linker reachable in one flag change.
- Lower the debug info level. Precondition: debug info generation shows up in compile minutes, and the debugging ahead needs line numbers, not variables. Set line-tables-only or level one, or split debug info out of the object files. Check: compile minutes. Risk: a debug session that needed full info finds less; raise the level for that session only.
- Raise parallelism against memory. Precondition: the report shows idle cores. Raise the tool's job count until cores or memory saturate; past memory, swapping and out-of-memory kills lose more than parallelism wins. Check: wall minutes and peak memory. Risk: an out-of-memory kill mid-build on a CI runner; size the job count to the smallest runner.

## Toolchain notes

Each note names its timing command, its graph command, and the flags the core names, verified against the tool's documentation.

Rust: `cargo build --timings` writes an HTML report under `target/cargo-timings/`; `cargo tree` prints the dependency graph and `cargo tree -d` lists dependencies compiled in more than one version. Set `debug = "line-tables-only"` in the profile, point `rustflags` at `mold` or `lld` with `-C link-arg=-fuse-ld=...`, and run cargo under `RUSTC_WRAPPER=sccache` for a shared compile cache.

C and C++: Ninja records each command's start and end in `.ninja_log`; `ninjatracing <build-dir>/.ninja_log > trace.json` turns that log into a trace for chrome tracing or speedscope. Use `ccache` through CMake's `CMAKE_C_COMPILER_LAUNCHER` and `CMAKE_CXX_COMPILER_LAUNCHER`, compile with `-g1` or split debug info with `-gsplit-dwarf`, and size `ninja -j N` to memory.

Go: time a cold build and a warm one separately (`go clean -cache`, then `time go build`), because the build cache under `GOCACHE` makes the warm number the honest local figure. `go build -debug-actiongraph=<file>` writes the action graph whose dependency chains name the critical path, and `-p n` caps parallel build commands. `go mod graph` and `go mod why -m <module>` expose unneeded modules and `go mod tidy` removes them. Set `CGO_ENABLED=0` when no cgo is required, since the C toolchain then never runs.

TypeScript: `tsc --extendedDiagnostics` prints where the compiler spends its time, and `tsc --generateTrace <dir>` writes a trace that `@typescript/analyze-trace` reads. Turn on `incremental` for `.tsbuildinfo` reuse, split the program with project references, and let the bundler transpile while `tsc --noEmit` checks types.

JVM: `gradle --profile` writes a per-task report under `build/reports/profile/` and `gradle --scan` opens the full timeline; set `org.gradle.parallel=true`, `org.gradle.caching=true`, and `--max-workers` to memory. Maven prints a per-module reactor summary at the end of every build; `mvn -T 1C` builds modules in parallel.

## What you hand back

The config diff, the before and after timing reports, and the moves kept with their numbers. One measured baseline per toolchain, and no move kept on faith.
