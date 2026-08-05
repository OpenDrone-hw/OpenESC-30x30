# 10µF 50V 1206 MLCC — Cost & Performance Analysis

## Context
- 60× 1206 caps on the 30x30 board (MOSFET bulk decoupling on +BATT rail)
- Operating voltage: 11.1–25.2V (3S–6S LiPo)
- Current part: GCM31CD71H106KE36L (Murata X7T, C5119295)
- All caps derate heavily at 25V DC bias — dielectric type matters less than expected

## DC Bias Reality (25V bias on 50V-rated 1206 10µF)

Samsung published data from their product tool (measured at 1kHz, 25°C):
- **X5R 10µF 50V 1206**: -87% → ~1.3µF effective
- **X7R 10µF 50V 1206**: -78% → ~2.2µF effective

Murata X7T performs similarly: expect ~2–3µF effective at 25V.
Bottom line: **all 10µF 50V 1206 MLCCs give only ~2–3µF at operating voltage.**

## Candidate Parts — 10µF 50V 1206

| LCSC | MPN | Mfg | Dielectric | Stock | Basic | Qty 5 | Qty 60 | Qty 500 | 60× cost |
|------|-----|-----|-----------|-------|-------|-------|--------|---------|----------|
| C5119295 | GCM31CD71H106KE36L | Murata | X7T | 2,319 | No | $0.342 | $0.342 | $0.279 | **$20.52** |
| C89632 | CL31B106KBHNNNE | Samsung | X7R | 1,011,829 | No | $0.051 | $0.051 | $0.041 | **$3.07** |
| C303950 | 1206B106K500NT | FH | X7R | 388,847 | No | $0.068 | $0.068 | $0.055 | **$4.07** |
| C5449000 | TCC1206X7R106K500HT | CCTC | X7R | 44,779 | No | $0.038 | $0.038 | $0.031 | **$2.30** |
| **C13585** | **CL31A106KBHNNNE** | **Samsung** | **X5R** | **3,941,714** | **Yes** | **$0.028** | **$0.028** | — | **$1.67** |

## Current 0805 Part for Reference

| LCSC | MPN | Mfg | Dielectric | Stock | Basic | 60× cost |
|------|-----|-----|-----------|-------|-------|----------|
| C440198 | GRM21BR61H106KE43L | Murata | X5R | 5,318,712 | Yes | $3.49 |

Note: 0805 has worse DC bias than 1206 (smaller dielectric volume).

## Recommendation

**C13585 (CL31A106KBHNNNE)** — Samsung X5R 10µF 50V 1206

- **$1.67 for 60 pcs** vs $20.52 for Murata X7T → **12× cheaper**
- 3.9M stock — no supply risk
- **JLCPCB basic part** — no $3 extended setup fee per unique part
- DC bias performance: ~1.3µF at 25V — slightly worse than X7R's ~2.2µF
- For bulk MOSFET decoupling, absolute capacitance matters less than ESR/ESL

If the extra ~1µF effective capacitance from X7R matters:
**C89632 (CL31B106KBHNNNE)** — Samsung X7R, $3.07 for 60, 1M stock, extended part

## Total BOM Impact (60 caps)

| Option | Part cost | Setup fee | Total | vs Murata X7T |
|--------|-----------|-----------|-------|---------------|
| GCM31CD71H106KE36L (current) | $20.52 | $3.00 | $23.52 | baseline |
| CL31A106KBHNNNE (X5R basic) | $1.67 | $0.00 | **$1.67** | **saves $21.85** |
| CL31B106KBHNNNE (X7R ext) | $3.07 | $3.00 | $6.07 | saves $17.45 |
| TCC1206X7R106K500HT (X7R ext) | $2.30 | $3.00 | $5.30 | saves $18.22 |

## SMBJ26A TVS Diode Alternative

Current: C908802 (GOODWORK) — 208 pcs at JLCPCB assembly, 190 shortfall, extended, €0.65

| LCSC | MPN | Mfg | Stock | Price | Notes |
|------|-----|-----|-------|-------|-------|
| **C123820** | SMBJ26A | MDD | 22,554 | $0.051 | Best stock, reliable mfg |
| C353371 | SMBJ26A | Shandong Jingdao | 12,930 | $0.034 | Cheapest |
| C315992 | SMBJ26A | Littelfuse | 4,296 | $0.086 | Brand name |

All are identical spec: 26V standoff, 28.9V breakdown, 42.1V clamp, 600W peak, SMB package.
Recommend **C123820** (MDD) — 22k+ stock, $0.051, proven JLCPCB supplier.

---
*Generated 2026-03-15*
