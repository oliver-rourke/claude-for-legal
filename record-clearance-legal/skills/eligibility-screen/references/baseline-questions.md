# Baseline questions and route determinations

Purpose: let a supervising attorney or staff member screen a conviction from memory or the court file, without a RAP sheet or a transcription. Five to eight questions per conviction populate a case entry in the normalized record; the determination names the statutory route the facts point to, the band, and whether a declaration is usually needed. Every rule here is statute text or a marked practice note. The Access Project's engine formulas were used to choose which questions matter and to name the routes; no code-section list or engine-only heuristic was carried over (see the last section).

Sources: PC 1203.4, 1203.4a and 1203.41 text as recorded on the relief cards `[statute / regulator site]` `[settled — last confirmed 2026-10-04]`; PC 1170(h) text via the public.law mirror `[verify at the leginfo link]`.

## How to run the interview

- Ask the client-level questions once, then the per-conviction questions in order. At most two questions per turn. Offer the listed options; accept free text and map it.
- Stop asking as soon as the route is settled. A completed-probation misdemeanor needs four answers, not eight.
- "Not sure" is recorded as `unsure` or `unknown` and becomes a review flag. Never resolve it by guessing.
- Never ask for a RAP sheet or a document. If the person offers one, say the screen does not read documents and continue with the questions.
- Echo the record back in the schema shape and get a yes before screening.

## Worked example: prose first, then only the gaps

The attorney says: "Felony, probation was granted but it got revoked and he did a split sentence in county jail with mandatory supervision; everything ended January 2025. Nothing pending, not on anything now, no registration, no fire camp. I don't remember the section."

Already answered: C1 none, C2 no, C3 no, C4 no, Q1 felony, Q2 yes, Q3 revoked, Q4 split sentence with mandatory supervision, Q7 2025-01, Q9 not known. Still unknown and route-relevant: county and conviction year (needed for the record and the pre-realignment check), nothing else. So the skill shows the draft record, asks one turn ("Which county, and what year was the conviction?"), echoes the completed record as YAML, and screens. Zero re-asked questions.

## Once per client

| # | Question | Options | Record field |
|---|---|---|---|
| C1 | Right now, is the person on probation, parole, mandatory supervision or post-release community supervision? | no / probation / parole / mandatory supervision / PRCS / not sure | `status.current_supervision` |
| C2 | Any open or pending case or charge? | no / yes / not sure | `status.pending_case` |
| C3 | Required to register under PC 290? | no / currently / terminated / not sure | `status.registration_290` |
| C4 | Fire camp, county hand crew or institutional firehouse during any of these sentences? | no / yes / not sure | `status.fire_camp` |

## Per conviction

| # | Ask when | Question | Options | Record field |
|---|---|---|---|---|
| Q1 | always | What level was the conviction? | infraction / misdemeanor / felony / felony later reduced to a misdemeanor / not sure | `offense_level` |
| Q2 | always | Was probation granted? | yes / no / not sure | `probation_granted` |
| Q3 | Q2 yes | How did probation end? | completed with no violations / ended early by the court / completed, but with a violation or a revocation that was reinstated / revoked and sentenced to custody / still on probation / not sure | `probation_outcome` (`completed`, `terminated_early`, `completed_with_violation`, `revoked`, `ongoing`, `unknown`) |
| Q4 | Q3 revoked, or Q2 no on a felony | What custody was imposed? | county jail / county jail split with mandatory supervision / state prison / fine only, no custody / not sure | `sentence` (or `revocation_custody` when Q3 was revoked) |
| Q5 | Q2 no on a misdemeanor or infraction | When was judgment pronounced? | month and year / not sure | `judgment_date` |
| Q6 | Q2 no on a misdemeanor or infraction | Any new conviction since that judgment? | no / yes / not sure | `new_conviction_since` |
| Q7 | any custody route | When did the sentence end, counting parole or supervision? | month and year / two dates if unsure which counts / not sure | `sentence_completed`, with the second date in `notes` |
| Q8 | state prison | Was the person sentenced before October 1, 2011? | no / yes / not sure | derived from `conviction_year`; ask only when the year is 2011 or unknown |
| Q9 | always, last | Code section, if known | text / not known | `code_section` |

## Determinations

Gates clear means C1 no, C2 no, C3 no, and no exclusion found. Any `unsure` or `unknown` on a field a row depends on makes the row `NEEDS ATTORNEY REVIEW`.

| Facts | Route | Band | Declaration usually needed | Source |
|---|---|---|---|---|
| Q2 yes; Q3 completed with no violations; gates clear | PC 1203.4 mandatory route: probation fulfilled | LIKELY ELIGIBLE | no | 1203.4(a)(1), "fulfilled the conditions of probation for the entire period" |
| Q2 yes; Q3 ended early | PC 1203.4 mandatory route: early discharge | LIKELY ELIGIBLE | no | 1203.4(a)(1), "discharged prior to the termination of the period of probation" |
| Q2 yes; Q3 completed with a violation or a reinstated revocation | PC 1203.4 discretionary route: interest of justice | NEEDS ATTORNEY REVIEW | yes | 1203.4(a)(1), "in its discretion and the interest of justice" |
| Q2 yes; Q3 still on probation | none yet; early termination under PC 1203.3 is the referral | NOT ELIGIBLE NOW, recheck when probation ends | n/a | 1203.4(a)(1), "on probation for an offense" |
| Q2 yes; Q3 revoked; Q1 felony; Q4 split sentence | PC 1203.41 route: split sentence, one year after completion | LIKELY ELIGIBLE if the year has run, else NOT ELIGIBLE NOW with the recheck date | yes | 1203.41(a)(2), one year after a 1170(h)(5)(B) sentence `[practitioner reading, verify]` |
| Q2 yes; Q3 revoked; Q1 felony; Q4 county jail | PC 1203.41 route: straight county jail, two years after completion | same pattern | yes | 1203.41(a)(2) |
| Q2 yes; Q3 revoked; Q1 felony; Q4 state prison | PC 1203.41 route: state prison, two years after completion; add the PC 1203.42 line when Q8 is yes | same pattern, plus the (a)(6) registration check | yes | 1203.41(a)(2), (a)(6) |
| Q2 yes; Q3 revoked; Q1 misdemeanor | PC 1203.4 discretionary route and PC 1203.4a both possible | NEEDS ATTORNEY REVIEW | yes | 1203.4a(a) requires "not granted probation"; treatment of revoked probation is an attorney call `[model knowledge — verify]` |
| Q2 no; Q1 misdemeanor, infraction or felony later reduced; Q5 at least one year ago; Q6 no; gates clear; sentence complied with | PC 1203.4a mandatory route | LIKELY ELIGIBLE | no | 1203.4a(a): "lapse of one year from the date of pronouncement of judgment," "fully complied with and performed the sentence," "lived an honest and upright life" |
| Q2 no; Q1 as above; Q5 less than one year ago | PC 1203.4a, waiting period | NOT ELIGIBLE NOW, recheck at judgment plus 12 months | n/a | 1203.4a(a) |
| Q2 no; Q1 as above; Q6 yes | PC 1203.4a discretionary route | NEEDS ATTORNEY REVIEW | yes | 1203.4a(b), "in its discretion and in the interest of justice" |
| Q2 no; Q1 felony; Q4 split sentence | PC 1203.41 route: split sentence, one year | LIKELY ELIGIBLE if run, else NOT ELIGIBLE NOW | yes | 1203.41(a)(2) |
| Q2 no; Q1 felony; Q4 county jail | PC 1203.41 route: straight county jail, two years | same | yes | 1203.41(a)(2) |
| Q2 no; Q1 felony; Q4 state prison | PC 1203.41 route: state prison, two years; add PC 1203.42 when Q8 yes | same, plus (a)(6) | yes | 1203.41(a)(2), (a)(6) |
| Q2 no; Q1 felony; Q4 fine only | unusual pattern | NEEDS ATTORNEY REVIEW | n/a | none of the three sections fits cleanly |
| C4 yes on this case | PC 1203.4b line added whatever the band | referral | n/a | 1203.4b(b)(4): no need to complete supervision |

**Exclusion checks that can downgrade a LIKELY ELIGIBLE row to NEEDS ATTORNEY REVIEW:** a PC 1203.4(b) section; any Vehicle Code section on a 1203.4 route (PC 1203.4(c)); on the 1203.4a route, a misdemeanor under PC 288(c), a misdemeanor under Vehicle Code 42002.1, or an infraction under Vehicle Code 42001 (PC 1203.4a(d)); registration on a prison route (PC 1203.41(a)(6)). When Q9 is "not known," say which checks could not run.

**"Felony later reduced to a misdemeanor"** is treated as a misdemeanor for the 1203.4a route `[model knowledge — verify: PC 17(b) text was fetched only in summary]`.

**Declaration usually needed** mirrors the discretionary routes: when the court "may" rather than "shall," a declaration describing the person's circumstances is the norm `[practice note]`.

## Crosswalk to The Access Project's engine labels

Optional. Programs that use the Clean Slate Engine will recognize these labels; everyone else can ignore the column.

| Route | Engine label |
|---|---|
| PC 1203.4 mandatory, probation fulfilled | 2a |
| PC 1203.4 mandatory, early discharge | 2b |
| PC 1203.4 discretionary | 2c |
| PC 1203.4a mandatory | 3a |
| PC 1203.4a discretionary | 3b |
| PC 1203.41 split sentence | 5a |
| PC 1203.41 straight county jail | 5b |
| PC 1203.41 state prison | 5c |
| PC 1203.42 pre-realignment prison | 6 |

## What was not carried over from the engine formulas, and why

- **Code-section lists** (always discretionary, never eligible, realignment-with-prison, wobbler, Prop 47, Prop 64). The statute's own exclusion lists are used instead; everything else is "full analysis decides."
- **Prop 47 and Prop 64 pre-emption of 1203.41.** Program policy, not statute; the report asks full analysis to decide.
- **Registration-date heuristics.** Replaced by the client-level registration question and the (a)(6) flag.
- **Engine date cutoffs.** Replaced by the statutory October 1, 2011 realignment date `[PC 1170(h)(7), verify at the leginfo link]` and the one-year and two-year periods in 1203.41(a)(2).
- **"Felony previously reduced" and Prop 47 or 64 eligibility as 1203.4a entry points.** Kept only as the "felony later reduced" option, tagged verify.
