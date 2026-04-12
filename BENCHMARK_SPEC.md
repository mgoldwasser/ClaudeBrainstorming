# BrainstormingBench — Build Prompt

**Use this document as a prompt for a fresh Claude Code session.** Open a new empty directory, start `claude`, and paste the contents of this file. Claude should produce a working benchmark repo that can evaluate any brainstorming system (including `brainstorm-kit` from `github.com/mgoldwasser/ClaudeBrainstorming`) on a frozen set of creativity problems.

---

## What you're building

A standalone Python repo — **BrainstormingBench** — that measures the creativity of brainstorming systems. It treats each system as a black box (problem in → list of ideas out) and scores the output on four dimensions from creativity psychology: fluency, flexibility, originality, elaboration. It also runs pairwise LLM-judge battles between systems and produces an Elo leaderboard.

The benchmark must live in its **own repo** and be **independent** of any specific brainstorming tool. Tools are plugged in via an `Adapter` interface.

## Design principles (read these before writing code)

1. **Benchmark ≠ tool.** Nothing in this repo may import or depend on `brainstorm-kit` internals. The `brainstorm-kit` adapter must shell out to Claude Code or replicate the prompt externally.
2. **Freezable.** `problems/v1.yaml` and `judge/rubric_v1.md` are versioned. Once released, they never change — we add `v2.yaml` instead. Comparability across runs matters more than convenience.
3. **Pairwise > absolute.** LLMs are unreliable absolute scorers but decent pairwise judges. The Elo leaderboard is the primary output; absolute metrics are diagnostic.
4. **Blind judging.** Judge prompts never contain system names. Outputs are labeled "A" and "B" with randomized order per comparison.
5. **Different judge than generator.** If a system uses Opus 4.6 to generate, the judge should be Sonnet 4.6 (or vice versa). Never let a model judge its own output family without flagging it.
6. **Cheap to re-run.** A full evaluation of one system on the v1 seed set should cost < $20 and finish in < 30 minutes. If the design drifts above that, flag it and simplify.

## Repo structure to produce

```
BrainstormingBench/
├── README.md
├── pyproject.toml                 # deps: anthropic, pyyaml, numpy, scikit-learn, hdbscan, sentence-transformers, click, rich
├── problems/
│   ├── v1.yaml                    # frozen 30-problem seed set (see below)
│   └── README.md                  # how to add problems (→ v2, not editing v1)
├── adapters/
│   ├── __init__.py
│   ├── base.py                    # Adapter ABC
│   ├── plain_claude.py            # baseline: single Claude call, "brainstorm 10 ideas for X"
│   ├── single_technique.py        # baseline: one hard-coded technique (stoner circle)
│   ├── brainstorm_kit.py          # invokes the brainstorm-kit plugin via claude CLI
│   └── human.py                   # reads pre-written human responses from disk
├── metrics/
│   ├── __init__.py
│   ├── fluency.py                 # count of distinct ideas
│   ├── flexibility.py             # # of semantic clusters via HDBSCAN on embeddings
│   ├── originality.py             # mean pairwise semantic distance; rarity vs corpus
│   └── elaboration.py             # avg tokens per idea, presence of mechanism/example
├── judge/
│   ├── __init__.py
│   ├── rubric_v1.md               # frozen judge rubric
│   ├── pairwise.py                # runs A-vs-B battles, returns winner + CoT reasoning
│   └── elo.py                     # updates Elo ratings from battle results
├── runs/                          # output dir, gitignored except for .gitkeep
│   └── .gitkeep
├── cli.py                         # click CLI: bench run / bench judge / bench report
├── leaderboard.md                 # auto-generated, committed after each official run
└── tests/
    ├── test_adapters.py
    ├── test_metrics.py
    └── test_judge.py
```

## The problem set (problems/v1.yaml)

30 problems across 5 categories, 6 each. Each problem is a realistic brainstorming prompt. The set must be frozen after initial creation.

Categories and example seeds (generate the rest in the same style — diverse, realistic, non-trivial):

```yaml
version: 1
frozen_at: 2026-04-12
problems:
  # --- product ---
  - id: product-01
    category: product
    prompt: "How might a small indie bookstore compete with Amazon?"
  - id: product-02
    category: product
    prompt: "Ways to make a standing desk more useful beyond just standing."

  # --- social / behavioral ---
  - id: social-01
    category: social
    prompt: "How to get people to actually follow through on New Year's resolutions."

  # --- technical / engineering ---
  - id: tech-01
    category: tech
    prompt: "Reduce tail latency in a globally distributed database without adding replicas."

  # --- creative / artistic ---
  - id: creative-01
    category: creative
    prompt: "A novel format for a weekly podcast about science history."

  # --- civic / systemic ---
  - id: civic-01
    category: civic
    prompt: "How to reduce single-occupancy car commuting in a mid-size city."
```

Requirements for the full set:
- Avoid problems with obvious "correct" answers.
- Span short-horizon (tomorrow) and long-horizon (decades) solutions.
- Include at least 3 problems where domain knowledge helps (so research-based techniques can shine).
- Include at least 3 problems where domain knowledge is unhelpful (pure imagination).

## Adapter interface (adapters/base.py)

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass

@dataclass
class Idea:
    text: str          # the idea itself
    origin: str | None # optional: which technique/agent produced it

@dataclass
class Response:
    problem_id: str
    system: str        # adapter name (e.g. "brainstorm-kit@1.0.0")
    ideas: list[Idea]
    raw: str           # full verbatim output, for auditing
    meta: dict         # latency_s, cost_usd, token_counts, etc.

class Adapter(ABC):
    name: str          # immutable identifier including version

    @abstractmethod
    def generate(self, problem: str) -> Response: ...
```

Notes:
- `plain_claude.py` uses the official `anthropic` SDK, `claude-opus-4-6`, adaptive thinking, a simple prompt.
- `brainstorm_kit.py` invokes `claude -p "/brainstorm-kit:brainstorm <problem>"` as a subprocess OR replicates the prompt verbatim via SDK — pick the subprocess path if a local Claude Code install is present, otherwise SDK. Document the choice in the adapter docstring.
- Every adapter must parse its raw output into a clean `list[Idea]` (split on bullets, numbers, sections).

## Metrics (all operate on a `Response`)

1. **Fluency** — count of distinct ideas after dedup (cosine-similarity > 0.85 on sentence embeddings = duplicate).
2. **Flexibility** — number of clusters when running HDBSCAN over idea embeddings. Use `sentence-transformers/all-MiniLM-L6-v2`.
3. **Originality** — two numbers:
   - **Within-response**: mean pairwise cosine distance between this response's ideas.
   - **Corpus-relative**: mean cosine distance from this response's ideas to the nearest neighbor in an "obvious ideas" corpus built from `plain_claude` runs. Higher = more novel vs baseline.
4. **Elaboration** — mean tokens per idea, plus a regex-based presence check for mechanism words ("because", "by", "via", "so that") and example words ("e.g.", "for example", "like"). Report both.

All metrics return floats and are deterministic given the same input.

## LLM judge (judge/pairwise.py)

Pairwise battles. For each problem, for each pair of systems `(A, B)`, run `N=3` battles with randomized A/B order. The judge sees:
- The problem
- Response A (ideas only, not system name)
- Response B (ideas only, not system name)
- The rubric (judge/rubric_v1.md)

Rubric asks the judge to pick a winner on *overall creative value* — defined as the combination of: novelty (would a competent person miss these?), diversity (do the ideas span meaningfully different approaches?), and usefulness (are any actually actionable?). The judge must return structured output:

```json
{
  "winner": "A" | "B" | "tie",
  "reasoning": "...",
  "novelty_winner": "A" | "B" | "tie",
  "diversity_winner": "A" | "B" | "tie",
  "usefulness_winner": "A" | "B" | "tie"
}
```

Use `client.messages.parse()` with a Pydantic schema for structured output. Judge model: `claude-sonnet-4-6` with `output_config: {effort: "medium"}`. Use adaptive thinking.

**Mitigations for known LLM-judge biases:**
- Randomize A/B position per battle (position bias).
- Truncate responses to equal token length before judging (length bias) OR explicitly instruct the judge to ignore length.
- Require CoT reasoning before the verdict (improves calibration — Zheng 2023).
- Run `N=3` battles per pair and take majority vote.

## Elo scoring (judge/elo.py)

Initialize every system at 1500. Process battles in random order. K-factor = 32. Ties count as half-win each. Output sorted leaderboard with 95% bootstrap confidence intervals (resample battles with replacement 1000 times).

## CLI (cli.py)

```
bench run --adapter plain_claude --problems v1 --out runs/plain-claude-2026-04-12/
bench run --adapter brainstorm_kit --problems v1 --out runs/bk-2026-04-12/
bench metrics runs/bk-2026-04-12/
bench judge --a runs/plain-claude-2026-04-12/ --b runs/bk-2026-04-12/ --battles 3
bench report --runs runs/  # regenerates leaderboard.md
```

## Acceptance criteria

The repo is done when:
1. `pytest` passes on all of `tests/`.
2. `bench run --adapter plain_claude --problems v1 --out /tmp/pc/` produces 30 response JSON files.
3. `bench metrics /tmp/pc/` prints per-problem and aggregate numbers for all 4 metric families.
4. `bench judge --a /tmp/pc/ --b /tmp/single/ --battles 3` produces battle results and updates Elo.
5. `bench report` generates a `leaderboard.md` with at least 3 systems ranked.
6. Total cost of steps 2-5 for 3 adapters is < $20 on the v1 set.
7. README explains how to add a new adapter in under 50 lines of code.

## Don't do

- Don't add a web UI.
- Don't add async/streaming adapters beyond what the SDK gives for free — simplicity matters more.
- Don't invent new creativity metrics — stick to the four above. Notes on what you'd add next go in `FUTURE.md`.
- Don't import or depend on `brainstorm-kit` source. Adapter talks to it externally only.
- Don't commit API keys or cached judge outputs with sensitive prompts.

## Anthropic SDK usage notes

- Model: `claude-opus-4-6` for generation adapters, `claude-sonnet-4-6` for the judge. Exact strings, no date suffixes.
- Use adaptive thinking: `thinking={"type": "adaptive"}`. Do not pass `budget_tokens`.
- Use `output_config={"effort": "medium"}` for the judge; default (`high`) for generators.
- Structured judge output: `client.messages.parse(...)` with a Pydantic model.
- Stream any call with `max_tokens > 4096`.

## Deliverable

A single git repo, pushable to `github.com/<user>/BrainstormingBench`, that an outside reviewer can clone, `pip install -e .`, set `ANTHROPIC_API_KEY`, and run end-to-end without further instructions.

---

**Stop and ask before proceeding if:** the seed problems need refinement, the judge rubric seems too vague, or the scope feels wrong for the time budget. Otherwise, build it.
