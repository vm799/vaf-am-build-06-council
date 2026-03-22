# BUILD 06 — PRESS PACK ⭐ SHOWPIECE
**VAF AM Series | Parallel Investment Research Council**
*Built with Claude AI + Anthropic Agents | Asset Management*

---

## LINKEDIN POST

**HOOK:**

> What if you had three senior analysts review every investment idea simultaneously?
> Bull. Bear. Risk. All running in parallel. Research note in 60 seconds. 🧵

---

**FULL POST:**

---

Day 3 of building 9 AI systems for asset management this week.

Today's build is the one I'm most excited to show you.

**The Investment Research Council.**

Here's the problem I'm solving:
Investment analysis suffers from confirmation bias.
An analyst builds a thesis. They're invested in it.
The bear case gets weak treatment.
Risk gets reviewed by compliance — after the investment decision.

What if the bear case was built by someone whose only job was to challenge the bull case?

I built that. In Python. Using Claude AI.

**Three agents. Same company. Running at the same time:**

```
asyncio.gather(
    BULL_AGENT.analyse(company),   ← investment thesis
    BEAR_AGENT.analyse(company),   ← challenge everything
    RISK_AGENT.analyse(company),   ← regulatory, ESG, liquidity
)
```

They don't talk to each other.
They run in parallel.
Each one has a different system prompt, a different mandate.

Then a SYNTHESIS agent reads all three and produces a one-page research note.

**Input:** `python run.py --company GSK --context "FDA approval + Q4 beat"`

**Output (60 seconds later):**
```
# Investment Research Note — GSK
Recommendation: 📈 BUY | Conviction: 🔴 HIGH

## Bull Case (HIGH confidence)
[Investment thesis with catalysts...]

## Bear Case (MEDIUM confidence)  
[Challenges and risks...]

## Risk Assessment
[Regulatory, ESG, liquidity risks...]

## Recommendation
BUY | High conviction | [Rationale]
```

This is what AI-native investment research looks like.

Built with:
→ Claude AI (Anthropic) — 4 agents, each with specialist system prompts
→ Python asyncio — true parallel execution
→ Pydantic — structured, typed outputs
→ No external databases, no APIs beyond Claude

The architecture is 150 lines of Python.
The research note is better than most first drafts.
And it took me 3 hours to build.

This is Build 06 of 9 this week.

Tomorrow: AI compliance checker. Fed a letter with 3 FCA violations. It catches all of them.

Full code in comments. Follow for daily builds. 👇

---

**#AssetManagement #AIinFinance #BuildInPublic #ClaudeAI #AnthropicAI #InvestmentResearch #AgenticAI**

---

**POST NOTES:**
- Post time: Wednesday 07:00 GMT
- Attach: terminal screenshot showing BULL + BEAR + RISK launching simultaneously
- This is your highest-engagement post of the week — the parallel agents concept is visually compelling
- Tag @Anthropic

---

## VIDEO SCRIPT ⭐ THIS IS YOUR BEST VIDEO

**Title:** "I Built an AI Investment Research Council — Bull, Bear & Risk Analysts Running in Parallel"
**Length:** 5–6 minutes (this one earns the extra time)
**Format:** Screen recording with voice over + brief face-to-camera intro/outro

---

### [00:00–00:30] FACE TO CAMERA — HOOK

"Confirmation bias is one of the biggest problems in investment research. An analyst builds a thesis, gets invested in it, and the bear case gets weak treatment. Today I'm showing you how I solved that with three AI agents running in parallel — each with an adversarial mandate. Bull. Bear. Risk. All analysing the same company at the same time."

---

### [00:30–01:30] THE ARCHITECTURE

*Show the architecture diagram or code*

"Here's the key design decision. The three agents don't talk to each other before synthesis. They run completely independently — using Python's asyncio.gather. That's what gives this adversarial quality. The bear agent has no idea what the bull agent said. It's purely challenging the investment on its own terms.

I'm using Claude AI from Anthropic for each agent. Each one gets a completely different system prompt — a different identity, a different mandate. The bull analyst's only job is to find reasons to buy. The bear analyst's only job is to challenge those reasons."

---

### [01:30–03:30] LIVE DEMO

*Terminal recording*

"Let me run this live. I'm going to analyse GSK — they had an FDA approval this week and a Q4 earnings beat.

*[type: `uv run python run.py --company GSK --context "FDA approval + Q4 beat"`]*

Watch the terminal. You can see all three agents launching simultaneously. BULL, BEAR, RISK — parallel. Not sequential.

*[show agents completing]*

Now synthesis is compiling the note... and here it is.

*[open the Markdown file]*

Executive summary. Bull case with high confidence. Bear case with medium confidence — notice it's actually challenging the valuation given the post-approval re-rating. Risk assessment flagging the US pipeline concentration.

This is a genuinely useful first-draft research note. In 60 seconds."

---

### [03:30–05:00] THE AM ANGLE + CLOSE

"What does this mean for asset management? First-draft research acceleration. Pre-committee challenge preparation. Coverage initiation on new names. Client reporting with balanced market commentary.

The AI doesn't replace the analyst. It means the analyst spends their time on the second draft — which is actually thinking — not the first draft, which is mostly gathering and structuring.

This is Build 06 of 9 this week. All built with Claude AI and Anthropic's agent patterns. The architecture is open-source. Code link below.

Follow for tomorrow's build: an FCA compliance checker that catches violations your team might miss."

---

## THUMBNAIL BRIEF

**Visual:** Three terminal windows side by side — BULL (green), BEAR (red), RISK (orange)
**Text overlay:** "3 AI Analysts. 1 Company. 60 Seconds."
**Sub-text:** "Built with Claude AI"
**Style:** Dark background, terminal aesthetic, bold typography

---

## RECORDING CHECKLIST
- [ ] Run the demo once before filming — know exactly what the output looks like
- [ ] Have the output file open ready to show
- [ ] Show the parallel execution in terminal clearly — this is the visual proof
- [ ] Include "Built with Claude AI + Anthropic" on screen
- [ ] This video is worth 8–10 minutes if the demo is compelling — don't rush it

---

*VAF AM Series | Vaishali Mehmi | github.com/vm799*
*Built with Claude AI + Anthropic Agents*
