# Expected outcomes for the sample intakes

These are the acceptance tests for `/record-clearance-legal:eligibility-screen`. A test driver runs each fixture through the skill without reading this file, then compares. "Must mention" items may be paraphrased; band strings must match exactly.

## Every fixture

- First line of the report is the AI-assisted draft label from the practice profile.
- A reviewer note block appears before the report body and names the statute cards' last-confirmed date and whether a research connector was used.
- No name, email, phone number, address or date of birth appears anywhere in the output.
- No fact appears that is not in the fixture or the profile.
- The report ends with the next-steps decision tree.
- Recheck dates, when given, show the arithmetic and carry `[model calculation — verify]`.

## Fixture 01

| Item | Expected band | Must mention | Must flag |
|---|---|---|---|
| Packet | LIKELY ELIGIBLE 1 of 1, NOT ELIGIBLE NOW 0, NEEDS ATTORNEY REVIEW 0 | no client-level disqualifier | none beyond standard verification prompts |
| C1 | LIKELY ELIGIBLE | PC 1203.4(a)(1) mandatory track: probation completed, not now serving, on probation or charged; in Next steps by item: 15 days' notice to the prosecutor before relief (PC 1203.4(d)(1)) and restitution not a bar | AUTOMATIC RELIEF MAY APPLY: probation completed without revocation, check the RAP sheet for a "relief granted" note (PC 1203.425); restitution must not appear in facts to confirm |

## Fixture 02

| Item | Expected band | Must mention | Must flag |
|---|---|---|---|
| Packet | LIKELY ELIGIBLE 1 of 2, NOT ELIGIBLE NOW 0, NEEDS ATTORNEY REVIEW 1 | parole ended, no current supervision, no pending case; the count format is used because no client-level disqualifier applies | none at packet level |
| C1 | LIKELY ELIGIBLE | PC 1203.4(a)(1) | AUTOMATIC RELIEF MAY APPLY on C1 only |
| C2 | NEEDS ATTORNEY REVIEW (the two readings diverge: 2024-02 plus 24 months = 2026-02 has already passed, 2025-08 plus 24 months = 2027-08 has not) | PC 1203.41(a)(2) two-year clause for a state prison sentence; PC 1203.41(a)(6) registration check satisfied by `registration_290: no`; both recheck dates shown with the arithmetic | `[review]` asking which date counts as completion of the sentence; `[model calculation — verify]` on both dates |
| A1 | NOT SCREENED | PC 851.91 sealing of an arrest that did not result in a conviction; referral | none |

## Fixture 03

| Item | Expected band | Must mention | Must flag |
|---|---|---|---|
| Packet | NEEDS ATTORNEY REVIEW | `pending_case: unsure` makes every band provisional until resolved | `[review]` on the pending-case answer |
| C1 | NEEDS ATTORNEY REVIEW | route "PC 1203.4 discretionary route: interest of justice"; probation was revoked once and reinstated, so the mandatory clauses do not apply; declaration usually needed: yes | `[review]` discretionary track |
| C2 | NOT ELIGIBLE NOW (both readings give the same band), with both recheck dates in the row: 2026-02 plus 12 months = 2027-02 under mandatory supervision and 2026-02 plus 24 months = 2028-02 under PRCS | PC 1203.41(a)(2): one year after completion for a PC 1170(h)(5)(B) sentence, two years for PC 1170(h)(5)(A) or state prison; "community supervision" could be mandatory supervision or PRCS | `[review]` on the supervision type; `[model calculation — verify]` on both dates |
| Any | no LIKELY ELIGIBLE band anywhere | | |

## Fixture 04

| Item | Expected band | Must mention | Must flag |
|---|---|---|---|
| Packet | client-level pre-screen only: no disqualifier found at client level | that no cases were provided and no per-case band can be given | fact request listing county, conviction year, offense level, sentence type, probation granted and outcome, completion month |
| Cases | none | | |

## Fixture 05

| Item | Expected band | Must mention | Must flag |
|---|---|---|---|
| Packet | LIKELY ELIGIBLE 0 of 1, NOT ELIGIBLE NOW 0, NEEDS ATTORNEY REVIEW 1 | no client-level disqualifier | none at packet level |
| C1 | NEEDS ATTORNEY REVIEW | PC 1203.4(c)(1) removes offenses described in Vehicle Code 12810(a) to (e) from the mandatory route and (c)(2) makes relief discretionary; the word "mandatory" must not describe this item | `[review]` on the Vehicle Code question; AUTOMATIC RELIEF MAY APPLY may still be raised (PC 1203.425(a)(1)(B)(iv)(I)(ia) has no Vehicle Code carve-out in the fetched text) |

## Fixture 06

| Item | Expected band | Must mention | Must flag |
|---|---|---|---|
| Packet | LIKELY ELIGIBLE 1 of 1, NOT ELIGIBLE NOW 0, NEEDS ATTORNEY REVIEW 0 | no client-level disqualifier | none at packet level |
| C1 | LIKELY ELIGIBLE | route "PC 1203.4a mandatory route"; PC 1203.4a(a): one year from the date of pronouncement of judgment has run (2023-04), sentence complied with, no new conviction since; declaration usually needed: no; the subdivision (d) exclusions do not reach PC 415; infraction-style 15-day notice does not apply, but the petition goes to the court of conviction | AUTOMATIC RELIEF MAY APPLY (PC 1203.425(a)(1)(B)(iv)(I)(ib): misdemeanor, sentence completed, more than one calendar year since judgment) |

## Interactive scenario A

No record is given. The user says: "Walk me through one. Felony, probation was granted but it got revoked and he did a split sentence in county jail with mandatory supervision; everything ended January 2025. Nothing pending, not on anything now, no registration, no fire camp. I don't remember the section." The skill must ask only what the baseline interview needs that the prose did not settle, build the record, echo it, and then screen.

| Item | Expected band | Must mention | Must flag |
|---|---|---|---|
| Packet | LIKELY ELIGIBLE 1 of 1, NOT ELIGIBLE NOW 0, NEEDS ATTORNEY REVIEW 0 | gates clear from the client-level answers | none at packet level |
| C1 | LIKELY ELIGIBLE | route "PC 1203.41 route: split sentence, one year after completion"; 2025-01 plus 12 months = 2026-01 has run as of 2026-10-04 `[model calculation — verify]`; discretionary, declaration usually needed: yes; 15 days' notice (PC 1203.41(e)) | `[review]` that the one-year period is the practitioner reading of the 1170(h)(5)(A)/(B) cross-reference; code section not known, so the exclusion checks could not run |
| Process | | the skill never asked for a RAP sheet or document; it asked at most two questions per turn; it echoed the record before screening | |

## Hard-stop test

With the same profile but `State: NV` in `## Jurisdiction`, running fixture 01 must stop before any screening, say that no relief card set exists for NV, point at `skills/eligibility-screen/references/relief/_template.md`, and must not apply California rules.
