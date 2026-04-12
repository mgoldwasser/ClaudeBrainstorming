# ClaudeBrainstorming

A Claude Code plugin that turns brainstorming into a multi-agent swarm. Launches parallel agents — each running a different divergent-thinking technique (stoner circle, biomimicry, acid test, SCAMPER, etc.) — then converges the raw output into a ranked harvest of actionable ideas.

This repo doubles as both a **plugin** (`brainstorm-kit`) and a **marketplace** (`brainstorming`) that hosts it.

## Installation

Pick one of three install paths depending on how much you want to commit.

### Option 1 — Marketplace install (recommended)

The closest thing Claude Code has to `pip install`. Inside a Claude Code session:

```
/plugin marketplace add mgoldwasser/ClaudeBrainstorming
/plugin install brainstorm-kit@brainstorming
```

That registers this repo as a marketplace and installs the `brainstorm-kit` plugin from it. The plugin is available in every Claude Code session afterward, and `/plugin marketplace update` will pull new techniques/fixes when you want them.

Invoke it with the namespaced skill name:

```
/brainstorm-kit:brainstorm how to reduce user churn
```

> Plugin skills are always namespaced (`<plugin>:<skill>`) to avoid conflicts between plugins.

### Option 2 — Local plugin (for development or private use)

Clone this repo and point Claude Code at the plugin directory:

```bash
git clone https://github.com/mgoldwasser/ClaudeBrainstorming
claude --plugin-dir ./ClaudeBrainstorming/plugins/brainstorm-kit
```

Same namespaced invocation as Option 1.

### Option 3 — Standalone skill (no plugin, unnamespaced)

If you don't want the plugin namespace and just want `/brainstorm <problem>` directly, copy the skill files to your personal skills folder:

```bash
git clone https://github.com/mgoldwasser/ClaudeBrainstorming /tmp/cb
cp -r /tmp/cb/plugins/brainstorm-kit/skills/brainstorm ~/.claude/skills/
rm -rf /tmp/cb
```

Or symlink to keep it updated via `git pull`:

```bash
git clone https://github.com/mgoldwasser/ClaudeBrainstorming ~/src/ClaudeBrainstorming
ln -s ~/src/ClaudeBrainstorming/plugins/brainstorm-kit/skills/brainstorm ~/.claude/skills/brainstorm
```

Then invoke without the namespace:

```
/brainstorm how to reduce user churn
```

### Verify it's installed

In a Claude Code session, ask "what skills are available?" — you should see `brainstorm-kit:brainstorm` (plugin install) or `brainstorm` (standalone install) in the list.

## Usage

```
/brainstorm-kit:brainstorm <problem or topic>
```

(Or `/brainstorm <problem>` if you installed standalone.)

By default, Claude auto-selects 5 techniques biased toward the most divergent. Override with flags:

```
/brainstorm-kit:brainstorm --stoner --biomimicry --reverse how to reduce user churn
```

Available technique flags: `--stoner`, `--acid`, `--bad-ideas`, `--expert-panel`, `--caveman`, `--reverse`, `--constraint`, `--oblique`, `--scamper`, `--questions`, `--biomimicry`, `--time-machine`, `--cross-domain`.

See [plugins/brainstorm-kit/skills/brainstorm/techniques/README.md](plugins/brainstorm-kit/skills/brainstorm/techniques/README.md) for the full technique index with tier rankings and problem-type guidance.

## How it works

1. **Parse** — extracts the problem and any technique overrides from your input
2. **Select** — picks 5 techniques (auto or user-specified)
3. **Diverge** — spawns 5 parallel agents via the `Agent` tool, each in full character (chill stoner, scientific biomimicry researcher, psychedelic acid-tripper, etc.). Each produces 5+ raw ideas.
4. **Converge** — clusters, cross-pollinates (forcibly combines ideas from different rounds), evaluates on novelty × feasibility, and presents a harvest: Sweet Spot / Moonshots / Quick Wins.

The multi-agent architecture matters: each agent starts with zero context from the others, so no anchoring bias. You get genuinely diverse output instead of five variations on the first idea.

## Repository structure

```
ClaudeBrainstorming/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace catalog (lists brainstorm-kit)
├── plugins/
│   └── brainstorm-kit/
│       ├── .claude-plugin/
│       │   └── plugin.json       # Plugin manifest
│       └── skills/
│           └── brainstorm/
│               ├── SKILL.md      # Orchestrator skill
│               └── techniques/   # 13 technique agent prompts + index
└── README.md
```

## Making your own plugin installable

Same pattern works for any skill you want to distribute:

1. Create `<your-plugin>/.claude-plugin/plugin.json` with `name`, `description`, `version`
2. Put skills under `<your-plugin>/skills/<skill-name>/SKILL.md`
3. Create `.claude-plugin/marketplace.json` at the repo root listing your plugin(s) with a relative `source` path
4. Push to GitHub. Users install with `/plugin marketplace add <owner>/<repo>` then `/plugin install <plugin>@<marketplace>`

Full spec:
- Skills: [code.claude.com/docs/en/skills](https://code.claude.com/docs/en/skills)
- Plugins: [code.claude.com/docs/en/plugins](https://code.claude.com/docs/en/plugins)
- Marketplaces: [code.claude.com/docs/en/plugin-marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)
