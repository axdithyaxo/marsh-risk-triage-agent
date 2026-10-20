# Insurance Risk Triage Agent

Built for Marsh's **Software Engineer, Generative AI and Innovation** role
(req `R_355019`, Dublin Innovation team) — a working demonstration, not a
CV claim, of what that posting specifically asks for: building AI-powered
applications end to end, evaluation-harness and guardrail discipline,
structured outputs, and agent patterns — built in Marsh's own domain
(risk and insurance) instead of a generic example.

## What it does

Three real, runnable scenarios, each a synthetic (invented, no real
client/claimant data) property-risk submission:

1. **Routine submission** — risk-assessed, model recommends BIND, still
   routed through human pre-execution oversight before anything is final.
2. **Elevated-risk submission** — model recommends REFER (flood-plain
   warehouse, high value), same human-gated path.
3. **Adversarial submission** — a prompt-injection attempt
   ("ignore all previous instructions...") is detected and blocked
   *before* it ever reaches a decision step.

Every run produces an HMAC-SHA256-signed, step-by-step audit log
(`lar_logs/run_*.json`) — tamper-evident, not just a plain log file — plus
a generated "Instructions for Use" document (system purpose, known
limitations, human-oversight requirements, prohibited uses).

## Built on Lár, not reinvented

This reuses [Lár](https://github.com/snath-ai/lar) (`lar-engine` on PyPI)
directly — the same framework already on my CV — composed into a new
domain rather than built from scratch. Every primitive below is real,
existing Lár code:

| Primitive | What it does here |
|---|---|
| `PIIRedactionEngine` | Redacts claimant PII (PPSN, phone) before anything else touches the submission |
| `PromptInjectionGuard` | Blocks adversarial content in untrusted external submissions |
| `ActionPolicy` / `PolicyRegistry` / `RiskScorerNode` | Registers risk tier, reversibility, and required oversight level for this action class, and dynamically routes to human review |
| `HumanJuryNode` | The actual human-in-the-loop escalation path |
| `DeployerTransparencyNode` | Generates real instructions-for-use documentation |
| `AuthorityLedger` | Records who/what authorised each step |
| `GraphExecutor(hmac_secret=...)` | Signs the full execution log with HMAC-SHA256 |

The closest existing precedent in Lár's own test suite is
`examples/compliance/28_eu_ai_act_loan_underwriter.py` — this borrows its
structure directly, adapted from loan underwriting to property risk
triage.

## Run it

```bash
pip install lar-engine
python risk_triage_agent.py
```

(If running against a local Lár checkout instead of the published
package, the script falls back to a local `sys.path` insert — see the top
of `risk_triage_agent.py`.)

## What's honest about this

- All three scenarios were actually run, not just written — the output
  above is real, not asserted.
- No real client, claimant, or policy data anywhere in this repo.
- The "routine" and "elevated-risk" scenarios both route through human
  oversight by design, not by accident — the registered policy marks
  bind/refer decisions as `reversibility=False`, so `RiskScorerNode`
  correctly requires human sign-off on every instance of this action
  class, regardless of how routine the submission looks. That's the
  responsible-by-design behaviour for a hard-to-reverse decision, not a
  bug this demo is hiding.
