---
name: drift-buster
description: >
  Systematic IOB curve integrity audit. Catches when an insulin activity chart drifts from
  published pharmacokinetic sources — wrong PK parameter lookup, zero-start violations,
  peakless-insulin peak violations, stacking arithmetic errors, or axis/legend mismatches.
  Designed for clinical insulin visualisation tools but adaptable to any PK curve renderer.

  AUTO-ACTIVATES on mention of any insulin brand or protocol keyword. Always run before
  editing anything insulin-related, even if the user hasn't explicitly asked for it.

  Insulin brands (any mention triggers): Levemir, Actrapid, Tresiba, Fiasp, Humalog,
  NovoRapid, Insulatard, Toujeo, Lantus, Basaglar, Lyumjev, Degludec, Glargine, Detemir,
  NPH, Humulin N, Regular insulin, Insulin R, ultra-rapid, rapid-acting, long-acting,
  intermediate-acting, basal insulin, bolus insulin.

  Protocol keywords (any mention triggers): TID, BID, OD, once-daily, twice-daily,
  three-times-daily, SC, IM, subcutaneous, intramuscular, basal regimen, bolus regimen,
  split-dose, mixed regimen, correction dose, sliding scale, insulin regimen, dosing
  protocol, insulin plan, paediatric insulin protocol.

  PK resource keywords (any mention triggers): Plank 2005, PMID, DOA, duration of action,
  pharmacokinetics, PK model, PK curve, PK profile, PK source, IOB engine, insulin engine,
  albumin_bound, decay_model, peak rate, onset, offset, PK reference, pk_source.

  Trigger phrases: "drift buster", "drift-buster", "chart looks wrong", "iob looks off",
  "check the curve", "audit the iob", "bust drift", "something is wrong with the chart",
  "dose not matching", "iob drifting", "curve looks wrong", "pk looks wrong",
  "check insulin", "check the pk", "validate the curve".

  Args: none = audit current branch · <insulin name> = scope to one insulin profile.
---

# Drift Buster — IOB / PK Curve Integrity Audit

Use when: an insulin activity / IOB chart looks wrong, a dose change breaks the curve,
after any edit to the PK engine, or after updating pharmacokinetic parameters.

---

## What is drift?

Drift = the displayed curve deviating from what the source pharmacokinetic paper predicts
for the given dose / weight / timing combination.

**Five known drift causes:**
1. PK parameter lookup off by a row (weight-band or dose-band boundary error)
2. IOB curve starting at zero when a prior-cycle dose exists
3. Peakless insulin (e.g. Tresiba / Degludec) showing a peak
4. Multi-dose stacking computed on wrong time axis (minutes vs hours, UTC vs local)
5. Rendered total ≠ sum of per-dose curves (floating-point accumulation or missed dose)

---

## Configuration (fill in before running)

| Field | Your value |
|---|---|
| PK source paper | e.g. Plank 2005 (PMID 15855574) |
| PK engine file | e.g. `src/insulin/iob-series.ts` |
| IOB totals entry point | e.g. `src/insulin/index.ts` |
| Chart component | e.g. `src/components/IOBChart.tsx` |
| Peakless insulins in your system | e.g. Tresiba, Toujeo |
| Multi-cycle window (hours) | e.g. 36 |

---

## Audit sequence

Run every check in order. Stop and report at the first ✗.

### Step 1 — Verify PK source

Read the PK parameter table in the engine file.

Check:
- The `pk_source` / citation field matches the paper you cited above.
- Duration-of-action (DoA) for the test dose matches the paper's Table for the given
  weight or dose band.
- `t_peak` is present and non-zero for peaking insulins; absent or 0 for peakless ones.

### Step 2 — Zero-start check

Check:
- When the visualisation window spans > 24 h (multi-cycle), the curve at the left edge
  (T = 0) is > 0 — a prior-cycle dose must seed non-zero activity.
- If the curve starts at exactly 0 with no seeding: ✗ DRIFT — cite file:line.

### Step 3 — Peakless insulin check

For every insulin configured as peakless:

Check:
- The activity curve is flat (± 5 %) across the entire DoA window.
- Any hump > 5 % above the plateau: ✗ DRIFT — cite file:line and delta value.

### Step 4 — Stacking arithmetic

For multi-dose regimens (≥ 2 doses):

Check:
- Each dose's per-dose curve is offset from its administration timestamp in hours
  (not minutes, not seconds, not raw epoch ms).
- Sum the per-dose arrays element-by-element. Confirm the total matches the
  `getTotalIOBSeries()` (or equivalent) output to within floating-point rounding
  (< 0.001 U/h error per point).
- Common bugs: `doseTime × 60` when `doseTime` is already in hours; UTC epoch vs
  local-midnight-relative offset.

### Step 5 — Visual axis & legend

Check the chart component:

| Element | Required |
|---|---|
| Y-axis label | Includes unit (e.g. `U / hour` or `U/h`) |
| Legend items | One entry per series: total IOB, per-dose, pressure bands |
| Dose marker format | Consistent: time · dose · unit |
| Pressure zones | If shown, labelled (e.g. Low / Moderate / High / Stacking risk) |

### Step 6 — Report

For each step: `✓ pass` or `✗ DRIFT — [step name] — [file:line] — [delta]`.

**Clean result:**
```
No drift detected. Curves match [PK source] for [insulin] [dose]U [weight]kg.
All 5 checks passed.
```

**Drift found:**
```
DRIFT DETECTED — Step [N]: [check name]
File: [path:line]
Expected: [value]
Actual:   [value]
Fix: [specific one-sentence instruction]
```

---

## Triage priority

Peakless-insulin peak violations and zero-start violations are **blocking** — fix these
before any other task. All other drift is high-priority but non-blocking.

---

## After the audit

If drift found → fix the specific file:line, re-run Drift Buster to confirm clean.
If clean → proceed with original task.
