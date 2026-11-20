# DStack Performance

DStack workflows for measured performance: hardware kernels, profilers, byte budgets, and optimization proof.

8 skills, category Coding. Install the whole plugin below, or take a single skill with `gh skill install`.

## Install

```bash
# Claude Code
/plugin marketplace add gosuda/dstack-skills
/plugin install dstack-performance@dstack

# Codex
codex plugin marketplace add gosuda/dstack-skills
codex plugin add dstack-performance@dstack

# Hermes Agent
hermes plugins install gosuda/dstack-skills/plugins/dstack-performance --enable

# Antigravity
agy plugin install https://github.com/gosuda/dstack-skills
agy plugin install dstack-performance@dstack
```

### Individual (gh skill)

```shell
gh skill install gosuda/dstack-skills plugins/dstack-performance/skills/<skill> \
  --agent <agent> --scope user
```

`<agent>` is `claude-code`, `codex`, `cursor`, `grok`, `kimi-cli`, or `antigravity`.

Cursor, Grok, Kimi, and Antigravity install from this same tree. The repository README gives each command.

## Skills

Each row states when to reach for the skill. Invoke one as `/dstack-performance:<name>` in Claude Code, `$<name>` in Codex, `dstack-performance:<name>` in Hermes Agent, or type `/` and pick it in Cursor.

| Skill | Trigger |
|---|---|
| cpu-kernel-authoring | Use when writing, optimizing, or benchmarking a C++ CPU kernel with AVX2 or AVX512 intrinsics for the Hugging Face kernels ecosystem. |
| decoded-memo-retention | Use when a query-scoped decoded-block cache is bounded only by serialized work bytes, or when fallback paths retain many decoded blocks. |
| fastopt | Use when a suspected hot path needs read-only hypotheses and benchmark designs, without waiting for benchmarks and without a code change. |
| fastopt-extreme | Use when estimating hot paths and proposing complexity-neutral cold simplifications while refusing complexity theater. |
| heaptrack | Use when tracking heap allocations on Linux with heaptrack: allocation hotspots, peak heap, temporary allocations, leaks, or a diff of two runs via heaptrack_print. |
| optimize | Use when one named path or performance symptom needs a single measured fix. |
| optimize-extremely | Use when a named subsystem must be made as fast as possible through a measured-floor campaign. |
| simd-intrinsics | Use when reading auto-vectorization reports, writing SSE2, AVX2, or NEON intrinsics, or fixing vectorization failures. |

## Workflows

[docs/guides/workflows.md](../../docs/guides/workflows.md) shows how these skills chain with the rest of the tree.
