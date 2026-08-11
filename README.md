# Pipeline Risk Plot — Leak and Rupture

A single-file, offline tool that turns corrosion rates, leak history and consequence estimates into a risk position for each line class in a mixed pipeline fleet, plotted separately for leak and rupture against acceptance limits you set.

**File:** `pipeline_risk_plot.html` — open it in any browser. No install, no internet, no external libraries. Nothing is uploaded anywhere.

---

## What problem this solves

A mixed fleet spans four or five orders of magnitude in consequence. A 4" flowline leak and a gas-lift trunkline outage both land in the same cell of a 5×5 matrix and look comparable when they are not. This tool puts probability and consequence on absolute log scales in common units — failures per mile-year and dollars per event — so a gas line and an oil line can be compared on the same chart, and so lines of equal risk fall on the same diagonal.

It separates leak from rupture because they are different limit states, not different severities. A low-pressure flowline is mathematically incapable of rupturing from corrosion; a high-pressure trunkline bursts at around 90% wall loss. Two plots make that visible in a way one plot cannot.

---

## Quick start

1. **Open the file** and click **Load worked example** to see a populated eight-entry fleet.
2. **Set your limits** in Step 1 — the dollars-per-mile-year below which you would not act, and above which you must. These colour the charts.
3. **Describe your lines** in Step 2 — one row per distinct combination of size, wall and pressure.
4. **Price a failure** in Step 3 — once for a leak, once for a rupture.
5. **Generate risk plots.**

Then read the results in this order: the two charts, the detail table, the model check, the recommendations, and the worked calculation for anything you need to defend.

---

## What you need before you start

| | What | Where it comes from |
|---|---|---|
| 1 | Diameter, wall, grade, normal operating pressure | Line list. Grade → SMYS: Gr B 35,000 psi, X42 42,000, X52 52,000, X60 60,000 |
| 2 | Some evidence of condition — **any one of three** | See evidence bases below |
| 3 | Roughly what a failure costs — cleanup, lost production, repair | Environmental and operations. Order of magnitude is enough |
| 4 | Your risk limits in dollars per mile-year | Start with the defaults and adjust once you see where the fleet lands |

Use **normal operating pressure, not MAOP**. Using MAOP on a line that runs well below it overstates rupture badly and is the most common way this kind of model goes wrong.

---

## Evidence bases — you do not need ILI on every line

Each row picks one of three routes, and the maths changes to suit. Unused fields grey out.

| Basis | What it needs | What it does |
|---|---|---|
| **Leak history** | Leak count, rupture count, mile-years of exposure | Probability comes straight from your record: `λ = (failures + 0.5) / mile-years`. No wall thickness, no growth rate, no assumption about mitigation. The half is a Jeffreys prior so a class with no leaks does not read as zero |
| **ILI** | Measured depth, tool tolerance, corrosion rate, rate scatter, feature count | Full structural route through Modified B31G and the growth model |
| **Estimated** | Corrosion rate and line age | Depth inferred from rate × age, uncertainty widened to ±25% to represent honest ignorance |

**Use leak history wherever you have it.** For flowline populations it is stronger evidence than any corrosion model, and it is the portion of your fleet you will never have to defend.

---

## Handling a large fleet

You do not want 3,000 rows. Group lines that share size, wall, pressure and service — most fleets collapse to 15–40 rows.

- The **Lines** field carries how many lines are in the group.
- **Position on the chart** is set by one representative line: "how bad is a line of this type."
- **Marker size** and **fleet risk** scale with count × length: "how much of my risk sits here."

Count deliberately does not move the point. A group of 200 flowlines and a group of 2 trunklines can land in the same place with very different totals.

**Different sizes in one category?** Enter each combination as its own row with a shared **Category** name, then switch *Plot points as* to rolled-up. Risk sums across the group and the point is placed to match that total, so the rollup can never quietly disagree with the detail table. You cannot average a 12"×0.250 and a 12"×0.375 — hoop stress goes as D/t and burst depth is nonlinear in it.

---

## Reading the plots

- **X axis** — probability of failure per mile-year, log scale
- **Y axis** — consequence in dollars per event, log scale
- **Bands** — green acceptable, amber tolerable, red unacceptable, from your limits. Because risk is probability × consequence, lines of constant risk are straight diagonals on log-log axes
- **Circles** liquid, **squares** gas
- **Marker area** ∝ exposure
- **Numbers** match the detail table. Click a marker to highlight its row; click a row to ring the marker on both plots

Entries marked *leak only* cannot burst at their operating stress — their whole risk sits on the left plot.

An entry low on the leak plot but present on the rupture plot is often the one to fund. Rupture consequence typically runs an order of magnitude above leak, so a probability a thousand times smaller can still dominate.

---

## Method

Full equations, with the intermediate values printed, are in the **Reference** and **Audit** sections of the tool. In summary:

**Remaining strength** — Modified B31G (0.85dL area method), with the Folias bulging factor. Solving `P_f = P_operating` for depth gives the critical depth the tool reports. Where that solves above 100% of wall, the defect perforates before it can burst and the line is leak-only.

**Growth and limit state** — depth grows linearly from the measured value. Two limit states run in parallel: a short pit at 100% wall produces a leak; a long defect at the burst depth produces a rupture. "Long" is taken as L ≈ 3√(D·t).

**Probability** — three inputs are treated as distributions rather than fixed numbers: corrosion rate (lognormal, at the COV you set), current depth (normal, from the inspection tolerance at 80% certainty), and burst equation error. The tool integrates the rate tail analytically and samples the other two, which is what lets it resolve probabilities far below one in the number of draws. The plotted number is the annual conditional hazard.

**Aggregation** — features combine as an independent series system, evaluated in log space so very small probabilities do not cancel to zero. Corrosion then combines with a background rate for non-corrosion threats, and the result is divided by length.

**Risk** — `PoF × (V·c + Q·D·m + R)`. Everything is priced in dollars because that is the only common unit in which a gas line and an oil line can be compared.

### On the random draws

The draw count is a setting on the arithmetic, **not a count of anything real**. It is not lines, not features, not miles. A fleet of 3,000 and a fleet of three use the same setting, because the draws are spent on one representative feature at a time.

If asked whether you simulated the fleet, the accurate answer is no. What you did was calculate each line class separately, allowing for the fact that depth and growth rate are not known exactly. That sentence holds up under pressure.

---

## Defending the model

The **Model Check** section compares predicted failures against your actual record and tells you what to say.

**Lead with this:** for every line running on leak history, there is no model to defend — the probability *is* your leak rate. That shrinks the claim you have to defend down to the portion of the fleet with no failure history.

**The number to bring:** predicted leaks per year against actual. Within a factor of two is consistent. Two to three is acceptable if you say so. Beyond that, apply the calibration factor or put the discrepancy on the slide yourself. The calibration factor scales model-derived probabilities only and never touches leak-history lines.

**Expect the rupture question.** "We've never had a rupture, so why is rupture most of the risk?" The panel computes the Poisson answer: if the model expects 0.81 ruptures over your exposure, the chance of observing zero is 45%. A clean record is not evidence against the model — it is the most likely single outcome. The other half of the answer is that rupture dominates on consequence, not frequency.

**Separate two claims.** Count agreement proves the magnitude is roughly right. It says nothing about whether the *ranking* is right, which is what the plot exists for. Test that by checking where your actual failures sat in the ordering. A model off by a factor of three that still puts failures in the top quintile is useful for allocating budget; a perfectly calibrated one that ranks randomly is not.

**Know what would change your mind.** Predicted-to-actual off by more than tenfold; leaks appearing on lines ranked in the bottom half; or the answer turning on an input nobody can source. Being able to state these is what makes the rest credible.

---

## Recommendations

The tool ranks actions per line based on where it actually sits, built around a distinction worth putting on your slides:

- **Inspection** reduces *uncertainty* — it does not reduce risk, but it decides whether the spend below is justified
- **Chemical and cleaning programmes** reduce the *rate*
- **Pressure reduction** moves the burst depth directly, and enough of it converts a rupture-capable line to leak-only
- **Valve spacing, leak detection, redundancy and containment** reduce *consequence*

Those are four different budgets. The engine decomposes consequence into cleanup, deferred production and repair, and points at whichever component exceeds 40%.

---

## Import and export

- **Paste from Excel** — select the block including headers, copy, paste into the box. Tab or comma separated both work.
- **Download blank template** — gives the exact column order.
- **Export current inputs** — a CSV you can keep alongside the source data.
- **Download results CSV** and **PNG of each plot** for reports.

A single file with no external libraries cannot open a binary `.xlsx`. Pasting achieves the same thing in one step.

---

## Limitations

- **Screening and comparison tool, not fitness-for-service.** It works on a representative feature per line class, not on your dig list. Use a full assessment to decide whether a specific joint stays in the ground.
- **Features are treated as independent.** Real corrosion is spatially correlated, so the series aggregation is conservative.
- **Feature count means features of comparable depth to the deepest** — not every feature on the line. Counting shallow features makes the answer conservative without making it better.
- **The rupture side may be unvalidatable.** If you have had no ruptures in the asset's history, no amount of data will validate it empirically. Verify the equations, benchmark the inputs, and say so.
- **Consequence cannot be validated against outcomes** — too few events. Treat it as a documented, agreed, sensitivity-tested assumption set, never as a measured quantity. Cleanup rate and netback belong to environmental and operations, not to integrity; settle them before the meeting.
- **Well-maintained lines may be background-driven.** If metal loss is decades from any limit, what you see is the non-corrosion rate. The tool flags this in the *Driving threat* column. The implication is uncomfortable but defensible: for those lines, inspection buys certainty rather than risk reduction.

---

## Sources

- ASME B31G, *Manual for Determining the Remaining Strength of Corroded Pipelines*
- Kiefner and Vieth, *A Modified Criterion for Evaluating the Remaining Strength of Corroded Pipe*, AGA/PRCI PR 3-805, 1989
- Folias, on the bulging of an axially cracked pressurised shell, 1965
- DNV-RP-F101, *Corroded Pipelines* — the main alternative capacity equation; generally gives higher allowable depths, worth running as a cross-check
- API 579-1 / ASME FFS-1, Parts 4 and 5
- API RP 581, *Risk-Based Inspection Methodology*
- API RP 1160 (liquids) and ASME B31.8S (gas)
- Ahammed and Melchers, *Engineering Structures*, 1997 — limit-state formulation
- Caleyo et al., *Corrosion Science*, 2009 — pitting depth and rate distributions
- NUREG/CR-6823, *Handbook of Parameter Estimation for Probabilistic Risk Assessment* — Jeffreys prior for zero-failure data
- API 1163 — inspection tolerance conventions
- 49 CFR 192.917 and 195.452; PHMSA incident data for background rates

Standards are revised. Check the current edition before citing an equation in a formal assessment.

---

## Technical notes

Plain HTML, CSS and JavaScript in one file. No frameworks, no CDN, no network calls. Works offline and can be emailed or put on a shared drive as-is. Tested logic includes the Modified B31G solver against hand calculation, the normal tail approximation against published quantiles to 1e-7, and the Poisson helper against closed-form values.
