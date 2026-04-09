# Brainstorming Techniques Index

Each technique lives in its own file. The orchestrator reads this index to select techniques, then loads ONLY the selected technique files to pass their agent prompts to subagents.

## Tier Rankings

Techniques are tiered by how much they break Claude out of default convergent-thinking mode:

| Tier | Meaning |
|------|---------|
| **S** | Maximally divergent — forces genuinely unusual thinking |
| **A** | Proven creativity frameworks with strong forced-perspective shifts |
| **B** | Reliable but more structured |
| **C** | Valuable but more analytical than wildly creative |

## Auto-Selection Rules

When auto-selecting 5 techniques, the orchestrator should:
- Include at least 2 S-tier techniques
- Include at least 1 A-tier technique
- Include at least 1 research round
- Vary the selection across sessions — don't always pick the same 5
- Match techniques to problem type (see guidance below)

## The 13 Techniques

| # | Technique | Flag | Type | Tier | File | Energy |
|---|-----------|------|------|------|------|--------|
| 1 | The Stoner Circle | `--stoner` | Imagination | S | [stoner-circle.md](stoner-circle.md) | Chill, vibes-only, zero judgment |
| 2 | Bad Idea Bonanza | `--bad-ideas` | Imagination | A | [bad-idea-bonanza.md](bad-idea-bonanza.md) | Chaotic, celebratory, gleefully destructive |
| 3 | The Expert Panel | `--expert-panel` | Imagination | A | [expert-panel.md](expert-panel.md) | Heated debate, diverse perspectives clashing |
| 4 | Acid Test | `--acid` | Imagination | S | [acid-test.md](acid-test.md) | Psychedelic, synesthetic, stream-of-consciousness |
| 5 | Caveman First Principles | `--caveman` | Imagination | B | [caveman.md](caveman.md) | Grunting, primal, radically simple |
| 6 | Reverse Brainstorm | `--reverse` | Imagination | B | [reverse-brainstorm.md](reverse-brainstorm.md) | Devious, strategic, saboteur-thinking |
| 7 | Constraint Removal | `--constraint` | Imagination | C | [constraint-removal.md](constraint-removal.md) | Expansive, what-if, progressively wilder |
| 8 | Oblique Strategies | `--oblique` | Imagination | S | [oblique-strategies.md](oblique-strategies.md) | Cryptic, zen-like, laterally provocative |
| 9 | SCAMPER | `--scamper` | Imagination | A | [scamper.md](scamper.md) | Systematic, methodical, checklist-powered |
| 10 | Question Explosion | `--questions` | Imagination | B | [question-explosion.md](question-explosion.md) | Socratic, deconstructive, assumption-busting |
| 11 | Biomimicry Safari | `--biomimicry` | Research | B | [biomimicry-safari.md](biomimicry-safari.md) | Scientific wonder, nature-as-genius |
| 12 | Time Machine | `--time-machine` | Research | C | [time-machine.md](time-machine.md) | Scholarly, pattern-matching across eras |
| 13 | Cross-Domain Transfer | `--cross-domain` | Research | A | [cross-domain-transfer.md](cross-domain-transfer.md) | Surprising connections, intellectual mashup |

## Problem-Type Matching Guidance

Different problem types benefit from different techniques:

- **Technical problems** → SCAMPER, Biomimicry, Caveman (first principles often reveals hidden assumptions in engineering)
- **People/culture problems** → Expert Panel, Question Explosion, Stoner Circle (social dynamics need diverse perspectives)
- **Product/design problems** → Bad Idea Bonanza, Cross-Domain Transfer, Acid Test (forced weird angles often produce the best product ideas)
- **Strategic problems** → Time Machine, Reverse Brainstorm, Constraint Removal (big-picture thinking)
- **Stuck/deadlocked** → Oblique Strategies, Acid Test (when you need to break out of a mental loop)

## File Format

Each technique file contains:
1. A header with: Type, Tier, Energy, Flag
2. An "Agent Prompt" section containing the full, ready-to-use prompt with a `{problem}` placeholder

The orchestrator reads the file, replaces `{problem}` with the user's problem statement, and passes the result directly to a subagent.
