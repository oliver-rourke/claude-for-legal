# Intake triage rules (check 1 of 2)

Check 1 asks one question: is this applicant worth an attorney's time right now, or is there an instant stop, a wait, or something to follow up first? It runs on the intake answers alone, before any conviction is looked at. Check 2, the per-conviction screen in `skills/eligibility-screen/references/screening-bands.md`, runs when check 1 says proceed or review, or when the attorney asks for it anyway.

The order and the endings come from The Access Project's intake flow (form logic checked 2026-09-24; `intake-flow.md` has the diagram) and its triage formula, with program-specific wording removed. The legal effect of each answer cites `screening-bands.md` section 1. Both `/record-clearance-legal:intake-import` (step b) and `/record-clearance-legal:eligibility-screen` (Step 2) apply this file; neither keeps a second copy.

## Outcomes

| Outcome | Meaning | What happens next |
|---|---|---|
| `PROCEED TO SCREEN` | No stop, no wait, nothing unsure on a field the triage turns on | Check 2 runs |
| `NEEDS ATTORNEY REVIEW` | A "not sure" on a field the triage turns on with no prior clearance, or a registration answer other than no | Check 2 runs with every band provisional |
| `NOT ELIGIBLE NOW` | A wait applies to the whole applicant: a pending case, probation still running, or a waiting period after parole or supervision with no other case to work on | The reason, the recheck trigger or date, the referral; check 2 only on request |
| `NOT ELIGIBLE FOR THE PROGRAM` | A required program gate failed, a prior review said no, or the consents are incomplete | The reason and the profile's "When a gate fails" rule; check 2 only on request |

Triage is routing for the program, not a legal determination. The words "eligible" and "ineligible" never stand alone in the output.

## Order of evaluation

Apply in this order. The first stop wins, but keep collecting flags so the output lists every follow-up.

1. **Program gates** (`program_gates.*` against the profile's `## Program eligibility gates`). A required gate answered `no` → `NOT ELIGIBLE FOR THE PROGRAM`, reason "[gate]"; apply the profile's "When a gate fails" rule. `unknown` on a required gate → follow-up flag, continue.
2. **Prior review decision** (`program_gates.prior_review_decision`, a staff field some programs keep). `not_eligible` → `NOT ELIGIBLE FOR THE PROGRAM`, reason "prior review decision". `cleared` → remember it; it cancels the not-sure review in rule 9. `none` → continue.
3. **Pending case** (`status.pending_case`). `yes` → `NOT ELIGIBLE NOW`, recheck "when the pending case resolves" (section 1: PC 1203.4(a)(1), PC 1203.41(a)(3)). `unsure` → review flag, continue.
4. **Current supervision** (`status.current_supervision`).
   - `probation`: when `status.fire_camp` is `yes` for that same case → continue with the fire camp referral flag (PC 1203.4b(b)(4): no need to complete supervision). Otherwise `NOT ELIGIBLE NOW`, recheck "when probation ends", referral: early termination of probation (PC 1203.3).
   - `parole`: `status.other_convictions: yes` → continue with the note "the parole case waits two years after release; the other cases can be screened now" (PC 1203.41(a)(2)); `no` → `NOT ELIGIBLE NOW`, recheck "two years after the parole release date"; `unknown` → review flag, continue.
   - `mandatory_supervision` (the form's "community supervision"): the same pattern with one year after supervision ends (PC 1203.41(a)(2) for a PC 1170(h)(5)(B) sentence), plus the flag "confirm mandatory supervision versus post-release community supervision, which follows a prison term and waits two years".
   - `prcs`: the same pattern with two years.
   - `unsure` → review flag, continue.
5. **Supervision ended in the past three years** (`status.supervision_ended_within_3_years` with `status.supervision_ended_bucket`).
   - `parole` with `under_2_years`: `other_convictions: yes` → continue with the note; `no` → `NOT ELIGIBLE NOW`, recheck = release month + 24 months when the month is known (show the arithmetic, tag `[model calculation — verify]`), otherwise "two years after the parole release date".
   - `mandatory_supervision` with `under_1_year`: the same with 12 months.
   - `2_years_or_more`, `1_year_or_more` or `none` → continue.
   - `unsure` on the field or the bucket → review flag, continue.
6. **Registration** (`status.registration_290`). `current`, `terminated` or `unsure` → review flag "registration bars a state prison felony under PC 1203.41(a)(6) and the PC 1203.4(b) sections"; continue.
7. **Trafficking or DV** (`status.trafficking_or_dv_remedies`). `yes` → follow-up flag "PC 236.14 and PC 1203.49 remedies; attorney conversation"; `unknown` → one-clause follow-up; never a stop.
8. **Consents** (`consents.*`, the five acknowledgments). When the source is a form or export that carried the consent questions: any `no` or blank, unless the profile says consents are optional → `NOT ELIGIBLE FOR THE PROGRAM`, reason "consents incomplete: [which]"; when the profile says optional → follow-up flag. When the source had no consent questions at all (prose, a RAP sheet excerpt, the interview, a record with `consents` absent), the consents are `unknown` and the result is the follow-up "collect the five consents before any filing", never a stop.
9. **Not-sure review.** Any review flag from rules 3, 4 or 5 with no `cleared` prior decision → `NEEDS ATTORNEY REVIEW`. A `cleared` decision cancels those flags; the registration flag from rule 6 still gives `NEEDS ATTORNEY REVIEW`.
10. Otherwise `PROCEED TO SCREEN`.

RAP sheet on hand (`status.has_rap_sheet`) is never a stop: `yes` adds the next step "the attorney reviews the RAP sheet"; `no` adds "request the RAP sheet from the Department of Justice" with the profile's RAP sheet guidance.

## Output line

One line, the same in both skills:

`**Check 1, intake triage:** [OUTCOME]. [reasons, in the order found]. [follow-ups]. [recheck trigger, or the date with its arithmetic and [model calculation — verify]].`

Synthetic examples:

- `**Check 1, intake triage:** PROCEED TO SCREEN. Program gates met; no pending case; no supervision now or in the past three years; not registered; consents complete. Follow-up: RAP sheet not on hand, request it from the DOJ.`
- `**Check 1, intake triage:** NEEDS ATTORNEY REVIEW. Pending case answered not sure, no prior clearance; everything else clear. Check 2 runs with every band provisional.`
- `**Check 1, intake triage:** NOT ELIGIBLE NOW. Released from parole 2025-03, under two years ago, no other convictions: recheck 2025-03 + 24 months = 2027-03 [model calculation — verify] (PC 1203.41(a)(2)).`
- `**Check 1, intake triage:** NOT ELIGIBLE FOR THE PROGRAM. Consents incomplete: record sharing and data use not acknowledged. Per the profile, send the consent form again before any screening.`

## What triage never does

- Decide a conviction. Check 2 does that, item by item, with the statute.
- Treat `unsure` as `no`. A hedge is a review flag or a follow-up, never a pass.
- Compute a date from a bucket alone. "Less than two years ago" gives a trigger; a month gives a date.
- Ask the trafficking or DV question. It records what the form collected.
