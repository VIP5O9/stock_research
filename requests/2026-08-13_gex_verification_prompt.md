# KIMI K3 — CODE REVIEW / VERIFICATION REQUEST (GEX pipeline)

**Requested:** 2026-08-13 · **Deliver to:** `reports/YYYY-MM-DD_gex_verification.md`

## TASK
Independent code review + mathematical verification of a newly built
`scripts/gex_pipeline.py` in the tradingview-mcp repo. The script computes
Gamma Exposure (GEX) and Vega Exposure (VEX) per strike from FREE Yahoo Finance
options chain data (open interest + implied volatility), with local
Black-Scholes greeks.

## CONTEXT (what the script is supposed to do)
- Fetches nearest-expiration option chain per ticker from Yahoo v7 finance
  options endpoint (crumb-auth handshake, throttled to survive rate limits)
- Computes BS gamma and vega per strike locally: T = max((exp-now)/86400/365, 0.01), r=0.04
  - gamma = phi(d1)/(S*sigma*sqrt(T)), d1=(ln(S/K)+(r+0.5σ²)T)/(σ√T)
  - vega = S*phi(d1)*sqrt(T)/100
- GEX per strike = gamma * openInterest * 100 * spot (calls +, puts −)
- VEX per strike = vega * openInterest * 100 (calls +, puts −)
- Writes data/gex_state.json with per-symbol spot, net_gex, net_vex, top-5 magnet levels
- Read-only. No broker/order code. Deterministic, never fabricates data.

## YOUR DELIVERABLES (verify, don't rubber-stamp)
1. **Math check**: is the BS gamma formula correct? Is the vega normalization
   (÷100) standard/consistent with the GEX dollar convention (×100 contracts × spot)?
   Would GEX units (dollars) be consistent between calls and puts?
2. **Data provenance**: does Yahoo v7 options actually return openInterest and
   impliedVolatility per strike? Any known gotchas (expired chains, zero OI,
   IV=1e-05 placeholders, index vs equity handling)?
3. **Rate-limit design**: is the crumb/session/backoff approach robust? What
   breaks it and what's the recommended hardening?
4. **Edge cases**: negative/zero T, IV<=0, spot<=0, missing OI, strike 0,
   no options for the nearest expiry, partial chains — does the script handle them?
5. **Determinism/honesty**: any path where it could emit a fabricated level?
6. **Verdict**: PASS / PASS-WITH-NOTES / FAIL with exact line-level reasons.
7. **Improvement list**: top 3 fixes ranked by correctness impact.

## FORMAT
Markdown report, start with a 3-line verdict. Table of findings:
Severity | Location | Finding | Fix. Cite exact line numbers where possible.
