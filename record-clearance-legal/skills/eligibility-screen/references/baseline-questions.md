# Baseline questions

Purpose: let a supervising attorney or staff member screen a conviction from memory or the court file, without a RAP sheet or a transcription. Four to eight questions per conviction populate a case entry in the normalized record; the route, band and declaration line then come from the rulebook, `screening-bands.md` section 2a. The Access Project's engine formulas were used to choose which questions matter and to name the routes; no code-section list or engine-only heuristic was carried over (`engine-crosswalk.md` has the label map and the list of what was left out).

Sources: PC 1203.4, 1203.4a and 1203.41 text as recorded on the relief cards `[statute / regulator site]`, last confirmed 2026-10-04; PC 1170(h) text via the public.law mirror `[verify at the leginfo link]`.

## How to run the interview

- Map everything the person already said onto the record fields first. Then ask only the questions whose fields are still `unknown` or `unsure` and can change the route, in the order below, at most two per turn. Offer the listed options; accept free text and map it.
- Stop asking as soon as the route is settled. A completed-probation misdemeanor needs four answers, not eight.
- "Not sure" is recorded as `unsure` or `unknown` and becomes a review flag. Never resolve it by guessing.
- Never ask for a RAP sheet or a document. If the person offers one, say the screen does not read documents and continue with the questions.
- Once the route is settled, show the verification checklist (numbered yes-or-no lines for the facts the screen will rely on), wait for yes or a correction, then screen. "record" shows the YAML.

## Worked example: prose first, then only the gaps

The attorney says: "Felony, probation was granted but it got revoked and he did a split sentence in county jail with mandatory supervision; everything ended January 2025. Nothing pending, not on anything now, no registration, no fire camp. I don't remember the section."

Already answered: C1 none, C2 no, C3 no, C4 no, Q1 felony, Q2 yes, Q3 revoked, Q4 split sentence with mandatory supervision, Q7 2025-01, Q9 not known. Still unknown and route-relevant: county and conviction year (needed for the record and the pre-realignment check), nothing else. So the skill asks one turn ("Which county, and what year was the conviction?"), then shows the verification checklist, waits for yes, and screens. Zero re-asked questions.

## Once per client

| # | Question | Options | Record field |
|---|---|---|---|
| C1 | Right now, is the person on probation, parole, mandatory supervision or post-release community supervision? | no / probation / parole / mandatory supervision / PRCS / not sure | `status.current_supervision` ("no" is recorded as `none`) |
| C2 | Any open or pending case or charge? | no / yes / not sure | `status.pending_case` |
| C3 | Required to register under PC 290? | no / currently / terminated / not sure | `status.registration_290` |
| C4 | Fire camp, county hand crew or institutional firehouse during any of these sentences? | no / yes / not sure | `status.fire_camp` |

## Per conviction

| # | Ask when | Question | Options | Record field |
|---|---|---|---|---|
| Q1 | always | What offense type was the conviction? | infraction / misdemeanor / felony / felony later reduced to a misdemeanor / not sure | `offense_type` |
| Q2 | always | Was probation granted? | yes / no / not sure | `probation_granted` |
| Q3 | Q2 yes | How did probation end? | completed with no violations / ended early by the court / completed, but with a violation or a revocation that was reinstated / revoked and sentenced to custody / still on probation / not sure | `probation_outcome` (`completed`, `terminated_early`, `completed_with_violation`, `revoked`, `ongoing`, `unknown`) |
| Q4 | Q3 revoked, or Q2 no on a felony | What custody was imposed? | county jail / county jail split with mandatory supervision / state prison / fine only, no custody / not sure | `sentence` (or `revocation_custody` when Q3 was revoked) |
| Q4b | Q2 yes, or any custody route | How long was the probation term or the sentence? | years or months / not sure | `sentence_term` |
| Q5 | Q2 no on a misdemeanor or infraction | When was judgment pronounced? | month and year / not sure | `judgment_date` |
| Q6 | Q2 no on a misdemeanor or infraction | Any new conviction since that judgment? | no / yes / not sure | `new_conviction_since` |
| Q7 | any custody route | When was the person sentenced (month and year; the conviction date if that is all you know)? If you know when the sentence actually ended, counting parole or supervision, give that instead. | month and year / two dates if unsure which counts / not sure | `sentencing_date`; `sentence_completed` when the actual end is known, with a second candidate date in `notes`; otherwise the screen estimates completion as sentencing date plus term |
| Q8 | state prison | Was the person sentenced before October 1, 2011? | no / yes / not sure | derived from `conviction_year`; ask only when the year is 2011 or unknown |
| Q9 | always, last | Code section, if known | text / not known | `code_section` |

County and conviction year are recorded with every case; ask for them when the prose did not give them.

## Determinations

The route, band and declaration line for every answer pattern are in `screening-bands.md` section 2a; do not keep a second copy here. Gates clear means `current_supervision: none`, `pending_case: no`, `registration_290: no`, and no exclusion found. "Felony later reduced to a misdemeanor" is treated as a misdemeanor for the PC 1203.4a route `[model knowledge — verify: PC 17(b) text was fetched only in summary]`.
