# Expected outcomes for the sample intakes

These are the acceptance tests for `/record-clearance-legal:eligibility-screen`. A test driver runs each fixture through the skill without reading this file, then compares. "Must mention" items may be paraphrased; band strings must match exactly. Today is taken as 2026-10-04.

## Every fixture

- Before the screen, the verification checklist appears: numbered yes-or-no lines for the facts the screen will rely on, ending with the yes-or-correction prompt; the screen follows a yes. The checklist is skipped only when the user asked to ("just screen", `--no-confirm`), which the eval prompts do.
- First line of the report is the AI-assisted draft label from the practice profile.
- The body opens with two lines: `**Check 1, intake triage:**` with the outcome from `triage-rules.md` and its reasons, then `**Check 2, eligibility screen:**` with the band line. When check 1 is NOT ELIGIBLE NOW or NOT ELIGIBLE FOR THE PROGRAM, check 2 runs only on request.
- The reviewer note is one line when the run is green; otherwise only the bullets that say something. It names the cards' last-confirmed date and whether a research connector was used.
- No name, email, phone number, address or date of birth appears anywhere in the output. Initials or a clinic ID are fine.
- No fact appears that is not in the fixture or the profile.
- The report ends with the decision tree: one line with three options in the compact form; the five-option form only with `--full`.
- Recheck dates, when given, show the arithmetic and carry `[model calculation — verify]`.
- One source tag per cite. No "packet band", "roll-up" or version number in the report.
- Length: a clean one-case compact report is under 380 words including the filing bullet; the no-case reply is under 220 (the header and reviewer note count); each extra case, flag or review question may add about 80 words. `--full` has no cap.
- Every item on a petition route (LIKELY ELIGIBLE, or NEEDS ATTORNEY REVIEW on a discretionary route) gets one "Filing in [county]" bullet under Next steps, from `skills/eligibility-screen/references/county-filing-guide.md`, tagged `[county filing guide, imported 2026-10-04 — verify with the court]`. A county not in the table gets the statewide default (CR-180, CR-181, proof of service per local practice, serve the prosecutor).

## Fixture 01

| Item | Expected band | Must mention | Must flag |
|---|---|---|---|
| Overall | LIKELY ELIGIBLE 1 of 1, NOT ELIGIBLE NOW 0, NEEDS ATTORNEY REVIEW 0 | client-level gates clear | none beyond standard verification prompts |
| C1 | LIKELY ELIGIBLE | route "PC 1203.4 mandatory route: probation fulfilled", PC 1203.4(a)(1): probation completed, not now serving, on probation or charged; declaration usually needed: no; in Next steps: 15 days' notice to the prosecutor before relief (PC 1203.4(d)(1)) and restitution not a bar; a filing bullet naming CR-180 and CR-181 with the statewide default because Sample County is not in the county table | AUTOMATIC RELIEF MAY APPLY: check the RAP sheet for a "relief granted" note (PC 1203.425); restitution must not appear as a fact to confirm |

## Fixture 02

| Item | Expected band | Must mention | Must flag |
|---|---|---|---|
| Overall | LIKELY ELIGIBLE 1 of 2, NOT ELIGIBLE NOW 0, NEEDS ATTORNEY REVIEW 1 | parole ended, no current supervision, no pending case; the count format is used because no client-level disqualifier applies | none at the overall level |
| C1 | LIKELY ELIGIBLE | PC 1203.4(a)(1) | AUTOMATIC RELIEF MAY APPLY on C1 only |
| C2 | NEEDS ATTORNEY REVIEW (the two readings diverge: 2024-02 plus 24 months = 2026-02 has passed, 2025-08 plus 24 months = 2027-08 has not) | PC 1203.41(a)(2) two-year clause for a state prison sentence; PC 1203.41(a)(6) registration check satisfied by `registration_290: no`; both recheck dates shown with the arithmetic | `[review]` asking which date counts as completion of the sentence; `[model calculation — verify]` on both dates |
| A1 | NOT SCREENED | PC 851.91 sealing of an arrest that did not result in a conviction; the attorney confirms whether the limitations period has run; referral | no number of years for the limitations period |

## Fixture 03

| Item | Expected band | Must mention | Must flag |
|---|---|---|---|
| Overall | NEEDS ATTORNEY REVIEW | `pending_case: unsure` makes every band provisional until resolved | `[review]` on the pending-case answer |
| C1 | NEEDS ATTORNEY REVIEW | route "PC 1203.4 discretionary route: interest of justice"; probation was revoked once and reinstated, so the mandatory clauses do not apply; declaration usually needed: yes | `[review]` discretionary track |
| C2 | NOT ELIGIBLE NOW (both readings give the same band), with both recheck dates: 2026-02 plus 12 months = 2027-02 under mandatory supervision and 2026-02 plus 24 months = 2028-02 under post-release community supervision | PC 1203.41(a)(2): one year after completion for a PC 1170(h)(5)(B) sentence, two years for PC 1170(h)(5)(A) or state prison; "community supervision" could be either | `[review]` on the supervision type; `[model calculation — verify]` on both dates |
| Any | no LIKELY ELIGIBLE band anywhere | | |

## Fixture 04

| Item | Expected band | Must mention | Must flag |
|---|---|---|---|
| Overall | client-level pre-screen only: no disqualifier found at client level | that no cases were given and no case gets a band; the six facts needed (county, conviction year, offense type, sentence type and term, probation granted and how it ended, sentencing month) as one list; the offer to walk through them | none |
| Cases | none | | |
| Length | under 220 words | | |

## Fixture 05

| Item | Expected band | Must mention | Must flag |
|---|---|---|---|
| Overall | LIKELY ELIGIBLE 0 of 1, NOT ELIGIBLE NOW 0, NEEDS ATTORNEY REVIEW 1 | client-level gates clear | none at the overall level |
| C1 | NEEDS ATTORNEY REVIEW | PC 1203.4(c)(1) removes offenses described in Vehicle Code 12810(a) to (e) from the mandatory route and (c)(2) makes relief discretionary; the word "mandatory" must not describe this item | `[review]` on the Vehicle Code question; AUTOMATIC RELIEF MAY APPLY may still be raised (PC 1203.425(a)(1)(B)(iv)(I)(ia) has no Vehicle Code carve-out in the fetched text) |

## Fixture 06

| Item | Expected band | Must mention | Must flag |
|---|---|---|---|
| Overall | LIKELY ELIGIBLE 1 of 1, NOT ELIGIBLE NOW 0, NEEDS ATTORNEY REVIEW 0 | client-level gates clear | none at the overall level |
| C1 | LIKELY ELIGIBLE | route "PC 1203.4a mandatory route"; PC 1203.4a(a): one year from the date of pronouncement of judgment has run (2023-04), sentence complied with, no new conviction since; declaration usually needed: no; the subdivision (d) exclusions do not reach PC 415; notice per local practice, petition in the court of conviction | AUTOMATIC RELIEF MAY APPLY (PC 1203.425(a)(1)(B)(iv)(I)(ib): misdemeanor, sentence completed, more than one calendar year since judgment) |

## Fixture 07 (raw paste)

The fixture's text block is pasted into the screen with no import step.

| Item | Expected | Must mention | Must not |
|---|---|---|---|
| Identifiers | one line "Dropped at import: name, date of birth, address" (order free); `person.reference` is `P.E.` | | the name, the date of birth or the street address anywhere, including the echo and any summary |
| Questions | at most one question, then the verification checklist, then the screen after a yes | | a request for a RAP sheet or any document |
| Checklist | the four gates, the one case in two lines, the program gates; no consents | | |
| Overall | LIKELY ELIGIBLE 1 of 1, NOT ELIGIBLE NOW 0, NEEDS ATTORNEY REVIEW 0 | gates clear; program gates met | |
| C1 | LIKELY ELIGIBLE | route "PC 1203.4 mandatory route: probation fulfilled"; PC 484 is not a PC 1203.4(b) section and not a Vehicle Code offense; 15 days' notice; restitution not a bar | |

## Fixture 08 (filing guidance, Mono County)

The user types the prose in `08-filing-guidance.md` with no record.

| Item | Expected | Must mention | Must not |
|---|---|---|---|
| Overall | LIKELY ELIGIBLE 1 of 1, NOT ELIGIBLE NOW 0, NEEDS ATTORNEY REVIEW 0 | gates clear | |
| C1 | LIKELY ELIGIBLE | route "PC 1203.4 mandatory route: probation fulfilled"; declaration usually needed: no | |
| Filing bullet | "Filing in Mono County" | CR-180 petition, CR-181 order, CR-106 proof of service by mail; serve the DA by mail; mail or in-person filing; the court sets a hearing (about a month out); the tag `[county filing guide, imported 2026-10-04 — verify with the court]` | MC-031 listed as required (the route is mandatory, so no declaration is usually needed); any Proposition 47, Proposition 64 or fire camp form; a DA address or email invented beyond the guide |
| Questions | none; the verification checklist, then the screen after a yes | | |

## Fixture 09 (RAP sheet excerpt)

The text block is pasted into the screen.

| Item | Expected | Must mention | Must not |
|---|---|---|---|
| Input | accepted as case facts; no refusal | the attorney still reads the full RAP sheet | a request for a document; "copied from a RAP sheet" as a reason to refuse |
| Mapping | San Diego County (Central courthouse); misdemeanor; HS 11377(a); probation only, term 2 years; sentenced 2023-05; completed | the estimate 2023-05 + 24 months = 2025-05 `[estimated from the term — verify against the record]` | an invented end month stated as fact |
| Check 1 | PROCEED TO SCREEN (gates clear from the prose; consents not asked, noted as a follow-up, not a stop, because the input is prose rather than a form) | | |
| C1 | LIKELY ELIGIBLE | route "PC 1203.4 mandatory route: probation fulfilled"; HS 11377 is not a PC 1203.4(b) section and not a Vehicle Code offense; declaration usually needed: no; AUTOMATIC RELIEF MAY APPLY | |
| Filing | "Filing in San Diego County": CR-180 petition with the San Diego CRM-205 work-up sheet, CR-181 order, CR-106 proof of service by mail; serve probation and the DA by mail; original + 3; remote appearance permitted; the North County form CRM-204 does not apply to Central | | any Proposition 47 form (not screened; HS 11377 may be noted as a Proposition 47 section `[model knowledge — verify]`) |

## Fixture 10 (triage, consents incomplete)

Run through `/record-clearance-legal:intake-import`.

| Item | Expected |
|---|---|
| Dropped | name |
| Check 1 | NOT ELIGIBLE FOR THE PROGRAM; reason "consents incomplete: record sharing, data use"; the profile's "When a gate fails" step or "send the consent form again"; check 2 offered on request only |
| Must not | any band for a conviction; "eligible" or "ineligible" standing alone |

## Fixture 11 (triage, not sure on the pending case)

Run through `/record-clearance-legal:intake-import`.

| Item | Expected |
|---|---|
| Check 1 | NEEDS ATTORNEY REVIEW; reason "pending case answered not sure, no prior clearance"; everything else clear; RAP sheet not on hand, request it; hand-off to the screen with every band provisional |
| Variant | with `program_gates.prior_review_decision: cleared`, PROCEED TO SCREEN |
| Language | `preferred_language: es` recorded; no effect on triage |

## Fixture 12 (triage, parole ended under two years ago)

Run through `/record-clearance-legal:intake-import`.

| Item | Expected |
|---|---|
| Check 1 | NOT ELIGIBLE NOW; released from parole 2025-03, under two years, no other convictions; recheck 2025-03 + 24 months = 2027-03 `[model calculation — verify]` (PC 1203.41(a)(2)); check 2 on request only |
| Must not | a band for a conviction; treating the bucket alone as a date (the month is given, so the date is computed) |

## Interactive scenario A

No record is given. The user says: "Walk me through one. Felony, probation was granted but it got revoked and he did a split sentence in county jail with mandatory supervision; everything ended January 2025. Nothing pending, not on anything now, no registration, no fire camp. I don't remember the section." The skill must ask only what the interview needs that the prose did not settle, build the record, echo the minimal record, and then screen.

| Item | Expected band | Must mention | Must flag |
|---|---|---|---|
| Overall | LIKELY ELIGIBLE 1 of 1, NOT ELIGIBLE NOW 0, NEEDS ATTORNEY REVIEW 0 | gates clear from the client-level answers | none at the overall level |
| C1 | LIKELY ELIGIBLE | route "PC 1203.41 route: split sentence, one year after completion"; 2025-01 plus 12 months = 2026-01 has run as of 2026-10-04 `[model calculation — verify]`; discretionary, declaration usually needed: yes; 15 days' notice (PC 1203.41(e)) | the one-year period is flagged as the practitioner reading of the 1170(h)(5)(A)/(B) cross-reference (a `[model knowledge — verify]` tag, a `[review]`, or the reviewer note's Before relying line); no PC 1203.4(b) or (c) check is listed as missing, because PC 1203.41 has none |
| Process | | the skill never asked for a RAP sheet or document; it asked one turn (county and year) with at most two questions; then it showed the verification checklist, waited for yes, and screened; YAML only on request | |

## Quick start test

Running `/record-clearance-legal:cold-start-interview`, choosing quick, and answering as the supervising attorney must ask no more than one question about the ethical preconditions (a yes or no), must not ask for the four typed items, and must end with a confirmation that lists role and attorney, state, review model, intake source, and the sections carrying defaults, with no `[PLACEHOLDER` marker left in the written profile.

## Hard-stop test

With the same profile but `State: NV` in `## Jurisdiction`, running fixture 01 must stop before any screening, say that no relief card set exists for NV, point at `skills/eligibility-screen/references/relief/_template.md`, and must not apply California rules.
