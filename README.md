# ClaudeBrainstorming

A Claude Code skill that turns brainstorming into a multi-agent swarm. Launches parallel agents — each running a different divergent-thinking technique (stoner circle, biomimicry, acid test, SCAMPER, etc.) — then converges the raw output into a ranked harvest of actionable ideas.

Invoke it with `/brainstorm <problem>` or let Claude trigger it automatically when you ask for creative ideas.

## Installation

Claude Code skills are just files in a `.claude/skills/<name>/` directory — no build step, no package manager. Pick the scope you want:

### Option 1 — Global (all your projects)

Install to your personal skills folder so it works everywhere on your machine:

```bash
mkdir -p ~/.claude/skills
cp -r .claude/skills/brainstorm ~/.claude/skills/
```

Or, if you want updates from this repo to flow automatically, symlink it:

```bash
mkdir -p ~/.claude/skills
ln -s "$(pwd)/.claude/skills/brainstorm" ~/.claude/skills/brainstorm
```

Then `git pull` in this repo to get new techniques or fixes.

### Option 2 — Per-project (only this repo)

The skill is already at `.claude/skills/brainstorm/` in this repo, so anyone who clones this project gets it automatically when they open Claude Code here. Nothing to install.

To add it to a different project you already have checked out:

```bash
mkdir -p /path/to/other-project/.claude/skills
cp -r .claude/skills/brainstorm /path/to/other-project/.claude/skills/
```

Commit `.claude/skills/brainstorm/` into that project's repo if you want it shared with the team.

### Verify it's installed

In Claude Code, run:

```
/brainstorm how to make meetings less painful
```

Or ask Claude "what skills are available?" — you should see `brainstorm` in the list.

## Usage

```
/brainstorm <problem or topic>
```

By default, Claude auto-selects 5 techniques (biased toward the most divergent). You can override:

```
/brainstorm --stoner --biomimicry --reverse how to reduce user churn
```

Available technique flags: `--stoner`, `--acid`, `--bad-ideas`, `--expert-panel`, `--caveman`, `--reverse`, `--constraint`, `--oblique`, `--scamper`, `--questions`, `--biomimicry`, `--time-machine`, `--cross-domain`.

See [.claude/skills/brainstorm/techniques/README.md](.claude/skills/brainstorm/techniques/README.md) for the full technique index with tier rankings and problem-type guidance.

## How it works

1. **Parse** — extracts the problem and any technique overrides from your input
2. **Select** — picks 5 techniques (auto or user-specified)
3. **Diverge** — spawns 5 parallel agents via the `Agent` tool, each in full character (chill stoner, scientific biomimicry researcher, psychedelic acid-tripper, etc.). Each produces 5+ raw ideas.
4. **Converge** — clusters, cross-pollinates (forcibly combines ideas from different rounds), evaluates on novelty × feasibility, and presents a harvest: Sweet Spot / Moonshots / Quick Wins.

The multi-agent architecture matters: each agent starts with zero context from the others, so no anchoring bias. You get genuinely diverse output instead of five variations on the first idea.

## Making your own skill installable

Same pattern works for any skill you build:

1. Create `.claude/skills/<your-skill>/SKILL.md` with YAML frontmatter (`name`, `description`, optional `argument-hint`, `allowed-tools`)
2. Add supporting files in the same directory and reference them from SKILL.md
3. Commit to a repo for per-project use, or copy/symlink to `~/.claude/skills/` for global use

Full spec: [code.claude.com/docs/en/skills](https://code.claude.com/docs/en/skills).
