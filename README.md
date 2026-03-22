# BUILD 06 — Parallel Investment Research Council
**VAF AM Series | Day: Wednesday | Build Time: ~3-4 hours**
*Built with Claude AI + Anthropic Agents | Asset Management*
⭐ **SHOWPIECE BUILD OF THE WEEK**

---

## WHAT THIS BUILDS
Three parallel AI agents — Bull, Bear, Risk — analyse the same company simultaneously. A synthesis agent compiles a professional one-page research note. One CLI command. 60 seconds. Full investment council output.

## AM PAIN POINT SOLVED
Research takes days. Investment committees need balanced perspectives. Confirmation bias is endemic. This gives every analyst three adversarial views before they've had their second coffee.

---

## FILE STRUCTURE
```
BUILD_06_COUNCIL/
├── README.md
├── .env.example
├── pyproject.toml
├── run.py                    ← ENTRYPOINT: python run.py --company GSK
├── src/
│   ├── agents/
│   │   ├── __init__.py
│   │   ├── base_agent.py
│   │   ├── bull_agent.py
│   │   ├── bear_agent.py
│   │   ├── risk_agent.py
│   │   └── synthesis_agent.py
│   ├── council.py            ← orchestrates all 4 agents
│   ├── formatter.py          ← Markdown output
│   ├── models.py
│   └── config.py
├── skills/
│   └── parallel_agents/SKILL.md
├── reports/research/         ← output goes here
├── tests/
│   ├── test_council.py
│   └── test_formatter.py
└── press_pack/
    ├── LINKEDIN_POST.md
    ├── VIDEO_SCRIPT.md
    └── THUMBNAIL_BRIEF.md
```

---

## FULL SOURCE CODE

### pyproject.toml
```toml
[project]
name = "vaf-am-build-06"
version = "1.0.0"
requires-python = ">=3.11"
dependencies = [
    "anthropic>=0.40.0",
    "pydantic>=2.10.0",
    "pydantic-settings>=2.7.0",
    "python-dotenv>=1.0.0",
]
[project.optional-dependencies]
dev = ["pytest>=8.3.0", "pytest-asyncio>=0.24.0"]
```

### run.py
```python
"""
VAF AM Build 06 — Parallel Investment Research Council
Built by Vaishali Mehmi using Claude AI + Anthropic Agents
github.com/vm799 | Asset Management Series

Usage: uv run python run.py --company GSK --context "Q4 earnings beat"
"""
import asyncio
import argparse
from pathlib import Path
from src.council import InvestmentCouncil
from src.models import ResearchInput

def parse_args():
    p = argparse.ArgumentParser(description="VAF Investment Research Council")
    p.add_argument("--company",  required=True, help="Company name e.g. GSK")
    p.add_argument("--ticker",   default=None,  help="Ticker e.g. GSK.L")
    p.add_argument("--sector",   default=None,  help="Sector e.g. Pharma")
    p.add_argument("--context",  default="",    help="Additional context")
    return p.parse_args()

async def main():
    args = parse_args()

    print("\n╔══════════════════════════════════════════════════╗")
    print("║   VAF Investment Research Council                ║")
    print("║   Built with Claude AI + Anthropic Agents       ║")
    print("╚══════════════════════════════════════════════════╝\n")

    research_input = ResearchInput(
        company=args.company,
        ticker=args.ticker,
        sector=args.sector,
        context=args.context,
    )

    council = InvestmentCouncil()
    note = await council.analyse(research_input)

    output_dir = Path("reports/research")
    output_dir.mkdir(parents=True, exist_ok=True)
    from datetime import date
    filename = f"{date.today().isoformat()}_{args.company.replace(' ', '_')}.md"
    output_path = output_dir / filename
    output_path.write_text(note.full_markdown, encoding="utf-8")

    print(f"\n✅ Research note saved: {output_path}")
    print("\n" + "━" * 50)
    print(note.full_markdown[:500] + "...\n[Full note in file]")

if __name__ == "__main__":
    asyncio.run(main())
```

### src/models.py
```python
from datetime import datetime
from pydantic import BaseModel

class ResearchInput(BaseModel):
    company: str
    ticker: str | None = None
    sector: str | None = None
    context: str = ""

class AgentAnalysis(BaseModel):
    agent_name: str       # "BULL" | "BEAR" | "RISK"
    analysis: str
    key_points: list[str]
    confidence: str       # "high" | "medium" | "low"
    generated_at: datetime = None

    def __init__(self, **data):
        if 'generated_at' not in data:
            data['generated_at'] = datetime.utcnow()
        super().__init__(**data)

class ResearchNote(BaseModel):
    company: str
    generated_at: datetime
    bull_case: AgentAnalysis
    bear_case: AgentAnalysis
    risk_assessment: AgentAnalysis
    recommendation: str   # "BUY" | "HOLD" | "SELL" | "UNDER REVIEW"
    conviction: str       # "high" | "medium" | "low"
    full_markdown: str = ""
```

### src/agents/base_agent.py
```python
from abc import ABC, abstractmethod
import anthropic
from ..models import ResearchInput, AgentAnalysis

class BaseAnalystAgent(ABC):
    def __init__(self, client: anthropic.AsyncAnthropic):
        self.client = client

    @property
    @abstractmethod
    def agent_name(self) -> str: ...

    @property
    @abstractmethod
    def system_prompt(self) -> str: ...

    async def analyse(self, input: ResearchInput) -> AgentAnalysis:
        print(f"[{self.agent_name}] Starting analysis of {input.company}...")
        context = f"Company: {input.company}"
        if input.ticker:  context += f" ({input.ticker})"
        if input.sector:  context += f" | Sector: {input.sector}"
        if input.context: context += f"\nContext: {input.context}"

        resp = await self.client.messages.create(
            model="claude-sonnet-4-5",
            max_tokens=800,
            system=self.system_prompt,
            messages=[{
                "role": "user",
                "content": (
                    f"{context}\n\n"
                    "Provide your analysis. End with a JSON block:\n"
                    '{"key_points": ["point1", "point2", "point3"], "confidence": "high|medium|low"}'
                ),
            }],
        )

        text = resp.content[0].text
        key_points = ["See full analysis"]
        confidence = "medium"
        try:
            import json, re
            m = re.search(r'\{[^{}]+\}', text, re.DOTALL)
            if m:
                parsed = json.loads(m.group())
                key_points = parsed.get("key_points", key_points)
                confidence  = parsed.get("confidence", confidence)
                text = text[:m.start()].strip()
        except Exception:
            pass

        print(f"[{self.agent_name}] ✓ Analysis complete | Confidence: {confidence}")
        return AgentAnalysis(
            agent_name=self.agent_name,
            analysis=text,
            key_points=key_points,
            confidence=confidence,
        )
```

### src/agents/bull_agent.py
```python
from .base_agent import BaseAnalystAgent

class BullAgent(BaseAnalystAgent):
    agent_name = "BULL"
    system_prompt = """You are a buy-side equity analyst making the investment case.
Your role: identify the strongest bull case — catalysts, competitive moats, valuation support,
growth drivers, and upside scenarios. Be rigorous and specific with data where possible.
Your analysis will be challenged by an adversarial bear analyst — make it defensible.
Structure: [Investment Thesis] [3 Key Catalysts] [Valuation Support] [Upside Case]
Be concise — 200-300 words maximum."""
```

### src/agents/bear_agent.py
```python
from .base_agent import BaseAnalystAgent

class BearAgent(BaseAnalystAgent):
    agent_name = "BEAR"
    system_prompt = """You are a skeptical short-seller analyst challenging the bull case.
Your role: identify the strongest bear case — execution risks, competitive threats,
valuation concerns, earnings quality issues, and downside scenarios.
Be specific — vague concerns are not useful. Quantify where possible.
Structure: [Core Challenge] [3 Key Risks] [Valuation Concern] [Downside Case]
Be concise — 200-300 words maximum."""
```

### src/agents/risk_agent.py
```python
from .base_agent import BaseAnalystAgent

class RiskAgent(BaseAnalystAgent):
    agent_name = "RISK"
    system_prompt = """You are a risk management specialist. Focus ONLY on:
regulatory risk, liquidity risk, ESG/sustainability risk, concentration risk,
geopolitical risk, and operational risk.
Do NOT opine on valuation — that is BULL/BEAR's job.
Structure: [Top Risk] [Regulatory] [ESG] [Liquidity] [Overall Risk Rating: 1-10]
Be concise — 150-200 words maximum."""
```

### src/agents/synthesis_agent.py
```python
import anthropic
from ..models import ResearchInput, AgentAnalysis, ResearchNote

class SynthesisAgent:
    def __init__(self, client: anthropic.AsyncAnthropic):
        self.client = client

    async def synthesise(
        self,
        input: ResearchInput,
        bull: AgentAnalysis,
        bear: AgentAnalysis,
        risk: AgentAnalysis,
    ) -> ResearchNote:
        print("[SYNTHESIS] Compiling research note...")

        prompt = f"""You are a senior investment analyst compiling a balanced research note.

BULL CASE ({bull.confidence} confidence):
{bull.analysis}

BEAR CASE ({bear.confidence} confidence):
{bear.analysis}

RISK ASSESSMENT:
{risk.analysis}

Produce a balanced one-page research note with:
1. A 2-sentence executive summary
2. Overall recommendation: BUY / HOLD / SELL / UNDER REVIEW
3. Conviction: high / medium / low
4. Brief rationale (3-4 sentences)

Respond with JSON only:
{{"recommendation": "...", "conviction": "...", "executive_summary": "...", "rationale": "..."}}"""

        resp = await self.client.messages.create(
            model="claude-sonnet-4-5",
            max_tokens=400,
            messages=[{"role": "user", "content": prompt}],
        )

        import json
        data = json.loads(resp.content[0].text.strip())
        from datetime import datetime

        return ResearchNote(
            company=input.company,
            generated_at=datetime.utcnow(),
            bull_case=bull,
            bear_case=bear,
            risk_assessment=risk,
            recommendation=data.get("recommendation", "UNDER REVIEW"),
            conviction=data.get("conviction", "low"),
        )
```

### src/council.py
```python
import asyncio
import anthropic
from .agents.bull_agent import BullAgent
from .agents.bear_agent import BearAgent
from .agents.risk_agent import RiskAgent
from .agents.synthesis_agent import SynthesisAgent
from .formatter import ResearchNoteFormatter
from .models import ResearchInput, ResearchNote, AgentAnalysis
from datetime import datetime

UNAVAILABLE = AgentAnalysis(
    agent_name="UNAVAILABLE",
    analysis="Agent failed to respond.",
    key_points=["Unavailable"],
    confidence="low",
)

class InvestmentCouncil:
    def __init__(self):
        self.client   = anthropic.AsyncAnthropic()
        self.bull      = BullAgent(self.client)
        self.bear      = BearAgent(self.client)
        self.risk      = RiskAgent(self.client)
        self.synth     = SynthesisAgent(self.client)
        self.formatter = ResearchNoteFormatter()

    async def analyse(self, input: ResearchInput) -> ResearchNote:
        print(f"\n[COUNCIL] Launching parallel analysis: {input.company}")
        print("[COUNCIL] BULL + BEAR + RISK running simultaneously...\n")

        bull_r, bear_r, risk_r = await asyncio.gather(
            self.bull.analyse(input),
            self.bear.analyse(input),
            self.risk.analyse(input),
            return_exceptions=True,
        )

        bull = bull_r if not isinstance(bull_r, Exception) else UNAVAILABLE._replace(agent_name="BULL")
        bear = bear_r if not isinstance(bear_r, Exception) else UNAVAILABLE._replace(agent_name="BEAR")
        risk = risk_r if not isinstance(risk_r, Exception) else UNAVAILABLE._replace(agent_name="RISK")

        print("\n[COUNCIL] All agents complete. Synthesis compiling...\n")
        note = await self.synth.synthesise(input, bull, bear, risk)
        note.full_markdown = self.formatter.format(note)
        return note
```

### src/formatter.py
```python
from .models import ResearchNote

class ResearchNoteFormatter:
    def format(self, note: ResearchNote) -> str:
        conviction_emoji = {"high": "🔴", "medium": "🟡", "low": "⚪"}.get(note.conviction, "⚪")
        rec_emoji = {"BUY": "📈", "SELL": "📉", "HOLD": "➡️"}.get(note.recommendation, "⏸️")

        bull_points = "\n".join(f"- {p}" for p in note.bull_case.key_points)
        bear_points = "\n".join(f"- {p}" for p in note.bear_case.key_points)
        risk_points = "\n".join(f"- {p}" for p in note.risk_assessment.key_points)

        return f"""# Investment Research Note — {note.company}
**Generated:** {note.generated_at.strftime('%d %b %Y %H:%M UTC')}
**Recommendation:** {rec_emoji} {note.recommendation} | **Conviction:** {conviction_emoji} {note.conviction.upper()}

---

## Executive Summary
{getattr(note, '_executive_summary', 'See full analysis below.')}

---

## 📈 Bull Case ({note.bull_case.confidence.upper()} confidence)
{note.bull_case.analysis}

**Key Catalysts:**
{bull_points}

---

## 📉 Bear Case ({note.bear_case.confidence.upper()} confidence)
{note.bear_case.analysis}

**Key Risks:**
{bear_points}

---

## ⚠️ Risk Assessment
{note.risk_assessment.analysis}

**Risk Factors:**
{risk_points}

---

## Recommendation
**{note.recommendation} | Conviction: {note.conviction.upper()}**

> ⚠️ *This note is AI-generated using Claude AI (Anthropic) and requires human review before use in investment decisions. Not investment advice.*
> *Generated by VAF Investment Council | github.com/vm799*
"""
```

---

## COLOSSUS QA CHECKLIST
- [ ] `asyncio.gather(return_exceptions=True)` — one agent failing doesn't crash council
- [ ] Each agent has 30s timeout (add in production)
- [ ] Synthesis always runs even with UNAVAILABLE agents
- [ ] Output file written atomically to reports/research/
- [ ] Every note has AI disclaimer — cannot be removed
- [ ] JSON parsing in base_agent has try/except — malformed JSON doesn't crash
- [ ] Reports directory auto-created

## QUICK START
```bash
git clone https://github.com/vm799/vaf-am-build-06
cd vaf-am-build-06
cp .env.example .env
uv sync
uv run python run.py --company GSK --context "Q4 2025 earnings beat, FDA approval"
```
