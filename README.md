# DStack Skills

Agent skills for agentic engineering, installable as plugin modules in Claude Code, Codex, Cursor, Grok, Kimi Code, and Hermes Agent.

Each skill is a `SKILL.md` file authored once under `plugins/<plugin>/skills/<slug>/`; every supported
agent reads that same tree, so a fix lands everywhere without a second copy or a registry release.
Install the base plugins, then invoke skills by name:

```text
/dstack-core:ask-dalcom I want to add a feature to this repository. Where do I start?
```

`ask-dalcom` answers with one route: which skills, in which order, what each step does, and what must
exist first. It recommends and runs none of it.
308 skills in 22 plugins.

## Why use it

- Skills are organized by working domain; install `dstack-core` and `dstack-code`, then add only the domains you use.
- One authored tree serves six agents through fixed manifests, so skills do not drift between copies.
- Skills ship as reviewable Markdown with named triggers; nothing is fetched from a package registry at install time beyond the repository itself.
- `dstack-core` includes six output styles and four MCP server configurations; `ask-dalcom` routes you to the right skill when you do not know the name.
- The maintenance scripts are dependency-free Node ESM and standard-library Python; `just check` runs every tree gate.

## Quick start

Install the base plugins in Claude Code:

```shell
claude plugin marketplace add gosuda/dstack-skills
claude plugin install dstack-core@dstack
claude plugin install dstack-code@dstack
```

Run `claude plugin list` to verify; both entries show enabled:

```text
❯ dstack-code@dstack
  Version: 2.1.19
  Scope: user
  Status: ✔ enabled

❯ dstack-core@dstack
  Version: 2.1.19
  Scope: user
  Status: ✔ enabled
```

Then invoke the router skill shown above, or pick a skill from the
[skill index](docs/specs/skill-index.md); install its plugin first when it lives
outside the two base plugins (`claude plugin install <plugin>@dstack`; other agents under Installation).

## Installation

Every surface installs from this repository; the manifests live at the repository root and at
`plugins/<plugin>/.<agent>-plugin/plugin.json`. The evidence for each surface is in
[docs/specs/distribution-surfaces.md](docs/specs/distribution-surfaces.md), and recorded install runs
are in [docs/specs/install-proof.md](docs/specs/install-proof.md).

### Claude Code

```shell
claude plugin marketplace add gosuda/dstack-skills
claude plugin install dstack-core@dstack
claude plugin install dstack-code@dstack
```

`claude plugin install <plugin>@dstack` adds any other plugin module, for example
`dstack-planning`, `dstack-run`, or `dstack-knowledge`.

### Codex

```shell
codex plugin marketplace add gosuda/dstack-skills
codex plugin add dstack-core@dstack
codex plugin add dstack-code@dstack
```

Codex resolves the plugin list from `.claude-plugin/marketplace.json` (the generated
`.codex-plugin/marketplace.json` is a forward-compatible copy); each plugin's own
`.codex-plugin/plugin.json` is what Codex loads first, including the core MCP configuration path.

### Cursor

```shell
cursor-agent plugin marketplace add github.com/gosuda/dstack-skills
```

Cursor indexes the repository and reads `.cursor-plugin/marketplace.json`. Install plugins from the
marketplace in the Cursor plugin UI.

### Grok

```shell
grok plugin marketplace add gosuda/dstack-skills
grok plugin install gosuda/dstack-skills#plugins/dstack-core
```

The `#plugins/<plugin>` suffix selects one plugin subdirectory; repeat the install command for each
plugin you want.

### Kimi Code

Kimi reads a marketplace registry from a local clone and requires absolute paths:

```shell
git clone https://github.com/gosuda/dstack-skills.git
```

```text
/plugins marketplace /absolute/path/to/dstack-skills/.kimi-plugin/marketplace.json
/plugins install /absolute/path/to/dstack-skills/plugins/dstack-core
```

### Hermes Agent

Each plugin is also a native Hermes plugin; install the modules you want straight from their
subdirectories:

```shell
hermes plugins install gosuda/dstack-skills/plugins/dstack-core --enable
hermes plugins install gosuda/dstack-skills/plugins/dstack-writing --enable
```

Hermes loads their skills as `<plugin>:<slug>` entries. Its install-time scanner reads every Markdown
file as if it were executable, so `dstack-native` scans as dangerous on its security teaching examples
and cannot be installed into Hermes; take its skills through `gh skill install` instead.

### One skill with gh skill

The GitHub CLI's `gh skill` installs a single skill for many agents. Passing the exact path skips a
full tree traversal:

```shell
gh skill install gosuda/dstack-skills plugins/dstack-core/skills/askme \
  --agent claude-code --scope user
```

Supported `--agent` values include `claude-code`, `codex`, `cursor`, `grok`, and `kimi-cli`;
`gh skill install --help` lists the full set.

## Usage

Invoke a skill by name in the form your agent uses:

```text
/dstack-core:ask-dalcom CI is red on my PR
```

| Agent | Invocation shape | Example |
|---|---|---|
| Claude Code | `/<plugin>:<slug>` | `/dstack-core:ask-dalcom` |
| Codex | `$<slug>` | `$ask-dalcom` |
| Cursor | `/` then pick the skill by name | `/ask-dalcom` |
| Hermes Agent | `<plugin>:<slug>` | `dstack-core:ask-dalcom` |

Grok and Kimi load the same skill tree; use each agent's own invocation form. Skill names are exact:
a near miss loads nothing. A skill can also load automatically when your request matches the trigger
sentence in its `description` field.

## Plugins

| Plugin | Category | Plugin | Category |
|---|---|---|---|
| dstack-core | Coding | dstack-knowledge | Productivity |
| dstack-planning | Productivity | dstack-learn | Productivity |
| dstack-code | Coding | dstack-skills | Productivity |
| dstack-review | Coding | dstack-writing | Writing |
| dstack-testing | Coding | dstack-visual | Design |
| dstack-git | Coding | dstack-design | Design |
| dstack-run | Coding | dstack-typescript | Coding |
| dstack-infra | Infrastructure | dstack-native | Coding |
| dstack-security | Security | dstack-rust | Coding |
| dstack-fuzzing | Security | dstack-performance | Coding |
| dstack-research | Research | dstack-marketing | Marketing |

`catalog/plugins.json` is the identity ledger: it holds each plugin's name, description, category,
and tags, and every manifest and registry is generated from it. Every plugin manifest and every
marketplace entry the harnesses read ships at one `releaseVersion`, currently 2.1.19, held in
`catalog/plugins.json`; the `"2"` at the top of `.kimi-plugin/marketplace.json`
is the registry schema version, not the release. The inert `.codex-plugin/marketplace.json`
forward-compat copy carries no per-entry version; Codex reads `.claude-plugin/marketplace.json`.

The largest plugins are `dstack-core` at 89, `dstack-marketing` at 28, and `dstack-run` at 20.

## Output styles

Six output styles ship in `dstack-core` and shape how the agent communicates. Switch through Claude
Code's `/config` or by setting `outputStyle` in `settings.json`: `DStack`, `AxiomMode`, `Builder`,
`Duet`, `Linus`, and `Eval`. Every style embeds a byte-identical copy of `system-prompt-baseline.md`;
`scripts/sync-baseline.py` owns that copy.

## Documentation

- [Getting started](docs/guides/getting-started.md): install the base plugins and run one skill.
- [Using skills](docs/guides/using-skills.md): invocation forms, stacking, and choosing between overlapping skills.
- [Workflows](docs/guides/workflows.md): skill chains mined from real sessions.
- [Skill index](docs/specs/skill-index.md): every skill with its trigger sentence.
- [Distribution surfaces](docs/specs/distribution-surfaces.md): what each agent requires and the evidence behind it.
- Methodology: [dstack](https://github.com/gosuda/dstack); project site: [outlinedriven.github.io](https://outlinedriven.github.io).

## Development

There is no package manager here. Every script is dependency-free Node ESM or standard-library
Python.

```shell
just              # list tasks
just render       # regenerate skill manifests, plugin manifests, registries, skill index, and the doctrine cascade
just check        # every gate
just verify       # every gate plus Agent Skills validation
```

`.pre-commit-config.yaml` is the gate list. `docs/specs/gates.md` explains what each generator
writes, what each gate proves, and how to read a failure.

### Contributing

Maintainer workflow lives in [CONTRIBUTING.md](CONTRIBUTING.md): how to publish, run gates, and
change skills. Agent-behavior rules live in [AGENTS.md](AGENTS.md).

## License

Apache License 2.0; see [LICENSE](LICENSE). Third-party attribution is in
[licenses/NOTICE](licenses/NOTICE).

Issues: https://github.com/gosuda/dstack-skills/issues
