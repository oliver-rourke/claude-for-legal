# Screening bands and triggers

The screen produces five band strings and nothing else. Use them exactly:

| Band | Meaning | Who acts |
|---|---|---|
| `LIKELY ELIGIBLE` | Every statutory threshold the card names is met on the facts in the record | Attorney confirms, then the case goes to full analysis or petition work |
| `NOT ELIGIBLE NOW` | A threshold is not yet met and the record says when it will be | Staff log the recheck date |
| `NEEDS ATTORNEY REVIEW` | A fact is missing or unsure, a discretionary clause is the only route, or an exclusion may apply | Attorney decides |
| `AUTOMATIC RELIEF MAY APPLY` | A flag added to an item, never a band on its own: PC 1203.425 may already have granted relief | Whoever reads the RAP sheet checks for the "relief granted" note |
| `NOT SCREENED` | A relief type this version does not screen, named with its referral target | Referral per the profile |

Sources: PC 1203.4 and PC 1203.41 text fetched from leginfo.legislature.ca.gov on 2026-10-03 and re-fetched from california.public.law on 2026-10-04; PC 1203.425 text fetched from california.public.law on 2026-10-04. All three `[statute / regulator site]`, last confirmed 2026-10-04. A report states that date once, in the reviewer note, and tags each quoted rule `[statute / regulator site]`. Subdivision pinpoints are given on every rule; the full texts are summarized in `relief/`.

## 1. Client-level gates

Run these first, from `intake.status`. They decide the packet before any case is looked at. The order of evaluation, the program gates, the prior review decision and the consents live in `skills/intake-import/references/triage-rules.md` (check 1), which applies this table; this section states the legal effect of each answer.

| Field | Value | Effect | Rule |
|---|---|---|---|
| `pending_case` | `yes` | Packet `NOT ELIGIBLE NOW`; recheck trigger "when the pending case resolves" | PC 1203.4(a)(1) requires that the person is not then "charged with the commission of an offense"; PC 1203.41(a)(3) requires that the person is not "charged with the commission of, an offense" |
| `pending_case` | `unsure` | Packet `NEEDS ATTORNEY REVIEW`; every item band is provisional | same clauses; an unsure answer cannot clear them |
| `current_supervision` | `probation` | Packet `NOT ELIGIBLE NOW` for every 1203.4 and 1203.41 item; add `NOT SCREENED` lines for early termination of probation (PC 1203.3) and fire camp relief (PC 1203.4b, which does not require completing probation, 1203.4b(b)(4)) | PC 1203.4(a)(1) "on probation for an offense"; PC 1203.41(a)(3) "on probation for" |
| `current_supervision` | `parole`, `mandatory_supervision` | Every 1203.41 item `NOT ELIGIBLE NOW`; every 1203.4 item `NEEDS ATTORNEY REVIEW` with `[review]` "is this supervision 'serving a sentence' for 1203.4(a)(1)?" | PC 1203.41(a)(3) "not on parole or under supervision pursuant to subparagraph (B) of paragraph (5) of subdivision (h) of Section 1170" |
| `current_supervision` | `prcs` | Every 1203.41 item and every 1203.4 item `NEEDS ATTORNEY REVIEW` with `[review]` "is post-release community supervision 'serving a sentence' for PC 1203.41(a)(3) and PC 1203.4(a)(1)? Subdivision (a)(3) names parole and 1170(h)(5)(B) supervision only" | PC 1203.41(a)(3); PC 1203.4(a)(1) |
| `current_supervision` | `unsure` | Packet `NEEDS ATTORNEY REVIEW` | |
| `registration_290` | `current`, `terminated`, `unsure` | Every prison-sentence item `NEEDS ATTORNEY REVIEW`; every 1203.4 item `NEEDS ATTORNEY REVIEW` | PC 1203.41(a)(6): a state prison felony qualifies only if it "did not result in a requirement to register as a sex offender pursuant to Chapter 5.5 (commencing with Section 290)"; PC 1203.4(b) lists excluded offenses |
| `fire_camp` | `yes` or `unsure` | Add a `NOT SCREENED` line: PC 1203.4b; note it covers Conservation Camp hand crews, county hand crews and institutional firehouses and lists its own excluded crimes in 1203.4b(a)(1)(A) to (H) | |
| `trafficking_or_dv_remedies` | `yes` | Add a `NOT SCREENED` line: PC 236.14 and PC 1203.49; add a staff follow-up flag | |
| `trafficking_or_dv_remedies` | `unknown` or `unsure` (the form did not ask) | No effect on any band and never a reason for `NEEDS ATTORNEY REVIEW`; add one staff follow-up clause under Next steps. The screen does not ask this question itself | |
| `program_gates` | any `no` | Report "program gate not met" separately from the legal screen; apply the profile's `When a gate fails` rule | program rule, not law |

## 2. Per-case routing

For each entry in `intake.cases`:

- `probation_granted: yes` with `probation_outcome` in `completed`, `terminated_early`, `completed_with_violation`, `ongoing` routes to the PC 1203.4 rules in section 3.
- `probation_granted: yes` with `probation_outcome: revoked` and `offense_type: felony` routes to the PC 1203.41 rules in section 4, reading `revocation_custody` as the sentence type. With `offense_type` misdemeanor or infraction it is `NEEDS ATTORNEY REVIEW`: the PC 1203.4 discretionary route and PC 1203.4a are both possible, and 1203.4a(a) speaks of a defendant "not granted probation" `[review]`.
- `probation_granted: no` with `offense_type` in `misdemeanor`, `infraction`, `felony_reduced_to_misdemeanor` routes to the PC 1203.4a rules in section 3a.
- `probation_granted: no` with `offense_type: felony` and `sentence` in `jail`, `split_mandatory_supervision`, `prison` routes to section 4; with `sentence: fine_only` it is `NEEDS ATTORNEY REVIEW` (unusual pattern).
- Name every routed item's **route** from section 2a (for example "PC 1203.4 mandatory route: probation fulfilled") and say whether a declaration is usually needed.
- `offense_type: unknown`, `sentence: unknown`, or `probation_granted: unknown` is `NEEDS ATTORNEY REVIEW` with a fact request naming the missing field.
- A conviction from another state or a federal court is `NOT SCREENED` with the jurisdiction named; California rules are never applied to it.
- Arrests route to `NOT SCREENED` with the PC 851.91 and 851.93 lines. The screen never states a limitations period; it writes "the attorney confirms whether the limitations period has run."

## 2a. Route determinations

The route names below are the only route names a report uses. Gates clear means `current_supervision: none`, `pending_case: no`, `registration_290: no`, and no exclusion found. Any `unsure` or `unknown` on a field a row depends on makes the row `NEEDS ATTORNEY REVIEW`.

| Facts | Route | Band | Declaration usually needed | Source |
|---|---|---|---|---|
| `probation_granted: yes`; `probation_outcome: completed`; gates clear | PC 1203.4 mandatory route: probation fulfilled | LIKELY ELIGIBLE | no | 1203.4(a)(1), "fulfilled the conditions of probation for the entire period" |
| `probation_granted: yes`; `probation_outcome: terminated_early`; gates clear | PC 1203.4 mandatory route: early discharge | LIKELY ELIGIBLE | no | 1203.4(a)(1), "discharged prior to the termination of the period of probation" |
| `probation_granted: yes`; `probation_outcome: completed_with_violation` | PC 1203.4 discretionary route: interest of justice | NEEDS ATTORNEY REVIEW | yes | 1203.4(a)(1), "in its discretion and the interest of justice" |
| `probation_granted: yes`; `probation_outcome: ongoing` | none yet; early termination under PC 1203.3 is the referral | NOT ELIGIBLE NOW, recheck when probation ends | n/a | 1203.4(a)(1), "on probation for an offense" |
| `probation_outcome: revoked`; `offense_type: felony`; `revocation_custody: split_mandatory_supervision` | PC 1203.41 route: split sentence, one year after completion | LIKELY ELIGIBLE if the year has run, else NOT ELIGIBLE NOW with the recheck date | yes | 1203.41(a)(2), one year after a 1170(h)(5)(B) sentence `[practitioner reading, verify]` |
| `probation_outcome: revoked`; `offense_type: felony`; `revocation_custody: jail` | PC 1203.41 route: straight county jail, two years after completion | same pattern | yes | 1203.41(a)(2) |
| `probation_outcome: revoked`; `offense_type: felony`; `revocation_custody: prison` | PC 1203.41 route: state prison, two years after completion; add the PC 1203.42 line when sentenced before October 1, 2011 | same pattern, plus the (a)(6) registration check | yes | 1203.41(a)(2), (a)(6) |
| `probation_outcome: revoked`; `offense_type` misdemeanor or infraction | PC 1203.4 discretionary route and PC 1203.4a both possible | NEEDS ATTORNEY REVIEW | yes | 1203.4a(a) requires "not granted probation"; treatment of revoked probation is an attorney call `[model knowledge — verify]` |
| `probation_granted: no`; `offense_type` misdemeanor, infraction or felony_reduced_to_misdemeanor; `judgment_date` at least 12 months ago; `new_conviction_since: no`; gates clear; sentence complied with | PC 1203.4a mandatory route | LIKELY ELIGIBLE | no | 1203.4a(a): "lapse of one year from the date of pronouncement of judgment," "fully complied with and performed the sentence," "lived an honest and upright life" |
| as above, `judgment_date` less than 12 months ago | PC 1203.4a, waiting period | NOT ELIGIBLE NOW, recheck at judgment plus 12 months | n/a | 1203.4a(a) |
| as above, `new_conviction_since: yes` | PC 1203.4a discretionary route | NEEDS ATTORNEY REVIEW | yes | 1203.4a(b), "in its discretion and in the interest of justice" |
| `probation_granted: no`; `offense_type: felony`; `sentence: split_mandatory_supervision` | PC 1203.41 route: split sentence, one year after completion | LIKELY ELIGIBLE if run, else NOT ELIGIBLE NOW | yes | 1203.41(a)(2) |
| `probation_granted: no`; `offense_type: felony`; `sentence: jail` | PC 1203.41 route: straight county jail, two years after completion | same | yes | 1203.41(a)(2) |
| `probation_granted: no`; `offense_type: felony`; `sentence: prison` | PC 1203.41 route: state prison, two years after completion; add PC 1203.42 when sentenced before October 1, 2011 | same, plus (a)(6) | yes | 1203.41(a)(2), (a)(6) |
| `probation_granted: no`; `offense_type: felony`; `sentence: fine_only` | unusual pattern | NEEDS ATTORNEY REVIEW | n/a | none of the three sections fits cleanly |
| `status.fire_camp: yes` | PC 1203.4b line added whatever the band | referral | n/a | 1203.4b(b)(4): no need to complete supervision |

**Declaration usually needed** mirrors the discretionary routes: when the court "may" rather than "shall," a declaration describing the person's circumstances is the norm `[practice note]`. The exclusion checks that can downgrade a LIKELY ELIGIBLE row are in sections 3, 3a and 4; when `code_section` is blank, say which checks could not run.

## 3. PC 1203.4 items

Statutory basis (PC 1203.4(a)(1)): relief is available "when a defendant has fulfilled the conditions of probation for the entire period of probation, or has been discharged prior to the termination of the period of probation, or in any other case in which a court, in its discretion and the interest of justice, determines that a defendant should be granted the relief," and only "if they are not then serving a sentence for an offense, on probation for an offense, or charged with the commission of an offense."

| `probation_outcome` | Band | Why |
|---|---|---|
| `completed`, client gates clear | `LIKELY ELIGIBLE`; route: PC 1203.4 mandatory route, probation fulfilled; declaration usually needed: no | first clause of (a)(1) |
| `terminated_early`, client gates clear | `LIKELY ELIGIBLE`; route: PC 1203.4 mandatory route, early discharge; declaration usually needed: no | second clause of (a)(1) |
| `completed_with_violation` | `NEEDS ATTORNEY REVIEW`; route: PC 1203.4 discretionary route, interest of justice; declaration usually needed: yes | third clause of (a)(1); "for the entire period" is not met |
| `revoked` | routed by section 2: felony to PC 1203.41 with `revocation_custody`; misdemeanor or infraction to `NEEDS ATTORNEY REVIEW` | |
| `ongoing` | `NOT ELIGIBLE NOW`; recheck "when probation ends" | "on probation for an offense" |
| `unknown` or `not_applicable` | `NEEDS ATTORNEY REVIEW` with a fact request | |

**Exclusion check.** If `code_section` is one of the sections PC 1203.4(b) names, downgrade the item to `NEEDS ATTORNEY REVIEW` with the note "may be excluded by PC 1203.4(b)". The statute text: "Subdivision (a) of this section does not apply to a misdemeanor that is within the provisions of Section 42002.1 of the Vehicle Code, to a violation of subdivision (c) of Section 286, Section 288, subdivision (c) of Section 287 or of former Section 288a, Section 288.5, subdivision (j) of Section 289, Section 311.1, 311.2, 311.3, or 311.11, or a felony conviction pursuant to subdivision (d) of Section 261.5, or to an infraction." When `code_section` is blank, say the exclusion check could not be run and list it under facts to confirm.

**Vehicle Code check.** PC 1203.4(c)(1) provides that subdivision (a) "does not apply to a person who receives a notice to appear or is otherwise charged with a violation of an offense described in subdivisions (a) to (e), inclusive, of Section 12810 of the Vehicle Code," and (c)(2) lets the court grant the relief only "in its discretion and in the interest of justice." If `code_section` is any Vehicle Code section, downgrade the item to `NEEDS ATTORNEY REVIEW` with the note "may be an offense described in Vehicle Code 12810(a) to (e); the mandatory route is unavailable under PC 1203.4(c)(1) and relief is discretionary under (c)(2)". When `code_section` is blank, add "was this a Vehicle Code offense?" to the facts to confirm, next to the subdivision (b) question.

**Restitution is never a fact to confirm.** "A petition for relief under this section shall not be denied due to an unfulfilled order of restitution or restitution fine" (PC 1203.4(c)(3)(A)); the same rule appears in PC 1203.41(d). Do not ask whether restitution was paid.

**Next step text for a `LIKELY ELIGIBLE` 1203.4 item:** a petition for dismissal in the court of conviction, which the person may make "in person or by attorney, or by the probation officer authorized in writing" (PC 1203.4(a)(1)); "relief shall not be granted under this section unless the prosecuting attorney has been given 15 days' notice of the petition" (PC 1203.4(d)(1)); an unfulfilled restitution order or fine "shall not be grounds for finding that a defendant did not fulfil the condition of probation" (PC 1203.4(c)(3)(B)).

## 3a. PC 1203.4a items

Statutory basis (PC 1203.4a(a)): a person "convicted of a misdemeanor and not granted probation" or "convicted of an infraction" shall, "at any time after the lapse of one year from the date of pronouncement of judgment," be permitted to withdraw the plea and have the pleading dismissed, if they "have fully complied with and performed the sentence of the court, are not then serving a sentence for an offense and are not under charge of commission of a crime, and have, since the pronouncement of judgment, lived an honest and upright life and have conformed to and obeyed the laws of the land." Subdivision (b) lets the court grant the same relief "in its discretion and in the interest of justice" after one year when not every requirement of (a) is met.

| Situation | Band |
|---|---|
| `judgment_date` at least 12 months ago, `new_conviction_since: no`, gates clear, sentence complied with | `LIKELY ELIGIBLE`; route: exactly "PC 1203.4a mandatory route" (no probation suffix; this section has no probation); declaration usually needed: no |
| `judgment_date` less than 12 months ago | `NOT ELIGIBLE NOW`; recheck date = judgment month plus 12 months, arithmetic shown, `[model calculation — verify]` |
| `new_conviction_since: yes` | `NEEDS ATTORNEY REVIEW`; route: PC 1203.4a discretionary route (subd. (b)); declaration usually needed: yes |
| `judgment_date: unknown` or `new_conviction_since: unknown` | `NEEDS ATTORNEY REVIEW` with a fact request |
| `offense_type: felony_reduced_to_misdemeanor` | treat as a misdemeanor for this route and add `[review]` "confirm the reduction" `[model knowledge — verify]` |

**Exclusion check.** Subdivision (d) excludes "a misdemeanor violation of subdivision (c) of Section 288," "a misdemeanor falling within the provisions of Section 42002.1 of the Vehicle Code," and "an infraction falling within the provisions of Section 42001 of the Vehicle Code." If `code_section` matches one, downgrade to `NEEDS ATTORNEY REVIEW` with "may be excluded by PC 1203.4a(d)." When blank, say the check could not run.

**"Honest and upright life"** is the attorney's judgment. The screen uses `new_conviction_since` as its only proxy and lists the rest under facts to confirm.

**Next step text for a `LIKELY ELIGIBLE` 1203.4a item:** a petition in the court of conviction, which the person may make "in person or by attorney, or by the probation officer authorized in writing" (subd. (c)(1)); for an infraction only, "by written declaration, except upon a showing of compelling need," with "at least 15 days' notice" to the prosecuting attorney (subd. (f)); the section states no notice period for a misdemeanor petition, so say "notice per local practice" rather than citing one; an unpaid restitution order is not a ground for denial (subd. (e)).

## 4. PC 1203.41 items

Statutory basis (PC 1203.41(a)): "If a defendant is convicted of a felony, the court, in its discretion and in the interest of justice, may order" dismissal, subject to:

- (a)(2) timing: "only after the lapse of one year following the defendant's completion of the sentence, if the sentence was imposed pursuant to subparagraph (B) of paragraph (5) of subdivision (h) of Section 1170, or after the lapse of two years following the defendant's completion of the sentence, if the sentence was imposed pursuant to subparagraph (A) of paragraph (5) of subdivision (h) of Section 1170 or if the defendant was sentenced to the state prison."
- (a)(3) status: "only if the defendant is not on parole or under supervision pursuant to subparagraph (B) of paragraph (5) of subdivision (h) of Section 1170, and is not serving a sentence for, on probation for, or charged with the commission of, an offense."
- (a)(6) registration: a state prison felony qualifies only if it "did not result in a requirement to register as a sex offender."

| Record fact | Waiting period |
|---|---|
| `sentence: split_mandatory_supervision` | one year after completion of the sentence (PC 1170(h)(5)(B)) |
| `sentence: jail` | two years after completion of the sentence (PC 1170(h)(5)(A)) |
| `sentence: prison` | two years after completion of the sentence |
| `probation_outcome: revoked` | read `revocation_custody` as the sentence type and apply the matching row |

**Completion date.** Use `sentence_completed` when the record has it. When it is unknown but `sentencing_date` and `sentence_term` are known, estimate completion as sentencing month plus term, show the arithmetic and tag it `[estimated from the term — verify against the record]`. A jail estimate ignores custody credits, so the real end is usually earlier; a prison estimate ignores parole, which follows release and is commonly counted as part of the sentence (see PC 3000(a)(1) on the card `[model knowledge — verify]`), so the estimate is a floor and the item carries `[review]` whenever its band depends on it. When an estimate and a stated date disagree, treat them as two candidate dates under the divergence rule below.

The PC 1203.4(b) and (c) exclusion checks belong to PC 1203.4 items only. A 1203.41 item has one exclusion, registration on a state prison felony (subd. (a)(6)); when `code_section` is blank on a 1203.41 item, do not list a (b) or (c) check as missing.

PC 1203.41(a)(2) cross-references PC 1170(h)(5)(A) and (B). Since 2015 the text of (h)(5)(A) describes the court's duty to suspend a concluding portion of the term rather than a straight jail term; the one-year and two-year split above follows the common practitioner reading of the cross-reference `[model knowledge — verify]`.

| Situation | Band |
|---|---|
| `sentence_completed` known, period elapsed, gates clear, `registration_290: no` | `LIKELY ELIGIBLE`; route: PC 1203.41 route, split sentence / straight county jail / state prison; declaration usually needed: yes (discretionary) |
| `sentence: prison` (or `revocation_custody: prison`) with `conviction_year` before 2011, or 2011 with sentencing before October 1, 2011 | keep the 1203.41 row and add a `NOT SCREENED` line for PC 1203.42 beside it; PC 1170(h)(7) applies realignment sentencing "to any person sentenced on or after October 1, 2011" `[verify at the leginfo link]` |
| `sentence_completed` known, period not elapsed | `NOT ELIGIBLE NOW`; recheck date = completion month plus the period; show the arithmetic; tag `[model calculation — verify]` |
| completion month could be either release from custody or discharge from supervision (two dates in `notes`) | compute both recheck dates, show both, add `[review]` "which date is completion of the sentence"; if the two readings give different bands (one period has run, the other has not), the item is `NEEDS ATTORNEY REVIEW`; if both give the same band, keep it |
| `sentence_completed: unknown` | `NEEDS ATTORNEY REVIEW` with a fact request |
| `sentence: unknown` | `NEEDS ATTORNEY REVIEW` with a fact request |
| `status.supervision_ended_within_3_years: mandatory_supervision` or an intake note calling it "community supervision" | compute the band under each reading (one year as mandatory supervision; two years as PRCS after a prison term). If both readings give the same band, keep it and put both recheck dates in the item's flags with `[review]` "mandatory supervision (one year) or PRCS after prison (two years)?". If the readings diverge, `NEEDS ATTORNEY REVIEW` with both dates shown |

## 5. Automatic relief flag

PC 1203.425 directs the Department of Justice, "commencing October 1, 2024, and subject to an appropriation in the annual Budget Act, on a monthly basis," to review the state databases and grant relief "without requiring a petition or motion" to people who meet all of its conditions (subds. (a)(1)(A), (a)(2)(A)). The record then carries "a note stating 'relief granted,' listing the date" (subd. (a)(2)(B)). The plugin never sees DOJ records, so this is a flag, not a band.

Add `AUTOMATIC RELIEF MAY APPLY` to an item when the record shows all of the following `[statute / regulator site]`:

- `registration_290: no` (subd. (a)(1)(B)(i): not required to register under the Sex Offender Registration Act);
- `current_supervision: none` (subd. (a)(1)(B)(ii): no active local, state or federal supervision record);
- `pending_case: no` (subd. (a)(1)(B)(iii): not serving a sentence and no indication of pending charges);
- and one of these conviction patterns (subd. (a)(1)(B)(iv)):
  - a probation case with `probation_outcome: completed` or `terminated_early` (sub-subclause (I)(ia): "appears to have completed their term of probation without revocation");
  - a misdemeanor or infraction without a completed probation term, with the sentence completed and "at least one calendar year" since the date of judgment (sub-subclause (I)(ib));
  - a felony with all terms of "incarceration, probation, mandatory supervision, postrelease community supervision, and parole" completed and "a period of four years" since completion with no new felony conviction, unless the felony is a serious felony under PC 1192.7(c), a violent felony under PC 667.5, or one requiring registration (subclause (II)).

When the flag does not apply, say which condition fails and when it would be met (for example "felony: four years from 2025-01 runs to 2029-01"); never say the section does not cover felonies, because subclause (II) does.

Text for the full report (the compact report uses one clause: check the RAP sheet for a "relief granted" note before filing, with the pinpoint): "The Department of Justice reviews records monthly and grants this relief without a petition when its records show eligibility; the RAP sheet then carries a 'relief granted' note next to the entry (PC 1203.425(a)(2)(B)). Check the RAP sheet before filing a petition for this item. A prosecutor or probation department may petition to block automatic relief up to 90 days before the eligibility date (PC 1203.425(b)(1)), and the program depends on an annual appropriation, so a missing note does not mean the person is ineligible."

## 6. Packet roll-up

1. If `pending_case` or `current_supervision` is `unsure`, or any row in section 1 produced a `NEEDS ATTORNEY REVIEW` effect: packet `NEEDS ATTORNEY REVIEW`. Every item band is shown but marked provisional. An unknown trafficking or DV answer never triggers this rule.
2. Else if a client-level disqualifier applies (`pending_case: yes`, `current_supervision: probation`): packet `NOT ELIGIBLE NOW` with the recheck trigger.
3. Else: packet summary in exactly this form, "LIKELY ELIGIBLE n of m, NOT ELIGIBLE NOW k, NEEDS ATTORNEY REVIEW j", with n, k and j counted over the case entries. A per-item `NOT ELIGIBLE NOW` never becomes the packet band; only a client-level disqualifier does. Example: clean gates, one case `LIKELY ELIGIBLE` and one case `NOT ELIGIBLE NOW` give the packet "LIKELY ELIGIBLE 1 of 2, NOT ELIGIBLE NOW 1, NEEDS ATTORNEY REVIEW 0".
4. If `cases` is empty: packet "client-level pre-screen only: [no disqualifier found | disqualifier: X]" plus a fact request for the six case fields (county, conviction year, offense type, sentence type and term, probation granted and how it ended, sentencing month), in the report template's no-case form. No per-case band is given.

Under-flagging is the one-way door. When two rules point to different bands and the record does not settle it, use `NEEDS ATTORNEY REVIEW` and say why.

## 7. Attorney-review triggers at a glance

Any of these on its own sends the item or the packet to `NEEDS ATTORNEY REVIEW`:

- an `unsure` or `unknown` on a field a rule depends on;
- revoked probation (discretionary clause only);
- a possible PC 1203.4(b) exclusion;
- a Vehicle Code offense, or a blank code section where the offense could be one (PC 1203.4(c));
- probation completed with a violation or a reinstated revocation (discretionary route only);
- revoked probation on a misdemeanor or infraction;
- a new conviction since judgment on the PC 1203.4a route, or a PC 1203.4a(d) exclusion;
- any registration answer other than `no`;
- a prison or jail item with an unknown completion date;
- "community supervision" that could be mandatory supervision or PRCS;
- a current parole, mandatory supervision or PRCS term on a 1203.4 item;
- an out-of-state or federal conviction (routes to `NOT SCREENED`, and the packet notes it);
- a trafficking or DV flag.
