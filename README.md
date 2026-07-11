# Standalone SPY/SPX Trading Knowledge Manual

This is a self-contained trading knowledge manual. It is designed to be read, shared, or used on its own. All pertinent setup definitions, indicator rules, probability tables, operating caveats, and live-analysis templates are included directly here.

This is research support and trade-planning education, not financial advice. Historical SPY statistics and extracted trading doctrine are not guarantees. SPX options execution, spreads, settlement, implied volatility, greeks, slippage, and trader discipline can dominate the underlying move edge.

Scope: individual trade reviews, one-off recaps, private notes, personal references, and external dependencies are intentionally excluded.

---

## 0. Live-trading operating rules

When analyzing SPX/SPY, use this sequence:

1. **State the product and timeframe**: SPX/SPY/ES, RTH vs premarket, 1m/3m/10m/1h/4h/daily context.
2. **Separate facts from thesis**:
   - Facts: price relative to PDC, ATR grid, VWAP, ribbon/EMA stack, PO state, compression, time of day.
   - Thesis: likely path, active setup, directional bias.
3. **Use if/then plans, not predictions**:
   - If price reclaims/holds X, target Y.
   - If price loses/closes through X, thesis invalidates.
4. **Always name**:
   - Setup
   - Preconditions
   - Entry trigger
   - Invalidation
   - Target ladder
   - Confidence / caveats
5. **Never treat an edge as certainty**. A 70–90% historical stat still fails.
6. **Prefer level-to-level thinking**. The system is built around hops through ATR levels, not one-shot full-ATR predictions.
7. **Respect time decay**. Early RTH triggers are much higher quality than late-day triggers, especially bullish Golden Gates.
8. **Mark stats vs doctrine**:

---

## 1. Vocabulary and notation

### Products

- **SPX**: S&P 500 cash index; cash-settled options; often used for 0DTE/1DTE premium trades.
- **SPY**: S&P 500 ETF; underlying used for the 25-year backtested statistics.
- **ES / /ES**: S&P futures; useful for overnight/premarket context.
- **QQQ / NDX / /NQ**: Nasdaq variants.
- **IWM / RUT**: Russell variants.
- **VIX / VVIX / VX**: volatility context.

### Session/time terms

- **Premarket**: 4:00–9:30 ET.
- **RTH**: 9:30–16:00 ET.
- **Opening range**: first 5m or 10m of RTH.
- **Power hour**: 15:00–16:00 ET.
- **OpEx / weekly / monthly / quad witching**: option-expiration context.
- **0DTE / 1DTE**: days to expiration.

### ATR level notation

- **PDC / central pivot / previous close**: zero-line for daily ATR levels.
- **Trigger**: ±23.6% ATR from previous close. Upper = call trigger. Lower = put trigger.
- **Golden Gate entry / GG open**: ±38.2% ATR. The Golden Gate **opens** here.
- **Midrange**: ±50% ATR.
- **Golden Gate completion / Golden Fib**: ±61.8% ATR. The Golden Gate **completes** here.
- **Full ATR**: ±100% ATR.
- **Extensions**: ±123.6%, ±161.8%, ±200%, ±261.8%, ±300%.

Critical terminology: **hitting the trigger does not mean the Golden Gate opened**. Trigger is 23.6%; GG opens at 38.2%; GG completes at 61.8%.

### Options/trade lifecycle

- **BTO / STC**: buy to open / sell to close.
- **STO / BTC**: sell to open / buy to close.
- **PCS / CCS**: put credit spread / call credit spread.
- **Partials / scale out**: take profit in pieces.
- **Runners**: final portion left for extension.
- **ITM**: in the money.

### Chart shorthand

- **HOD / LOD**: high/low of day.
- **HOW / LOW**: high/low of week.
- **Reclaim**: regain a level/EMA.
- **Lose/lost**: break below a level/EMA.
- **Hold**: test and respect a level.
- **Break and hold**: breakout plus confirmation.
- **Stacked bullish**: shorter EMAs above longer EMAs.
- **Stacked bearish**: shorter EMAs below longer EMAs.
- **Curling / flattening**: EMA slope changing direction or losing slope.

---

## 2. Core Saty indicators

### 2.1 Saty ATR Levels

**Standalone construction:**

- Anchor: previous period close.
- Volatility unit: 14-period ATR using Wilder/RMA smoothing.
- Day mode: previous regular-session close plus daily ATR.
- Multiday mode: previous weekly close plus weekly ATR.
- Swing mode: previous monthly close plus monthly ATR.
- Position mode: previous quarterly close plus quarterly ATR.
- Long-term mode: previous yearly close plus yearly ATR.
- The same Fibonacci ladder is mirrored above and below the anchor.
- All levels are potential support/resistance, targets, or invalidation zones.

**Formula concept:**

```text
upper_level = previous_close + ATR14 * level_multiplier
lower_level = previous_close - ATR14 * level_multiplier
```

Where level multipliers include 0.236, 0.382, 0.500, 0.618, 0.786, 1.000, 1.236, 1.618, 2.000, 2.618, and 3.000.

**Definition:** ATR-based Fibonacci grid using the previous period close and the 14-period ATR from a higher timeframe. Day mode uses daily ATR, multiday uses weekly, swing uses monthly, position uses quarterly, long-term uses yearly.

**Key levels:**

| Level | Distance from previous close | Live-use meaning |
|---|---:|---|
| Trigger | ±23.6% ATR | First long/short trigger zone |
| GG entry / GG open | ±38.2% ATR | Golden Gate opens |
| Midrange | ±50% ATR | Intermediate target; watch R:R |
| GG completion / Golden Fib | ±61.8% ATR | Main Golden Gate target |
| Full ATR | ±100% ATR | Large daily move; cumulative probability low |
| Extensions | ±123.6%, ±161.8%, ±200% | Momentum extension zones |

**Implementation notes:**

- ATR uses Wilder/RMA smoothing, not SMA: `ewm(alpha=1/period, adjust=False)`.
- Intraday ATR tables use daily ATR reference, matching TradingView `request.security(ticker, 'D', ta.atr(14))` behavior.
- Daily/weekly candles use RTH data only.
- Bad tick clipping caps bar wicks at 2% beyond the candle body to remove phantom prints while preserving real volatility.

**Operating rule:** use the ATR grid as support/resistance and target ladder. Do not jump straight from PDC to full ATR unless the setup supports it.

### 2.2 Saty Pivot Ribbon Pro

**Definition:** multi-layer EMA cloud/ribbon system showing trend structure, pullback zones, compression candles, and conviction signals.

**EMA layers:**

| EMA | Role |
|---:|---|
| 8 | Fast EMA; short-term trend/pullback reference |
| 13 | Pullback overlap EMA; used in slow cloud variant and Vomy/iVomy logic |
| 21 | Pivot EMA; core trend reference |
| 34 | Often used as stop/reference in Saty trade examples |
| 48 | Slow EMA; major trend/cloud reference |
| 200 | Long-term trend anchor |

**Cloud/ribbon reading:**

- Fast cloud: EMA 8 vs EMA 21; green when 8 ≥ 21, red when 8 < 21.
- Slow cloud: EMA 13/48 or 21/48; blue/aqua bullish, orange bearish.
- Ribbon flip: fast cloud changes color; potential trend change.
- Conviction arrows: 13/48 EMA cross, slower but higher-confidence trend confirmation.
- Candle bias:
  - Green: up candle above EMA48.
  - Red: down candle below EMA48.
  - Blue: down candle above EMA48 = pullback in bullish trend.
  - Orange: up candle below EMA48 = bounce in bearish trend.
  - Gray/violet: compression candles.

**Saty doctrine:** trade with the trend; wait for confirmation; do not chase; prefer pullback-to-ribbon entries; avoid sideways/entangled ribbons.

### 2.3 Saty Phase Oscillator

**Definition:** range-normalized momentum oscillator measuring distance from EMA21, normalized by ATR.

Formula:

```text
raw_signal = ((price - EMA21) / (3 * ATR14)) * 100
oscillator = EMA(raw_signal, 3)
```

**Zones:**

| Zone | Range | Meaning |
|---|---:|---|
| Extended Up | > +100 | Overbought extreme |
| Distribution | +61.8 to +100 | Potential topping/profit-taking zone |
| Neutral Up | +23.6 to +61.8 | Healthy uptrend territory |
| Neutral | -23.6 to +23.6 | No clear momentum |
| Neutral Down | -61.8 to -23.6 | Healthy downtrend territory |
| Accumulation | -100 to -61.8 | Potential bottoming zone |
| Extended Down | < -100 | Oversold extreme |

**Signals:**

- Leaving accumulation: crosses above -61.8; potential long/reversal.
- Leaving distribution: crosses below +61.8; potential short/take-profit.
- Leaving extreme down: crosses above -100; strong reversal signal.
- Leaving extreme up: crosses below +100; strong reversal signal.
- Rising/falling state matters; high+rising vs high+falling and low+falling vs low+rising can change GG odds.

**Caveat:** 60m PO has a known accuracy gap versus TradingView because extended-hours data inflates hourly ATR; 10m PO accuracy is much better. Still, 60m PO was more predictive in the historical Bilbo GG tests; treat this as useful but audit-sensitive.

### 2.4 Compression / Squeeze

Compression is detected when Bollinger Band width is small relative to ATR:

- BB width = `2 * stdev(21)`.
- Compression active when BB width < `2 * ATR(14)`.
- Expansion confirmed when BB width grows and exceeds about `1.854 * ATR` threshold.

Saty-style terminology also maps Squeeze/Squeeze Pro:

- Red dots = squeeze on.
- Green dots = squeeze fired.
- Pro intensity colors: black/narrow, red/normal, orange/wide, green/fired.
- “5–10 dots” is a heuristic for enough compression to matter.

### 2.5 VWAP, RAF, volume, VIX, gamma

Use these as context/confluence, not as standalone rules unless the setup requires them:

- **VWAP**: intraday volume-weighted mean; useful for ORB, mean reversion, and institutional trend context.
- **Ready Aim Fire Pro / RAF**: signal dashboard; conviction arrow is tied to the 13/48 confirmation concept.
- **Volume oscillator**: volume momentum; declining volume + wicks can mark exhaustion.
- **VIX key levels**: volatility regime/context.
- **Gamma levels**: possible pin/resistance/support zones around OpEx and large strikes.

### 2.6 Unusual Whales Periscope net gamma exposure

Periscope net gamma exposure is a strike-by-strike map of market-maker gamma positioning. Use it to judge expected dealer hedging behavior around SPX strikes, not as a standalone bullish/bearish signal.

**What the bars mean**

- **Green / positive gamma bar**: market makers are net long options at that strike. Customers are net sellers of options there. The exact customer call/put mix is less important than the dealer gamma sign.
- **Red / negative gamma bar**: market makers are net short options at that strike. Customers are net buyers of options there.

**Mechanical implication**

- **Positive net gamma is suppressive**. Dealers tend to fade movement as price approaches or moves around that strike: sell rips, buy dips. The result is pinning, dampening, magnet behavior, and lower realized volatility near the strike.
- **Negative net gamma is accelerative**. Dealers tend to hedge with the move: buy strength and sell weakness. The result is trend extension, faster air-pocket movement, and higher realized volatility if price enters or breaks through that zone.

**Important nuance**

Positive gamma does not mean bullish. Negative gamma does not mean bearish. Gamma sign describes how dealer hedging is likely to respond after price moves. Direction still comes from price action, level interaction, trend/ribbon state, PO state, catalysts, and trade setup confirmation.

**How to read the ladder**

1. Mark spot price on the strike ladder.
2. Identify the nearest large positive bars above and below spot. These are gamma walls, pin zones, or dampening zones.
3. Identify the nearest large negative bars above and below spot. These are acceleration zones or air pockets.
4. Mark sign flips between adjacent strikes. A flip from positive to negative can create a regime boundary: below it, price may chop/pin; above it, price may accelerate, or vice versa.
5. Compare the profile above spot versus below spot:
   - Positive gamma overhead: upside may be capped or slow unless price accepts above the wall.
   - Positive gamma below: downside may be supported or magnetized back upward unless price rejects through it.
   - Negative gamma overhead: upside breakout can run quickly if price accepts into it.
   - Negative gamma below: downside flush can extend quickly if price loses support into it.
6. Tie the gamma zones to the normal trading map: PDC, daily trigger, GG open, GG completion, full ATR, VWAP, ORB high/low, premarket high/low, pivot ribbon, and known support/resistance.

**Interpretation patterns**

- **Spot inside a strong positive gamma cluster**: expect chop, pinning, failed breakouts, smaller range, and better odds for mean-reversion tactics unless a clean acceptance break occurs.
- **Spot between a positive wall above and negative pocket below**: upside is dampened, but downside can accelerate if support breaks. Bear trades improve only after the break; shorts into support before the break are lower quality.
- **Spot between negative overhead and positive below**: upside can squeeze if resistance is reclaimed; downside may be cushioned near the positive wall.
- **Large positive bar exactly at a round strike near spot**: that strike can become the day’s magnet, especially near expiration.
- **Stacked negative bars with little positive support**: do not assume nearby supports will hold mechanically; flushes can overshoot.
- **Mixed small bars with no dominant wall**: gamma is secondary; prioritize ATR grid, ORB, VWAP, ribbon, and PO.

**Trading use**

- Use positive gamma walls as potential targets, chop/pin areas, take-profit zones, or places to demand stronger confirmation before chasing continuation.
- Use negative gamma pockets as areas where a confirmed break can travel faster than normal; do not fade them blindly.
- For long calls, prefer a clean reclaim and hold above the nearest positive wall before expecting continuation through it.
- For puts, prefer a clean loss and hold below the nearest positive floor before expecting acceleration lower.
- If price is trapped between two positive walls, expect compression and smaller targets.
- If price breaks from a positive zone into a negative zone, expect volatility expansion and faster level-to-level movement.

**Live-read template**

When using a Periscope gamma screenshot, answer in this order:

1. Current spot relative to the ladder.
2. Biggest positive walls by strike.
3. Biggest negative air pockets by strike.
4. Local regime: suppressive, accelerative, mixed, or flip-zone.
5. Bull trigger, target, invalidation.
6. Bear trigger, target, invalidation.
7. Any overlap with ATR trigger/GG levels, ORB, VWAP, PDC, or pivot/ribbon structure.

**Caveats**

- Expiry matters. Same-day gamma can shift quickly and is most relevant for intraday flow. Longer-dated exposure is smoother and less reactive.
- The profile changes intraday as customers trade options. A morning gamma read can be stale later in the session.
- Walls are probabilistic, not absolute. A strong catalyst, trend day, failed auction, or volatility shock can blow through any wall.
- Magnitude is useful mainly for ranking strikes within the same profile; do not overcompare different expiries or different products.
- Gamma gives expected hedging pressure after movement. It does not replace setup confirmation.

---

## 3. Daily market preparation framework

Before evaluating a trade, build the morning/context map:

1. **Higher timeframe context**
   - Daily trend: price vs daily 21 EMA, daily PO zone/rising/falling.
   - 4h PO: extended/distribution/accumulation/rollover.
   - 1h PO: especially for Bilbo GG and call→put reversal filters.
   - 10m ribbon: trend/sideways/compression.
2. **Levels**
   - PDC / previous close.
   - Daily call/put triggers (±23.6%).
   - Daily GG open levels (±38.2%).
   - Daily 50%, 61.8%, 78.6%, full ATR.
   - Premarket high/low.
   - Opening range high/low after 5m/10m.
   - VWAP.
   - Support/resistance and supply/demand zones.
3. **Regime**
   - Trend day: ribbon stacked, pullbacks hold.
   - Compression: squeeze/box forming; wait for expansion direction.
   - Chop: price around PDC/VWAP, entangled ribbon; avoid or use credit/income logic.
   - Event/expiration: OpEx, CPI/Fed, quad witching, gamma/pin context.
4. **Primary setup map**
   - Do not chase random candles. Identify which setup is active.
   - If no primary setup is active, do nothing.

Saty doctrine: identify market condition first, then apply the matching primary setup; “FOMO is stupid.” Doing nothing is often the best course.

---

## 4. Primary setup map

### Trend continuation

- Golden Gate continuation.
- Bilbo Golden Gate.
- ORB continuation.
- Vomy/iVomy ribbon continuation.
- Pivot/ribbon pullback.

### Breakout / expansion

- Trigger break and hold.
- 10m compression → expansion.
- Bilbo Box break (1h and up only).
- Opening range break.
- Resistance/support break and retest.

### Mean reversion

- ±1 ATR RTM.
- Gap fill / midpoint fill.
- Price vs daily 21 EMA extreme reversion.
- 4h PO leaving extreme/distribution/accumulation.

### Reversal/exhaustion

- Call trigger → put trigger morning reversal.
- 4h PO rollover + OpEx window.
- Bull/bear divergences.
- Failed breakout.
- H pattern / Head & Shoulders / wedges / Wyckoff.

### Income / credit spread

- Trigger Box held for 30m/1h and selling opposite-side spreads at ±61.8 or ±100.
- Avoid spread structures when trend/volatility can quickly invalidate the box.

---

## 5. Golden Gate system

### 5.1 Definition and mechanics

The Golden Gate is the path from ±38.2% ATR to ±61.8% ATR:

- Bull GG opens when price reaches +38.2% ATR.
- Bull GG completes when price reaches +61.8% ATR.
- Bear GG opens when price reaches -38.2% ATR.
- Bear GG completes when price reaches -61.8% ATR.

Ideal doctrine: clear trend, EMAs stacked in the direction of the move, and entry at/near the ribbon if possible. Take profit at the 61.8 level / Golden Fib.

### 5.2 Level-to-level probabilities

Day-mode, within the same day:

| Move | Historical probability |
|---|---:|
| Close → ±Trigger | 99.2% in either direction in the current study set; older framing also cites 80% in a different level-to-level definition |
| Trigger → ±38.2% | 78.6% bull / 79.0% bear pooled; **67.0% / 69.8% for the live cohort** (open before trigger, cross intraday) |
| 38.2% → 61.8% | 62.5% bull / 65.4% bear pooled; **52.0% / 56.6% for the live cohort** (open before 38.2, cross intraday) |
| 61.8% → 78.6% | 60% |
| 78.6% → full ATR | 55% |
| Close → full ATR cumulative | ~2% |
| Bull GG baseline completion | 62.5% (2,139/3,421) |
| Bear GG baseline completion | 65.4% (2,090/3,196) |

**Cohort honesty (2026-04-26 rerun):** the pooled numbers mix three populations — days that *gap open* beyond the target (trivially 100%), days that open between the levels, and days that approach the level intraday. A trader watching price approach the level live faces the intraday-cross cohort, not the pooled number. For the GG leg that is 52.0% bull (1,224/2,355) and 56.6% bear (1,325/2,339). Use the live-cohort numbers for trade decisions; the pooled numbers only describe the full historical population.

### 5.3 Bilbo Golden Gate: 1h PO filter

The 60-minute Phase Oscillator filter changes GG completion odds. **These are the corrected numbers**: the original study had a look-ahead bug (10m triggers were classified by the still-unfinished 1h candle's PO — up to 60 minutes of forward leakage). It was fixed on 2026-04-26 (point-in-time-safe join: only fully closed 1h bars visible at trigger time) and fully rerun on the 2000–2025 window.

Bull GG completion by 1h PO state (corrected):

| 1h PO state | Completion |
|---|---:|
| High + Rising | 71.0% (137/193) |
| High + Falling | 73.8% (124/168) |
| Mid + Rising | 60.5% (n=2,195) |
| Mid + Falling | 63.0% (n=817) |
| Baseline | 62.5% |

Bear GG completion by 1h PO state (corrected):

| 1h PO state | Completion |
|---|---:|
| Low + Falling | 85.1% (120/141) |
| Low + Rising | 86.6% (129/149) |
| Mid + Falling | 61.2% (n=2,103) |
| Mid + Rising | 69.6% (n=773) |
| Baseline | 65.4% |

What survived the fix, and what did not:

- **Extreme-zone edge survived, smaller**: bear Low-zone Bilbo still completes ~85–87% vs 65.4% baseline (+20pp); bull High-zone ~71–74% vs 62.5% (+9–11pp). Bear Bilbo remains the standout configuration.
- **The mid-zone slope splits were a look-ahead artifact.** Pre-fix, counter-slope mid buckets looked like "avoid" cells (bull Mid+Falling 51.5%, bear Mid+Rising 54.2%). Post-fix they partly *inverted* (63.0% and 69.6%). Do not use 1h PO slope in the mid zone as a filter in either direction.
- Extreme-zone sample sizes shrank materially under the honest join (bull High+Rising n=372 → 193; bear Low+Falling n=265 → 141): the setup is rarer than previously published.

Continuation beyond 61.8 (to 78.6 / full ATR / 123.6) was published from the same pre-fix pipeline and has **not** been rerun with the corrected join — treat those ladder numbers as unverified and do not size on them.

### 5.4 10m vs 60m PO

The published claim that the 60m PO was 5–12x more predictive than the 10m PO for GG completion came from a page whose mapped study did not reproduce it, and it predates the look-ahead fix. Treat "prefer 60m PO over 10m for Bilbo" as **unverified** pending a corrected rerun; the corrected §5.3 tables above are the only Bilbo numbers with a clean join.

### 5.5 Entry optimization

| Entry method | Result / interpretation |
|---|---|
| Immediate at 38.2 | 63–65% completion, +10% ATR EV; appears on all GG days |
| 10m EMA8 pullback | 62–63%, +10–12% EV; appears 97% |
| 10m EMA21 pullback | 58%, +8% EV; appears 88% |
| 1h EMA21 pullback | 42%, +7–9% EV; appears 57–62%; best R:R 2.3–3.2x |
| 50% midpoint | 60%, negative EV (-3%); reward too small vs risk |
| Trigger pullback | 43–48% completion but 38.2% ATR reward |

Saty doctrine favors entries at ribbon pullbacks when possible; the backtest says immediate and EMA8/EMA21 pullbacks have similar hit rates, while deeper 1h EMA21 pullbacks have better R:R but lower completion.

### 5.6 Invalidation

The trigger (23.6%) is the key stop/kill level once GG is active:

- Trigger holds: 84–89% GG completion.
- 10m close back through trigger: 45–51% completion.
- Delta: -39pp; strongest level-based invalidation signal.

Other warnings:

- 1h EMA21 break: -20 to -28pp delta.
- 10m EMA48 break: -18 to -20pp delta.
- 10m EMA21 break: only -6pp, weaker.
- 10m EMA8 break: noise.

Operational rule: if in a GG trade, a 10m close back through trigger is a major de-risk/cut signal.

### 5.7 Timing / Subway

GG completion by trigger hour:

| Trigger time | Bull GG completion | Bear GG completion |
|---|---:|---:|
| Open | 86% | 88% |
| 09:30 | 63% | 64% |
| 10:00 | 57% | 60% |
| 11:00 | 55% | 56% |
| 12:00 | 48% | 56% |
| 13:00 | 39% | 56% |
| 14:00 | 41% | 48% |
| 15:00 | 16% | 30% |

Key: open triggers are highest conviction; bearish GGs remain more viable later than bullish; late bullish triggers are weak.

---

## 6. Trigger Box system

### 6.1 Definition

A trigger box day opens between PDC and a trigger:

- **Bearish trigger box**: open below PDC but above put trigger. Occurs 22.6% of days (n=1,462).
- **Bullish trigger box**: open above PDC but below call trigger. Occurs 26.3% of days (n=1,698).

Saty doctrine: in a trend, break of call trigger can justify calls; break of put trigger can justify puts; stops can be just below/above the trigger; scale at midrange and ±1 ATR. 2–4 DTE on SPY was noted as a preference in the extracted doctrine.

### 6.2 Hold-time implications

GG open rates / day outcome once the box holds:

| Condition | Bull GG open | Bear GG open |
|---|---:|---:|
| Baseline | 57.3% | 59.0% |
| Box held 30m | 72.6% | 71.6% |
| Box held 1h | 82.2% | 80.2% |

If GG has not triggered after hold:

- After 30m: 59–62% still trigger later.
- After 1h: 55–60%.
- After 2h: ~50% coin flip.
- After 2.5h: below 50% for bullish.
- After 5h / 2:30 PM: effectively over (~25–31%).

When PDC is reclaimed in first hour:

- 73% reach opposite trigger.
- 49% reach opposite GG entry (38.2%).

### 6.3 Trigger Box credit spreads

Win rate here means price does **not** reach the sold-side target level.

Sell call spreads from bearish box:

| Condition | +38.2 | +61.8 | +100 |
|---|---:|---:|---:|
| All days | 66.6% | 84.6% | 96.1% |
| Held 30m | 79.7% | 90.6% | 97.8% |
| Held 1h | 85.8% | 93.6% | 98.7% |

Sell put spreads from bullish box:

| Condition | -38.2 | -61.8 | -100 |
|---|---:|---:|---:|
| All days | 64.8% | 79.9% | 92.6% |
| Held 30m | 76.0% | 87.7% | 96.0% |
| Held 1h | 82.6% | 92.0% | 97.5% |

Two approaches:

- ±100: 97–99% win, lower premium.
- ±61.8: 88–93% win, more premium but worse loss ratio.

Use ±38.2 as stop concept. Setup fires ~10% of trading days.

---

## 7. Call trigger confirmation and call→put reversal

### 7.1 Call trigger confirmation

Setup: open inside trigger box, then first 3-minute close above the call trigger. Target: +38.2% ATR.

Universe:

- 49.4% of days open inside the box.
- 62.3% of those get a confirmed trigger close.

Outcomes:

| Metric | Value |
|---|---:|
| Overall hit rate | 73.8% (1,496 / 2,027) |
| Clean run hit rate, no close back below trigger | 97.1% (747 / 769) |
| Invalidated hit rate, closed back below trigger | 59.5% (749 / 1,258) |
| Edge from invalidation filter | +37.6pp |
| Median time to target | 18 minutes / 6 bars |

By time:

- 09:30 trigger: 81.4% hit.
- 10:00: 76.2%.
- 10:30: 73.0%.
- 11:00: 73.3%.
- 14:00: 74.0%.
- 15:30: 29.1%.

Key: a 3-minute close back below the trigger is a powerful kill signal; clean trades before 14:00 were near-100% in this study.

### 7.2 Call trigger → put trigger morning reversal

Setup: SPY reaches daily call trigger before noon, later crosses below PDC, then reaches daily put trigger before noon. Outcomes measured from put-trigger touch through RTH close.

| Outcome after put trigger | Rate |
|---|---:|
| Back to PDC | 73.7% |
| Back to call trigger | 43.3% |
| Downside GG opens (-38.2%) | 75.3% |
| Downside GG completes (-61.8%) | 43.6% |
| Reaches -1 ATR | 18.5% |

1h PO filter at put-trigger touch:

| 1h state | PDC recovery | Downside GG open | GG complete | -1 ATR | Close below put |
|---|---:|---:|---:|---:|---:|
| Bullish expansion | 77.7% | 69.6% | 32.4% | 14.2% | 34.5% |
| Compression | 67.1% | 75.8% | 46.8% | 19.6% | 44.7% |
| Bearish expansion | 82.8% | 79.3% | 47.1% | 20.1% | 40.2% |

Interpretation:

- This reversal often creates both a PDC recovery and downside GG open; do not force a single clean outcome.
- PDC mean reversion is much more reliable than a full move back to call trigger.
- Hourly PO compression is the most bearish filter: lowest PDC recovery and highest close-below-put rate.
- Bullish hourly expansion suppresses downside continuation.

---

## 8. Gap framework

### 8.1 Concepts

- Gap fill: price returns to prior close.
- Midpoint fill: price reaches halfway back through the gap.
- Small gaps are more likely to fill quickly.
- Counter-trend gaps fill faster than pro-trend gaps.

### 8.2 Midpoint fill rates

Gap up midpoint fill rates day 1:

| Gap size | All | EMA21 bearish | EMA21 bullish |
|---|---:|---:|---:|
| <0.25% | 94% | 99% | 93% |
| 0.25–0.5% | 83% | 96% | 77% |
| 0.5–1% | 73% | 85% | 66% |
| 1–2% | 57% | 67% | 49% |
| 2%+ | 62% | 68% | 57% |

Gap down midpoint fill rates:

| Gap size | All | EMA21 bearish | EMA21 bullish |
|---|---:|---:|---:|
| <0.25% | 95% | 92% | 96% |
| 0.25–0.5% | 88% | 84% | 91% |
| 0.5–1% | 77% | 71% | 86% |
| 1–2% | 70% | 66% | 80% |

Key: tiny gaps (<0.25%) are near-certain midpoint fills; counter-trend gaps fill faster; large gap-ups in compression plus bull trend resist filling.

### 8.3 Same-day full gap-fill table

Full gap fill means price returns all the way to the prior close during the same session. These rates are different from midpoint-fill rates above.

| Gap size | Gap-up full fill | Gap-down full fill |
|---|---:|---:|
| <0.10% | 92.0% | 92.9% |
| 0.10–0.25% | 76.5% | 78.9% |
| 0.25–0.50% | 58.6% | 62.9% |
| 0.50–0.75% | 44.6% | 47.7% |
| 0.75–1.00% | 40.2% | 34.2% |
| 1.00–1.50% | 28.3% | 36.7% |
| 1.50–2.00% | 20.0% | 31.1% |
| 2.00–3.00% | 27.5% | 41.5% |
| 3.00%+ | 43.8% | 15.0% |

Key distinction: midpoint fill is much easier than full fill. A small gap can have a very high midpoint-fill rate while a full fill becomes materially less reliable once the gap exceeds roughly 0.25–0.50%.

### 8.4 Operational use

Use gaps as a contextual magnet, not an automatic trade:

- If gap is small and counter-trend, gap/midpoint fill can be primary target.
- If gap is pro-trend and ribbon/PO support continuation, avoid fading too early.
- Combine gap target with PDC, VWAP, premarket high/low, and ATR levels.
- If price rejects a supply/demand zone and gap remains below/above, gap fill becomes a target; scale at intermediate support/resistance.

---

## 9. Compression / expansion / Bilbo Box

### 9.1 10m compression → expansion

Setup: 10m PO enters compression / Bollinger squeeze for at least 30 minutes. Expansion direction is measured from compression range midpoint over 120 minutes after expansion.

Total events: 6,116; 3,311 bullish and 2,805 bearish.

Baseline:

- 54.1% bullish expansions; 45.9% bearish.
- Expansion direction correct 91% of the time, measured by max profit > max drawdown.
- Bullish mean profit 0.58%, mean net +0.31%.
- Bearish mean profit 0.69%, mean net +0.35%.

EMA 21/48 trend predicts direction:

| Compression duration | 21>48 → Bull% | 21<48 → Bull% | Edge |
|---|---:|---:|---:|
| 30–50m | 63.5% | 42.5% | +21pp |
| 60–110m | 68.2% | 34.8% | +33pp |
| 120–170m | 78.6% | 30.2% | +48pp |
| 180m+ | 83.7% | 22.9% | +61pp |

Compression length increases reliability:

| Duration | N | Mean profit | Net > 0 |
|---|---:|---:|---:|
| 30–50m | 1,795 | 0.607% | 76.1% |
| 60–110m | 2,256 | 0.625% | 81.4% |
| 120–170m | 1,119 | 0.640% | 83.6% |
| 180m+ | 946 | 0.680% | 86.2% |

ATR position at expansion:

- Above +61.8%: 91% bullish.
- Trigger–38.2%: 81% bullish.
- Bull trigger box: 67% bullish.
- Bear trigger box: 38% bullish / 62% bearish.
- Below -61.8%: 89% bearish.

Operational rule: long compression + aligned EMA21/48 + supportive ATR position = strong directional clue. Time of day matters less than trend and ATR position.

### 9.2 Bilbo Box breakout

Definition: Bilbo Box is the range of the first 5 compression bars. Once the box locks, break-watch begins; price trading outside the range is the break. Tested variants:

- Immediate at boundary.
- Close-outside confirmation.
- Retest.

Sample: 50,889 break events across 3m/10m/1h/4h/1d, 2000–2026. **Live use is hourly and up only**: the intraday frames (3m/10m) showed only marginal texture (~51% hit, ~+0.03R median) with no usable standalone edge, and are excluded from the playbook.

10-bar Net-R median (tradable frames):

| TF | N | Immediate | Close | Retest |
|---|---:|---:|---:|---:|
| 1h | 3,399 | +0.04 | 0.00 | -0.02 |
| 4h | 936 | +0.03 | +0.05 | -0.01 |
| 1d | 279 | +0.06 | +0.09 | 0.00 |

Takeaways:

1. Take the break; do not wait for retest. Retest underperforms everywhere.
2. Do not wait for textbook 5-bar formation; short boxes (1–4 bars) carried more edge than full 5-bar boxes.
3. Higher timeframes favor bull side; shorting daily compression breaks was negative expectation due to SPY upward drift.
4. Stops at opposite boundary; realistic TP 0.5–1.0R.
5. Even on 1h+, the raw signal is modest (+0.04 to +0.09R median). Stack with trend/time/volatility filters; do not trade it standalone at size.

Caveats: 1h PO/data issue; 1d sample underpowered; ambiguous outside bars excluded. A follow-up higher-timeframe PO study found same-direction PO expansion does NOT improve Bilbo outcomes (tested on intraday cohorts) — do not treat PO confirmation as a green-light.

---

## 10. Mean reversion systems

### 10.1 ±1 ATR RTM

Saty doctrine: RTM / return-to-mean works around ±1 ATR extremes, Phase Oscillator extremes, ribbon distance, and exhaustion. It is both-directional and should be paired with context, not blindly faded.

Operational pattern:

- Identify extension into ±1 ATR / extended PO / far-from-ribbon condition.
- Look for exhaustion: wicks, declining volume, squeeze fired at extreme, PO curling/leaving extreme.
- Prefer reversal back toward nearer ATR levels, VWAP, EMA/ribbon, or PDC rather than demanding full reversal.
- If trend is strong and ribbon remains clean, a ±1 ATR touch may become continuation, not reversion.
- Manage quickly; lotto/no-stop examples exist in Saty, but for systematic use define invalidation.

### 10.2 Price vs daily 21 EMA reversion

Absolute extremes over 25 years:

- Max above daily EMA21: +7.21%.
- Max below: -18.53%.
- Median: +0.68%; SPY naturally rests slightly above EMA21.
- 83.7% of days close within ±2% of EMA21.

Mean reversion by deviation:

| Deviation | 1d | 5d | 10d | 20d |
|---|---:|---:|---:|---:|
| > +5% | -0.73% | -0.88% | -0.58% | +0.43% |
| < -5% | +0.24% | +0.78% | +0.97% | +3.31% |
| < -7% | +0.50% | +1.75% | +2.64% | +5.36% |

>4% above EMA21 zone:

- 50 episodes.
- 100% touched EMA21 within 28 days; median 8 days.
- Peak day forward returns: 1d -0.83%, 3d -1.02%.
- Practical signal: 4h PO declining while daily PO still rising (n=38): 1d -0.42%, 2d -0.81%, 3d -0.85%.
- Strongest signal: 4h PO big drop delta < -10 (n=7): 1d -1.64%, 2d -1.29%.

### 10.3 4h PO rollover + OpEx window

Signal: 4h PO peak ≥80, then crosses below 80 (leaving distribution). Baseline sample: 118 signals over 25 years; baseline ≥1% 5d hit rate = 50.8%.

Within OpEx Friday + post-OpEx 1–5 day window:

| Horizon | N | ≥0.5% | ≥1.0% | ≥1.5% | ≥2.0% | Median |
|---|---:|---:|---:|---:|---:|---:|
| 1d | 26 | 42% | 19% | 15% | 4% | -0.40% |
| 3d | 26 | 62% | 46% | 23% | 15% | -0.95% |
| 5d | 26 | 73% | 50% | 27% | 23% | -0.99% |
| 10d | 26 | 77% | 69% | 46% | 38% | -1.37% |

Extended filter: weekly or monthly ATR position ≥0.618 (n=21): 10d horizon had 62% hit ≥1%, 43% hit ≥1.5%, 38% hit ≥2%, median -1.25%.

Takeaways:

- Edge plays over days, not necessarily immediate dump.
- OpEx Fri + post-Monday strongest pair.
- Long-dated puts preferred over weeklies when playing the tail.
- Deep extension (wk/mo ATR ≥1.0) underperforms moderate extension at 5d but has fatter left tails at 10d.

---

## 11. Saty discretionary setup foundations

### 11.1 ORB — Opening Range Breakout

Definition: 5m or 10m opening range breakout/breakdown, using the opening range, VWAP, trend, and target ladder.

Doctrine distilled from foundational entries:

- Identify the 5m or 10m opening range.
- Prefer a clear trend/ribbon context.
- Enter on break of the opening range or on retest of a relevant trigger/level after trend confirmation.
- In bearish ORB examples, targets included premarket low, support, and -1 ATR; scaling at each target.
- Manual stops often use 34 EMA or reclaimed trigger/level depending on setup.
- Avoid first-10-minute noise; wait for opening range definition unless the setup is specifically an open trigger/GG.

Live template:

- Preconditions: OR established; price breaking/holding outside OR; ribbon/VWAP supports direction; no major chop.
- Entry: break and hold OR high/low or retest after break.
- Invalidation: close back inside range or loss/reclaim of key EMA/trigger against trade.
- Targets: PM high/low, PDC, VWAP, ATR trigger/GG levels, support/resistance.

### 11.2 Vomy / iVomy

**Vomy**: bearish setup using Pivot Ribbon, 8/13 EMA cross/ribbon flip, squeeze/compression, Phase Oscillator, and ATR targets.  
**iVomy**: inverse/bullish version.

Vomy doctrine:

- Clear trend preferred, often visible on 3m/10m ribbon.
- Conviction arrow / 13-48 confirmation adds confidence.
- Bear flag breakdown, lower highs/lower lows, or failed reclaim can support short thesis.
- Entry often occurs on pullback into ribbon / rejection at 8 EMA or 13 EMA, not chase.
- Invalidation can be reclaim of 13 EMA or relevant ribbon level against the short.
- Targets: put trigger, -1 ATR, support/gap fill; scale into targets.

iVomy doctrine:

- Bullish ribbon / inverse ribbon flip.
- Price holds previous close or key support.
- Entry on dip/retest into 8/13 EMA or recovery/close above 13 EMA.
- Targets: 50% ATR, 61.8% Golden Fib, prior high, call trigger/GG ladder.
- Take profit aggressively on lottos; Saty doctrine explicitly says to take profit on lottos at 100%.

### 11.3 Support/resistance and supply/demand

Support/resistance doctrine:

- Go long at support if price pulls back and holds.
- Go short at resistance if price rallies and rejects.
- If support breaks, short the retest where old support becomes resistance.
- If resistance breaks, long the retest where old resistance becomes support.
- Scale at the next support/resistance level.

Supply/demand doctrine:

- Identify zones with multiple rejections or lower-high/lower-low structure.
- Supply + 21 EMA/ribbon resistance can define short entry.
- Invalidation is breakout through high of supply zone.
- Targets can include call/put triggers, gap fill, and nearby support.

### 11.4 Power Play, failed breakout, divergences, patterns

Use these as secondary setup families unless current context explicitly matches them:

- **Power Play / PP**: pivot/ribbon/trend structure setup.
- **Failed breakout / FBO**: counter-trend move after breakout fails at support/resistance, often volume-confirmed.
- **Bull/Bear divergence**: PO/RSI/MACD divergence with price structure; reversal signal.
- **Bull flag / bear flag**: continuation pattern into ribbon/EMA support/resistance.
- **H Pattern, Head & Shoulders, wedges, Wyckoff**: structure/pattern context; require level confirmation.
- **Time Warp**: multi-timeframe analysis; do not mix timeframes without stating which is primary.

---

## 12. Multiday and swing Golden Gate

### 12.1 Weekly ATR / multiday GG

Conditioned on previous day daily PO:

- Bull weekly GG: 65% complete day 1, 84% by day 5.
- Bull Bilbo: prior day PO High+Rising: 74% day 1, 84% by day 5 (n=115).
- Counter bull: prior day PO Mid+Falling: 53% day 1, 78% by day 5.
- Bear weekly GG: 72% complete day 1, 83% by day 5.
- Bear Bilbo: prior day PO Low+Falling: 94% day 1 (n=54), strongest signal in studies.
- Counter bear: prior day PO Mid+Rising: 60% day 1.

**Caveat:** the multiday script was flagged in audit for a current-week variant of the same PO-join observability problem that invalidated the intraday Bilbo numbers (see §5.3), and these conditioned splits have not been rerun with a corrected join. The daily-PO conditioning here uses prior-*day* values (less exposed than intra-hour joins), but treat the exact percentages — especially the 94% cell (n=54) — as unverified, and note the intraday lesson that counter-slope "avoid" cells can invert once look-ahead is removed.

### 12.2 Monthly ATR / swing GG

Conditioned on previous week weekly PO:

- Bull monthly swing GG: 10.7% day 1, 35.1% day 5, 54.8% day 10, 73.2% day 20 (n=299).
- Bear monthly swing GG: 33.8% day 1, 58.1% day 5, 67.9% day 10, 76.5% day 20 (n=234).
- Weekly PO adds minimal edge at monthly timeframe.
- Bearish is 3x faster than bullish on day 1.
- Monthly moves take weeks.
- Full ATR only 24% bull / 42% bear by day 20.

---

## 13. Trade construction playbooks

### 13.1 Day-mode Golden Gate long/short

**Preconditions**

- Price reaches ±38.2% ATR.
- Ribbon supports direction: stacked or at least not strongly opposing.
- Time of day is favorable, especially open/first hour.
- 1h PO is not counter to direction; ideal Bilbo state if available.

**Entry**

- Immediate at ±38.2; or pullback to 10m EMA8/EMA21/ribbon for improved execution.

**Invalidation**

- 10m close back through ±23.6 trigger is primary kill signal.
- 1h EMA21 or 10m EMA48 break are serious warnings.

**Targets**

- ±50% ATR partial optional.
- ±61.8% ATR primary target.
- 78.6/full ATR only when Bilbo state/trend supports continuation.

**Avoid**

- Late bullish triggers.
- Entangled/sideways ribbon.
- Midpoint entries with poor R:R unless another setup supports it.

### 13.2 Bilbo Golden Gate momentum continuation

**Preconditions**

- GG opens at 38.2.
- 1h PO state aligns:
  - Bull: high+rising or high+falling.
  - Bear: low+falling or low+rising.
- Trend/ribbon supports direction.

**Entry**

- 38.2 immediate or EMA8/ribbon pullback.

**Invalidation**

- Trigger close break.
- PO/ribbon reversal against trade.

**Targets**

- 61.8 minimum target.
- 78.6/full ATR extension more justified, especially bearish low+falling.

**Caveat**

- Use the corrected §5.3 numbers: extreme-zone edge is real but smaller than originally published (bear low-zone ~85%, bull high-zone ~71–74%), the setup is rarer, and mid-zone slope splits carry no signal. Directional texture, not a reason for oversizing.

### 13.3 Trigger Box debit/credit plan

**Debit version**

- If price breaks call trigger in bullish context, calls/longs; stop below trigger.
- If price breaks put trigger in bearish context, puts/shorts; stop above trigger.
- Scale 50% at midrange; all/mostly out at ±1 ATR except optional runner.

**Credit version**

- If bearish box holds 30–60m, consider call credit spreads above +61.8 or +100.
- If bullish box holds 30–60m, consider put credit spreads below -61.8 or -100.
- Use ±38.2 as stop/invalidating level.

### 13.4 Compression expansion breakout

**Preconditions**

- 10m compression for at least 30m.
- EMA21/48 trend aligned.
- ATR position supports expansion direction.

**Entry**

- At expansion confirmation / break of compression range.
- For Bilbo Box (1h+ frames only), immediate break beats retest historically.

**Invalidation**

- Failure back into box/range.
- Opposite boundary break.

**Targets**

- 0.5–1R for Bilbo Box.
- ATR level ladder and support/resistance for broader compression expansion.

### 13.5 Gap fill / midpoint fill

**Preconditions**

- Gap size known.
- Trend filter known: gap is counter-trend or pro-trend.
- Price rejects continuation or loses opening direction.

**Entry**

- Break of opening range toward gap.
- Failed breakout / rejection at supply/demand.
- PDC magnet confirmed by VWAP/ribbon/PO.

**Invalidation**

- Gap continues with trend and holds above/below key level.
- Ribbon strongly supports continuation.

**Targets**

- Gap midpoint first.
- Full gap/PDC second.

### 13.6 ORB continuation

**Preconditions**

- Opening range defined.
- Clear trend / VWAP / ribbon support.
- Avoid initial 10-minute randomness unless using a separate open trigger setup.

**Entry**

- Break and hold opening range high/low.
- Retest of OR boundary or trigger in trend.

**Invalidation**

- Close back inside OR.
- Loss/reclaim of EMA/ribbon level against trade.

**Targets**

- PM high/low, support/resistance, ATR trigger/GG, ±1 ATR.

### 13.7 Vomy / iVomy

**Preconditions**

- Ribbon flip/8-13 structure in direction.
- Trend confirmation or conviction arrow.
- Pullback into ribbon or reclaim/rejection at 8/13 EMA.

**Entry**

- Vomy short: 8/13 EMA rejection, bear flag breakdown, put trigger break.
- iVomy long: 8/13 EMA reclaim/hold, previous close hold, call trigger path.

**Invalidation**

- Reclaim/loss of 13 EMA against trade.
- Failed ribbon flip / sideways market.

**Targets**

- Trigger, 50%, 61.8, ±1 ATR, support/resistance/gap fill.

### 13.8 ±1 ATR RTM

**Preconditions**

- Price extended to ±1 ATR or far from ribbon/EMA.
- PO extreme/leaving extreme, divergence, exhaustion, or fired squeeze.

**Entry**

- Confirmation of rejection/failed continuation.
- Safer after PO/ribbon confirms, not at first touch in a trend day.

**Invalidation**

- Clean continuation through ±1 ATR with trend/ribbon support.
- No exhaustion; volume expands with trend.

**Targets**

- Next ATR level back inward, VWAP, ribbon/EMA, PDC.

### 13.9 4h PO / OpEx rollover put plan

**Preconditions**

- 4h PO peak ≥80 then crosses below 80.
- OpEx Friday or post-OpEx 1–5 day window preferred.
- Weekly/monthly ATR position ≥0.618 strengthens.

**Entry**

- Use rejection/rollover confirmation; avoid expecting immediate dump.
- Prefer longer-dated puts if playing 10d tail.

**Invalidation**

- 4h PO reclaims/rises; price holds trend/ribbon and no downside follow-through.

**Targets**

- 1% / 1.5% / 2% drawdown bands over 5–10d, plus ATR levels and daily EMA21.

### 13.10 Call→put morning reversal

**Preconditions**

- Call trigger hit before noon.
- Price crosses below PDC.
- Put trigger reached before noon.
- 1h PO state assessed.

**Entry**

- At put-trigger touch/retest or after PDC loss confirms.

**Invalidation**

- PDC reclaim with failure to extend lower.
- Bullish 1h expansion reduces downside continuation.

**Targets**

- Downside GG open/complete.
- PDC recovery possibility must be respected.
- -1 ATR only ~18.5% overall; do not assume.

---

## 14. Backtested edges quick table

| Edge / study | Best live-use takeaway |
|---|---|
| Level-to-level | Trigger→38.2 = 67–70% live cohort (79% pooled); 38.2→61.8 = 52–57% live cohort (62.5–65.4% pooled); full ATR cumulative ~2% |
| Baseline GG | Bull GG 62.5%; Bear GG 65.4% |
| Bilbo GG (corrected) | Bull PO high-zone 71–74%; Bear PO low-zone 85–87%; mid-zone slope splits carry no signal |
| Bilbo continuation | Unverified post-fix; do not size on the published ladder |
| 60m vs 10m PO | Unverified post-fix; corrected §5.3 tables are the only clean Bilbo numbers |
| GG entry | Immediate/EMA8 entries solid; 1h EMA21 best R:R but lower hit |
| GG invalidation | 10m trigger close break cuts completion by ~39pp |
| GG timing | Open triggers ~86–88%; late bull weak |
| Trigger Box | Box held 1h → ~80% GG open |
| Trigger Box spreads | 1h-held box: ±61.8 spreads ~92–93.6%; ±100 ~97.5–98.7% |
| Gap midpoint | <0.25% gaps fill midpoint ~94–95% day 1 |
| Weekly bear Bilbo | Prior daily PO low+falling bear weekly GG = 94% day-1 (n=54, unverified post-fix) |
| Monthly GG | Bear monthly moves faster; monthly moves take weeks |
| Compression expansion | 180m+ compression + bullish EMA21>48 = 83.7% bullish expansion |
| 4h PO + OpEx | Extended OpEx rollover has 10d tail edge; not immediate |
| Bilbo Box | 1h+ frames only; immediate beats retest; retest underperforms |
| Call trigger confirmation | Clean 3m close above trigger → 38.2 hit 97.1%; median 18m |
| Call→put reversal | PDC recovery 73.7%; downside GG open 75.3%; -1 ATR 18.5% |
| EMA21 reversion | >4% above daily EMA21 touched EMA21 within 28d in 50/50 episodes |

---

## 15. Options execution and risk translation

Underlying-level probabilities do not automatically become option-trade probabilities. A setup can hit the underlying target and still produce poor option results if entry is late, implied volatility collapses, spread is wide, or strike selection is poor.

### 0DTE / 1DTE implications

- Fast targets matter more than distant targets. If a study says median winner reaches target in 18 minutes, that matters for 0DTE.
- Time decay punishes late-day trades, especially late bullish GG attempts.
- Use tighter invalidation with 0DTE. A 10m close back through trigger can be too slow for far OTM contracts; watch price action and spread behavior.
- Prefer taking partials at the first clean level when options are inflated.

### Debit trades

- Best for clean directional setups with nearby targets: call-trigger confirmation, early GG, Bilbo GG, ORB continuation, compression expansion.
- Avoid buying premium in chop around PDC/VWAP unless the setup is explicitly a volatility expansion play.
- If the underlying target is only 0.5R away but option premium already expanded, reward/risk may be poor despite good underlying odds.

### Credit spreads

- Best when the setup says a level is unlikely to be reached: trigger box held 30–60 minutes, especially selling beyond ±61.8 or ±100.
- Do not sell spreads into active trend expansion against the short strike.
- Define stop around the level that invalidates the box logic, often ±38.2.
- Higher win rate does not equal better expected value if loss size is too large; manage spread width and max loss.

### Position sizing

- Strong historical edge: still size for failure.
- Audit caveat or small sample: reduce size or treat as context only.
- Late-day, event-driven, or high-IV conditions: reduce size or require extra confirmation.
- If invalidation is vague, the trade is not ready.

---

## 16. Caveats and data-quality notes

1. **Research, not advice**: all stats are historical SPY research, not guarantees.
2. **SPY vs SPX**: backtests are on SPY; SPX options execution/liquidity/settlement differ.
3. **1h PO caveat**: hourly Phase Oscillator has TradingView mismatch due to extended-hours ATR inflation. Use 60m PO because it tested predictive, but mark uncertainty.
4. **Bilbo GG look-ahead fix (2026-04-26)**: the original Bilbo GG study classified 10m triggers by the still-unfinished 1h candle's PO (up to 60 min of forward leakage). §5.3 now carries the corrected rerun. Still unverified post-fix: the continuation-beyond-61.8 ladder, the 60m-vs-10m PO comparison, and the weekly Bilbo conditioned splits in §12.1.
5. **1d Bilbo Box underpowered**: daily Bilbo Box sample n=279; bullish drift drives much of higher-TF bull edge.
6. **Upward drift bias**: 25-year SPY history favors bull breaks on higher timeframes.
7. **Extracted doctrine**: some Saty claims come from OCR/vision extraction of screenshots. Use as doctrine/terminology, not validated quant.
8. **Avoid example contamination**: example chart tickers, dates, visible panels are not rules unless doctrine text explicitly states them.
9. **Cohort pooling, not just conflicting numbers**: older level-to-level framings (Trigger→38.2 = 80%, 38.2→61.8 = 69%) pooled gap-over days (trivially 100%) with live intraday crosses. The 2026-04-26 rerun separates them — see §5.2. For live decisions use the intraday-cross cohort: Trigger→38.2 ≈ 67–70%, 38.2→61.8 ≈ 52–57%, full ATR cumulative ~2%.
10. **Execution matters**: options spreads, 0DTE greeks, slippage, IV, and strike selection can dominate the theoretical underlying move edge.

---

---

## 17. Standalone setup scoring rubric

Use this rubric to keep live analysis disciplined. Do not mechanically add scores as if they guarantee an outcome; use them to expose whether a trade has enough confluence.

### Directional confluence

| Factor | Bullish read | Bearish read | Neutral / avoid |
|---|---|---|---|
| Price vs PDC | Above and holding | Below and holding | Chopping around PDC |
| ATR ladder | Above call trigger/GG path | Below put trigger/GG path | Between levels with no trigger |
| VWAP | Above and reclaiming dips | Below and rejecting bounces | Mean-reverting through VWAP |
| 10m ribbon | Stacked/up-sloping | Stacked/down-sloping | Entangled/flat |
| 1h PO | High/rising for bull GG | Low/falling for bear GG | Mid against direction |
| Compression | Expansion aligns with trade | Expansion aligns with short | Box unresolved |
| Time of day | First hour supports GG | Bearish can persist later | Late bullish trigger is weak |

### Trade-quality buckets

- **A setup:** named setup active, level trigger present, higher-timeframe context aligned, invalidation clear, target nearby with acceptable reward/risk.
- **B setup:** named setup active but one major confluence missing; size smaller or demand better entry.
- **C setup:** mostly thesis, not enough structure; observe or paper-plan only.
- **No trade:** no named setup, chop around PDC/VWAP, conflicting ribbon/PO, or invalidation too wide.

### Minimum trade ticket

Before entering, define:

1. Setup name.
2. Direction.
3. Entry trigger.
4. Stop/kill signal.
5. First target.
6. Second target or runner rule.
7. Time stop.
8. Why this is not chop.
9. What evidence would flip the thesis.

---

## 18. Compact live-analysis template

Use this when the trader sends “SPX” or asks for a live plan:

```text
Context
- Product/timeframe:
- Price vs PDC:
- ATR grid: trigger / 38.2 / 50 / 61.8 / full ATR:
- Premarket high/low:
- OR high/low:
- VWAP:
- Ribbon/EMA stack:
- PO state: 10m / 1h / 4h / daily:
- Compression/squeeze:
- Event context:

Active setup(s)
1. Setup:
   Preconditions met:
   Missing confirmation:
   Entry trigger:
   Invalidation:
   Targets:
   Historical edge:
   Confidence:

If/then plan
- Bull case:
- Bear case:
- No-trade/chop case:

Risk notes
- What would make this wrong:
- Time-of-day decay:
- Stats caveat:
```

---

## 19. One-screen decision hierarchy

1. **Higher TF**: daily/4h/1h PO, daily EMA21 distance, weekly/monthly ATR extension.
2. **Day levels**: PDC, trigger, 38.2, 50, 61.8, full ATR, PM high/low, OR high/low, VWAP.
3. **Regime**: trend, compression, chop, event/OpEx.
4. **Setup**: GG, Bilbo GG, trigger box, gap, ORB, Vomy/iVomy, RTM, compression/Bilbo Box.
5. **Entry**: exact candle/level condition; no vague chasing.
6. **Invalidation**: exact level and close requirement.
7. **Targets**: nearest level-to-level ladder; partials; time expectation.
8. **Risk**: late-day decay, 0DTE Greeks, audit caveats, no-trade option.

If none of the above identifies a clean setup: **do nothing**.
