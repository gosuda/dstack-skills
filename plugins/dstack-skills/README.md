# DStack Skills

DStack workflows for authoring, auditing, benchmarking, and publishing agent skills.

6 skills, category Productivity. Install the whole plugin below, or take a single skill with `gh skill install`.

## Install

```bash
# Claude Code
/plugin marketplace add gosuda/dstack-skills
/plugin install dstack-skills@dstack

# Codex
codex plugin marketplace add gosuda/dstack-skills
codex plugin add dstack-skills@dstack

# Hermes Agent
hermes plugins install gosuda/dstack-skills/plugins/dstack-skills --enable

# Antigravity
agy plugin install https://github.com/gosuda/dstack-skills
agy plugin install dstack-skills@dstack
```

### Individual (gh skill)

```shell
gh skill install gosuda/dstack-skills plugins/dstack-skills/skills/<skill> \
  --agent <agent> --scope user
```

`<agent>` is `claude-code`, `codex`, `cursor`, `grok`, `kimi-cli`, or `antigravity`.

Cursor, Grok, Kimi, and Antigravity install from this same tree. The repository README gives each command.

## Skills

Each row states when to reach for the skill. Invoke one as `/dstack-skills:<name>` in Claude Code, `$<name>` in Codex, `dstack-skills:<name>` in Hermes Agent, or type `/` and pick it in Cursor.

| Skill | Trigger |
|---|---|
| cascade-dedup | Use when prompt-doctrine is duplicated, drifted, or conflicting across output-style embeds and external harness AGENTS files. |
| dedup-skills | Use when asked to deduplicate a skill tree, fold overlapping skills, or cut the skill count: analyze, gate per family, then fold. |
| generate-my-taste | Use when asked to generate a personal taste skill from local evidence. |
| model-challenge | Use when a user requests an independent Codex or Gemini review of uncommitted code, a branch diff, or a specific commit. |
| model-retuning | Use when asked to run /model-retuning to retune a skill corpus for a new model, measurement-first. |
| review-plugin-submission | Use when asked to review a plugin for marketplace readiness via a read-only audit of published quality gates. |

## Workflows

[docs/guides/workflows.md](../../docs/guides/workflows.md) shows how these skills chain with the rest of the tree.
