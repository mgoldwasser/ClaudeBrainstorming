---
name: brainstorm
description: Creative brainstorming using a multi-agent swarm. Launches parallel agents with divergent thinking techniques (stoner circle, biomimicry, acid test, etc.) then converges into actionable ideas. Use when generating creative ideas for a problem.
argument-hint: [--technique-names] <problem or topic to brainstorm>
allowed-tools: Agent WebSearch WebFetch
---

# Brainstorming Swarm

You are the **orchestrator** of a creative brainstorming session. Your job is to run a multi-agent swarm where each agent uses a different divergent thinking technique, then converge the results into actionable ideas.

**Start by reading [techniques/README.md](techniques/README.md)** — it's a lightweight index of all 13 available techniques with their tiers, flags, and file paths. Do NOT read the individual technique files yet — only load the ones you actually select in Phase 3.

## How This Works: The Double Diamond

```
    DIVERGE                          CONVERGE
    (Agent Swarm)                    (You, the Orchestrator)
    
    ╱  Agent 1: Stoner Circle  ╲
   ╱   Agent 2: Acid Test       ╲        Cluster
  ╱    Agent 3: Bad Ideas         ╲       Cross-Pollinate
 ╱     Agent 4: Biomimicry         ╲      Evaluate
╱      Agent 5: Expert Panel        ╲     Harvest
Problem ──────────────────────────────── Ideas
```

---

## Phase 1: PARSE

Extract from `$ARGUMENTS`:
1. **The problem statement** — everything that isn't a flag
2. **Technique overrides** (optional) — flags like `--stoner`, `--acid`, `--biomimicry`, `--reverse`, `--caveman`, `--bad-ideas`, `--expert-panel`, `--constraint`, `--time-machine`, `--cross-domain`, `--oblique`, `--scamper`, `--questions`

If the user provided no arguments, ask them what they want to brainstorm.

---

## Phase 2: SELECT TECHNIQUES

Consult [techniques/README.md](techniques/README.md) for the full list of techniques, their flags, tiers, and problem-type guidance.

**If the user specified techniques via flags:** Use those (any number they specified).

**If auto-selecting (default):** Pick 5 techniques following the rules in the index:
- At least 2 S-tier techniques
- At least 1 A-tier technique
- At least 1 research round
- Vary the selection across sessions
- Match techniques to problem type per the guidance in the index

---

## Phase 3: DIVERGE — Launch the Agent Swarm

**This is the critical step.** Launch ALL selected technique agents IN PARALLEL using the Agent tool. Each agent must be completely independent — they should have ZERO context from each other.

**Step 3a — Load the selected techniques:**
Read ONLY the files for the techniques you selected (not all 13). The file paths are in [techniques/README.md](techniques/README.md). For example, if you selected Stoner Circle, read `techniques/stoner-circle.md`.

**Step 3b — Construct each agent's prompt:**
For each selected technique:
1. Take the entire content of the "Agent Prompt" section from that technique's file
2. Replace the `{problem}` placeholder with the user's problem statement
3. Use the resulting string as the `prompt` parameter to the Agent tool
4. Set `description` to the technique name (e.g., "Stoner Circle brainstorm")

**Step 3c — Launch all agents in parallel:**
Make all Agent tool calls in a **single message** so they execute concurrently. Do NOT launch them sequentially — the whole point of the swarm is parallelism and independence.

**Important agent configuration:**
- For **Research rounds** (Biomimicry, Time Machine, Cross-Domain Transfer): The agent will need web search access. These agents should use WebSearch to find real examples.
- For **Imagination rounds**: No web access needed. Pure creative output.
- Each agent should produce **5+ raw ideas**, unfiltered and in the voice/persona of their technique.

**Example of launching 5 agents in parallel (single message, multiple tool calls):**
```
Agent({ description: "Stoner Circle brainstorm", prompt: "<contents of stoner-circle.md agent prompt with {problem} replaced>" })
Agent({ description: "Acid Test brainstorm", prompt: "<contents of acid-test.md agent prompt with {problem} replaced>" })
Agent({ description: "Bad Idea Bonanza brainstorm", prompt: "<contents of bad-idea-bonanza.md agent prompt with {problem} replaced>" })
Agent({ description: "Biomimicry Safari brainstorm", prompt: "<contents of biomimicry-safari.md agent prompt with {problem} replaced>" })
Agent({ description: "Expert Panel brainstorm", prompt: "<contents of expert-panel.md agent prompt with {problem} replaced>" })
```

---

## Phase 4: CONVERGE — Synthesize Results

Once all agents return their results, YOU (the orchestrator) perform the convergence. This is where the real magic happens.

### Step 1: Present the Raw Output

Show the user what each agent produced, organized by technique. Use clear headers:

```
## Round 1: The Stoner Circle
[agent output]

## Round 2: Acid Test
[agent output]
```

### Step 2: Cluster

Group ALL ideas (across all rounds) into thematic categories. Look for:
- Ideas that appeared independently in multiple rounds (strong signal)
- Unexpected themes that emerged across techniques
- Contradictions between rounds (these are often the most interesting)

### Step 3: Cross-Pollinate

This is the highest-value step. Take ideas from DIFFERENT rounds and forcibly combine them:
- "What if [Stoner Circle idea #3] met [Biomimicry idea #7]?"
- "What if we applied [the mechanism from Bad Idea flip #2] to [the constraint identified in Reverse Brainstorm]?"
- Generate at least 5 hybrid/mutant ideas that couldn't exist without multiple techniques contributing.

### Step 4: Evaluate

Rate every idea (originals + hybrids) on two dimensions:
- **Novelty** (1-5): How unexpected/original? Would someone think of this in a normal 5-minute brainstorm?
- **Feasibility** (1-5): How practical to actually implement with current resources?

### Step 5: Harvest — The Final Output

Present the results in this format:

```
## The Harvest

### The Sweet Spot (Novel + Feasible)
Top 3-5 ideas that score high on BOTH novelty and feasibility. These are the gems.

### Moonshots
Top 3 wild ideas that are highly novel but need significant work to be feasible.
Worth pursuing if you're feeling ambitious.

### Quick Wins
Top 3 highly feasible ideas that could be started tomorrow.
Lower novelty but high practical value.

### Idea Log
Full list of all ~30+ ideas organized by source technique, for reference.
```

---

## Rules of Engagement

These rules apply to the ENTIRE session:

1. **No self-censoring.** This is brainstorming. Bad ideas are CELEBRATED. The filter comes later (in convergence), not during divergence.
2. **Quantity over quality during divergence.** More raw material = better synthesis. Each agent should produce at least 5 ideas.
3. **Personality shifts must be DRAMATIC.** Each agent should feel genuinely different — different voice, different energy, different vocabulary. If they all sound like "helpful AI assistant," the technique isn't working.
4. **The synthesis is where you earn your keep.** Don't just cherry-pick the best idea from one round. The cross-pollination step is mandatory and should produce genuinely novel combinations.
5. **Surprise the user.** If every idea is something they could have thought of on their own, the session has failed. At least 30% of ideas should make the user say "I never would have thought of that."
6. **Research rounds should use real data.** When running Biomimicry, Time Machine, or Cross-Domain Transfer, actually use WebSearch to find real examples. Don't make them up.
