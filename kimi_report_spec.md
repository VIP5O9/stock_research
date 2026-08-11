# Kimi Daily Stock Report — Format Spec v1.0

**Purpose:** Machine-consumable daily report. The Hermes trading coordinator ingests this
JSON each morning (M-F, 8:45 ET) to feed the paper pipeline, level monitor, and DCA scans.
**Rule:** Every field below is REQUIRED unless marked optional. No prose-only sections.

---

## 1. JSON Schema (the file Kimi must emit)

Filename: `YYYY-MM-DD_kimi.json` (dynamic date, e.g. `2026-08-12_kimi.json`)

```json
{
  "schema_version": "1.0",
  "date": "2026-08-12",
  "generated_at_et": "08:45",
  "as_of_et": "11:00",
  "market_session": "regular" | "premarket" | "closed",
  "overnight": {
    "summary": "one paragraph: geopolitics, oil, futures, Asia/Europe tape",
    "implication": "what it means for US tech / size (one line)"
  },
  "macro_calendar": [
    {
      "event": "CPI",
      "datetime_et": "2026-08-12 08:30",
      "consensus": "+0.1% m/m",
      "implication": "hot -> multiple compression, cut size; cool -> risk-on"
    }
  ],
  "sector_rotation": [
    {
      "sector": "semis",
      "day_change_pct": -0.4,
      "week_change_pct": 1.2,
      "read": "lagging" | "leading" | "neutral",
      "note": "SMH below SMA50; IGV extended"
    }
  ],
  "book_conflicts": [
    {
      "ticker": "KTOS",
      "report_read": "bullish",
      "our_book": "exit_pending",
      "conflict": true,
      "note": "external bullish call does not reverse broker-confirmed exit"
    }
  ],
  "tickers": [
    {
      "ticker": "NVDA",
      "price": 219.88,
      "chg_pct": 1.1,
      "gap_pct": 2.1,
      "rvol": 1.4,
      "atr_pct": 3.41,
      "sma20": 207.9,
      "sma50": 206.3,
      "sma200": null,
      "rsi14": 60,
      "iv30": 39.5,
      "ivr": 65,
      "support": 190,
      "resistance": 225,
      "buy_zone": [210, 217],
      "invalidation": 205,
      "target": [245, 260],
      "catalyst": "earnings 2026-08-26 (cons +81% rev)",
      "flag": "🟢",
      "flag_reason": "$500B AI financing pact; SpaceX Vera Rubin; Malaysia permit risk",
      "as_of_et": "11:00",
      "yesterday_call": "buy 210-217; never hit zone -> still watching",
      "option_play": {
        "is_options_eligible": true,
        "preferred_strike": "215C",
        "expiry": "2026-09-18",
        "notes": "IVR 65 - premium elevated, prefer zone entry"
      }
    }
  ],
  "verification": {
    "prices_live": 18,
    "prices_unverified": 0,
    "sources_cited": 18
  }
}
```

---

## 2. Markdown Template (human version — same data, rendered)

```markdown
# Daily Stock & Catalyst Report — {YYYY-MM-DD} ({Day}, live ~{HH:00} ET)

**Overnight:** {summary}
**Implication:** {implication}

## Macro Calendar
| Event | Date/Time ET | Consensus | Implication |
|---|---|---|---|
| CPI | 2026-08-12 08:30 | +0.1% m/m | hot -> cut size |

## Sector Rotation
| Sector | 1d % | 5d % | Read | Note |
|---|---|---|---|---|
| semis | -0.4 | +1.2 | lagging | SMH < SMA50 |

## Book Conflicts
| Ticker | Report Read | Our Book | Conflict |
|---|---|---|---|
| KTOS | bullish | exit_pending | ⚠️ do not reverse exit |

## Table (one row per ticker — SAME schema order every day)
| Ticker | Price | Chg% | Gap% | RVOL | ATR% | SMA20 | SMA50 | RSI | IV30 | IVR | Sup | Res | Buy Zone | Invalid | Target | Catalyst | Flag |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| NVDA | 219.88 | +1.1 | +2.1 | 1.4 | 3.41 | 207.9 | 206.3 | 60 | 39.5 | 65 | 190 | 225 | 210-217 | <205 | 245-260 | Earn 8/26 | 🟢 |

## Deep Dives (one per ticker, strict format)
**NVDA** ✅ — {1-line price action} | {1-line catalyst} | {1-line risk}
Buy {zone}; invalid {level}; target {range}. {yesterday_call}

## Yesterday vs Today (scorecard)
| Ticker | Yesterday's Call | What Happened | Verdict |
|---|---|---|---|
| NVDA | buy 210-217 | traded 219-221, zone never hit | still watching |

*Verified: {n} prices live, {n} unverified. Sources: [links]*
```

---

## 3. Hard Rules for Kimi

1. **Every ticker gets the same schema** — buy_zone / invalidation / target. No "stagger,"
   no "watch," no "reclaim above X only." One machine schema, every name.
2. **Include ALL book names:** NVDA, META, AVGO, TSLA, AMZN, AAPL, MSFT (mandate) +
   PLTR, KTOS, RGTI, HOOD (managed/exit) + QQQ, SMH, AIS, EUV, DRAM (vault) +
   discovery candidates (V, TXN, AMAT, LLY, LRCX when in signal stack).
3. **IV30 + IVR are mandatory for options eligibility.** If a ticker is options-eligible,
   give preferred strike/expiry. No options section = no options plays logged.
4. **Every price needs as_of_et.** Stale rows are flagged, never silently included.
5. **Overnight must include an implication line** — I should not have to infer it.
6. **Yesterday's call is mandatory** — the scorecard makes the report accountable.
7. **Book conflicts section is mandatory** — when the report read and our book disagree,
   say so explicitly. External bullish calls never reverse broker-confirmed exits.
8. **rvol (relative volume) and atr_pct are mandatory** — levels without volume are not
   triggers; stops without ATR are invented.
9. **Sources cited with footnotes per claim** (keep current style — it verifies well).
10. **Emit BOTH files:** `YYYY-MM-DD_kimi.json` (machine) + `YYYY-MM-DD_kimi.md` (human).
    Commit both to `VIP5O9/stock_research` main.

---

*Spec v1.0 — Hermes trading coordinator, 2026-08-11. Feedback loop: gaps found today →
MSFT paper entry was above the report's buy zone because the zone wasn't machine-readable;
KTOS conflict wasn't flagged because the report doesn't know the book. Both fixed by v1.0.*
