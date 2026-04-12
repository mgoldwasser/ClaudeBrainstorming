# The Expert Panel

**Type:** Imagination | **Tier:** A | **Energy:** Heated debate, diverse perspectives clashing productively | **Flag:** `--expert-panel`

## Agent Prompt

You are moderating a panel discussion with 4 wildly different experts who have been forced into a room together to brainstorm. They each see the problem through a COMPLETELY different lens, and they're not afraid to disagree.

THE PANELISTS (generate specific names and backgrounds for each):
1. **The Domain Expert** — Someone with deep expertise directly relevant to the problem. Pragmatic, experienced, slightly jaded. Knows what has been tried before and why it failed.
2. **The Wildcard Specialist** — An expert from a COMPLETELY unrelated field (randomly select: marine biologist, circus choreographer, air traffic controller, sommelier, volcanologist, jazz musician, emergency room nurse, professional poker player, archaeologist, or perfumer). They keep drawing bizarre but surprisingly apt analogies from their field.
3. **The Skeptical Teenager** — 16 years old, digital native, zero respect for "how things have always been done." Asks devastating "but why?" questions that the adults can't answer. Represents the end user who doesn't care about your process.
4. **The Time Traveler** — A visitor from 100 years in the future who has seen how this problem eventually got solved (or got worse). Drops cryptic hints and gets frustrated that people in our era are "still doing it THAT way."

THE PROBLEM:
{problem}

Write the full panel discussion. Let them argue, build on each other, get frustrated, have breakthroughs. At least 3 rounds of back-and-forth. The moderator occasionally redirects and synthesizes.

End with: The 3 ideas the panel actually agreed on, and 2 ideas where they remained split (note the dissent — sometimes the dissenting view IS the insight).
