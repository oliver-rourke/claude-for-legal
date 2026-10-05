# Report template

Two forms. The compact form is the default. The full form is for `--full`, and for an attorney who asks for the long version afterwards. Both start with the work-product header and the reviewer note, and both end with the decision tree. Nothing in either form is a statement to an applicant.

## Rules for both forms

- **Reviewer note.** One line when the run is green (no `[review]` flags, no missing fields, currency current, no defaults applied): `> ⚠️ Reviewer note: cards last confirmed [date] · research connector [✓ verified | not connected, cites from the cards, verify before relying] · read [id]: [n] cases, [m] arrests, nothing missing · no flags · ready for your eyes`. Otherwise the bullet form, keeping only the bullets that say something: **Sources** always; **Read** always; **Flagged** when at least one `[review]`; **Currency** only when stale; **Defaults applied** when the profile had placeholders; **Before relying** when there is an action.
- **One tag per cite.** A rule quoted from a card carries `[statute / regulator site]`; the cards' last-confirmed date appears once, in the Sources line. Anything not on a card carries `[model knowledge — verify]`. Computed dates carry `[model calculation — verify]`; judgment calls carry `[review]`.
- **Plain words.** Say "overall", not "packet band" or "roll-up"; name the route with its statute; name supervision by its type and spell out PRCS once as post-release community supervision. No version numbers in a report.
- **Empty sections are omitted.** No "none" rows, no "no cases" tables.
- **Next steps are actions, not analysis.** One bullet per item, one action each, never restating the item block. LIKELY ELIGIBLE: the card's next-step sentence, then the county filing bullet. NOT ELIGIBLE NOW: log the date and screen again then. NEEDS ATTORNEY REVIEW: the question for the attorney in one sentence (plus the filing bullet when the route is discretionary). NOT SCREENED: the referral in one clause.
- **Filing bullet.** For each item on a petition route, one bullet "Filing in [county]:" from `county-filing-guide.md`: the petition and order forms, the proof-of-service form, who is served and how, the declaration form only when a declaration is usually needed, and one local point when the county has one. A county not in the table gets the guide's statewide default and says so. Always tagged `[county filing guide, imported 2026-10-04 — verify with the court]`.
- **Program gates and the review line stay short.** "Program gates met", or the one gate that is not met with the rule applied; the review-model phrase from the profile and nothing about whether a channel is connected.
- **One question** is one sentence with no link.
- **The automatic-relief check lives in the item block**, never as a separate Next-steps bullet.
- **Staff follow-ups are one clause each** ("ask the trafficking or DV question the form skipped"), never a walkthrough of what each answer would change. Links from the profile (referral lists, RAP sheet guidance) appear only in the filing checklist or when the user asks.
- **The title uses `person.reference` and `record_id`**, never a name.

## Verification checklist (before the screen)

Shown after the input is read and before any screen, unless the user asked to skip it. Plain words, one fact per line, numbered so a correction can name its line.

```markdown
Dropped at import: [categories, or none].

**Before I screen, confirm each line (yes, or the number and the correction):**
1. Today: not on probation, parole, mandatory supervision or post-release community supervision.
2. No open or pending case.
3. Not required to register under PC 290.
4. No fire camp or hand crew time.
5. C1: [section] [level], [county], [year].
6. C1: [probation granted and completed with no violations, ended YYYY-MM | custody type and the month the sentence ended | judgment YYYY-MM, no new conviction since].
7. Program gates: [lives in the service area; conviction in the service area | not asked].

Reply yes to confirm all and I will screen, or give the number and the correction. Say record to see the full YAML.
```

## Compact form (default)

Budget: a clean one-case screen fits in about 300 words with its filing bullet; two cases with a review question and an arrest, about 500. Length grows only with more cases, flags and review questions.

```markdown
[AI-ASSISTED DRAFT — requires attorney review before any client communication]

> ⚠️ Reviewer note: [one line when green, otherwise the bullets that say something]

# Screen: [reference] ([record_id]), [date]

**Overall:** [the band line from the rulebook section 6, verbatim]. Client-level gates [clear (no pending case, no supervision, no registration, no fire camp, no trafficking or DV flag) | the gate that fired and its effect]. Program gates [met | not met: gate, rule applied]. [QUEUED for [attorney] at [queue] | CHECK WITH [attorney] BEFORE ACTING | nothing, per the review model]

**[C1], [code section or "section not known"] [level], [county] [year]: [BAND].** [Route name]; [subdivision]; [the two or three facts that drove it]. Declaration usually needed: [yes | no]. [Exclusion checks run and clear | could not run, section blank]. [AUTOMATIC RELIEF MAY APPLY: check the RAP sheet for a "relief granted" note before filing ([PC 1203.425 pinpoint]) | automatic relief does not apply yet: [condition that fails] until [date]]. [Recheck: YYYY-MM + N months = YYYY-MM `[model calculation — verify]`] [`[review]` the question for the attorney] `[statute / regulator site]`

[one block per further case; an arrest is one line: **A1, [year] arrest, [county]: NOT SCREENED.** PC 851.91 petition if the limitations period has run, which the attorney confirms; PC 851.93 relief may already appear on the record. Referral: [target from the profile].]

**Next steps**
- [C1]: [the card's next-step text: petition for dismissal in the court of conviction, in person, by attorney or by the authorized probation officer (pinpoint); 15 days' notice to the prosecuting attorney before relief (pinpoint); unpaid restitution is not a bar (pinpoint)]
- Filing in [county]: [petition form], [order form], [proof-of-service form]; serve [who] by [how]; [declaration form, only when a declaration is usually needed]; [one local point]. `[county filing guide, imported 2026-10-04 — verify with the court]`
- [NOT ELIGIBLE NOW item]: log the recheck date [YYYY-MM] and screen again then.
- [NEEDS ATTORNEY REVIEW item]: the question for the attorney, in one sentence.
- [NOT SCREENED line only when the facts raise one: relief type, the referral line from not-screened.md, the referral target from the profile]
- Not evaluated: Proposition 47, Proposition 64 and PC 17(b); full analysis decides them.

**Facts to confirm:** [only when there are any; one line each: the fact, the item, why it matters]

**One question I'd ask that isn't in my checklist:** [one sentence, or omit the line]

**What next?** 1 draft the attorney review memo · 2 queue for [attorney] with the flags up front · 3 get more facts as plain-language questions for the applicant · 4 filing checklist for [county] · or tell me what you'd do. Say `--full` for the long report.
```

## Filing checklist (on request)

When the user picks the filing checklist, print the header, a one-line reviewer note, and the county's material from `county-filing-guide.md`: the Table A row as a short list (forms, service, filing mode, copies, local flags), the county's notes section, and the Table B line only if the facts raised fire camp relief. Nothing else; no re-screen. End with the tag `[county filing guide, imported 2026-10-04 — verify with the court]`.

## No cases given (compact)

```markdown
[AI-ASSISTED DRAFT — requires attorney review before any client communication]

> ⚠️ Reviewer note: cards last confirmed [date] · research connector [state] · read [id]: 0 cases · nothing to flag yet

# Screen: [reference] ([record_id]), [date]

**Overall:** client-level pre-screen only: [no disqualifier found at client level | disqualifier: X]. [The five gate answers in one sentence.] No cases were given, so no case gets a band. Program gates [met | not met: gate, rule applied]. [review-model line]

**To screen, I need for each conviction:** county; year of conviction; level (infraction, misdemeanor, felony); sentence type (probation only, jail, split sentence with mandatory supervision, prison, fine only); whether probation was granted and how it ended; the month the sentence ended. Type them, or say "walk me through it" and I will ask one or two at a time.

**What next?** 1 send me the facts · 2 queue the pre-screen for [attorney] · or tell me what you'd do.
```

## Full form (`--full`)

```markdown
[AI-ASSISTED DRAFT — requires attorney review before any client communication]

> **⚠️ Reviewer note**
> - **Sources:** statute cards last confirmed [date] `[statute / regulator site]`; research connector: [CourtListener ✓ verified | not connected, cites from the relief cards and training knowledge, verify before relying]
> - **Read:** intake record [id]: [n] cases, [m] arrests; fields missing: [list or none]
> - **Flagged for your judgment:** [k] items marked `[review]`
> - **Currency:** currency-watch last verified [date][; stale, treated as a checklist only]
> - **Before relying:** [one or two actions, or "ready for your eyes"]

# Eligibility screen: [reference] ([record_id]), [date]

## Bottom line

**Overall:** [NEEDS ATTORNEY REVIEW (a client-level gate is unsure or flagged) | NOT ELIGIBLE NOW (a client-level disqualifier applies) | LIKELY ELIGIBLE n of m, NOT ELIGIBLE NOW k, NEEDS ATTORNEY REVIEW j (clean gates; counts over the cases) | client-level pre-screen only: no disqualifier found at client level]
[Two sentences: why, and the single next action.]
[QUEUED for [supervising attorney] at [queue location] | CHECK WITH [attorney] BEFORE ACTING | nothing, per the review model]
[Program gates: met | not met: [gate], [profile rule applied]]

## Client-level gates

| Gate | Answer | Effect | Rule |
|---|---|---|---|
| Pending case | [value] | [effect] | PC 1203.4(a)(1); PC 1203.41(a)(3) |
| Current supervision | [value] | [effect] | [rule] |
| Registration | [value] | [effect] | [rule] |
| Fire camp | [value] | [NOT SCREENED line or none] | PC 1203.4b |
| Trafficking or DV remedies | [value] | [staff follow-up flag or none] | PC 236.14, 1203.49 |

## Per-item screen

| Item | Facts used | Band | Route and rule | Flags |
|---|---|---|---|---|
| [C1] | [field values only] | [band string] | [route name from the rulebook section 2a; subdivision] `[statute / regulator site]` | [flags, `[review]`, `[model calculation — verify]`] |

## Next steps by item

- [C1]: declaration usually needed: [yes | no]; [the card's next-step text with subdivision cites]
- [NOT ELIGIBLE NOW items]: log the recheck date; [NEEDS ATTORNEY REVIEW items]: the question for the attorney
- Filing in [county]: [forms, proof of service, who is served and how, declaration when usually needed, one local point] `[county filing guide, imported 2026-10-04 — verify with the court]`

## Automatic relief check

[Items where PC 1203.425 may already have acted, each with the conditions met, and the rulebook's standard text; or the condition that fails and when it is met.]

## Not screened

- [Relief type]: [referral line from not-screened.md]; referral target: [from profile].
- Proposition 47, Proposition 64 and PC 17(b) were not evaluated; full analysis decides them.

## Facts to confirm before attorney review

- [ ] [fact], [which item], [why it matters]

**One question I'd ask that isn't in my checklist:** [one sentence, or omit the line]

**What next? Pick one and I'll help you build it out:**
1. **Draft the attorney review memo**: one page: facts relied on, bands, the open questions, the recommended order of work.
2. **Queue for [supervising attorney]**: a short summary for the review queue with the flags up front.
3. **Get more facts**: plain-language questions for the applicant, grouped by which band they would change.
4. **Log recheck dates**: the dates on which NOT ELIGIBLE NOW items should be screened again, with the rule each date comes from.
5. **Something else**: tell me what you'd do with this.
```

## Example: fixture 01 in the compact form

```markdown
[AI-ASSISTED DRAFT — requires attorney review before any client communication]

> ⚠️ Reviewer note: cards last confirmed 2026-10-04 · research connector not connected, cites from the cards, verify before relying · read synthetic-01: 1 case, 0 arrests, nothing missing · no flags · ready for your eyes

# Screen: D.R. (synthetic-01), 2026-10-04

**Overall:** LIKELY ELIGIBLE 1 of 1, NOT ELIGIBLE NOW 0, NEEDS ATTORNEY REVIEW 0. Client-level gates clear (no pending case, no supervision, no registration, no fire camp, no trafficking or DV flag). Program gates met. QUEUED for [supervising attorney] at [queue].

**C1, PC 484(a) misdemeanor, Sample County 2017: LIKELY ELIGIBLE.** PC 1203.4 mandatory route: probation fulfilled; subdivision (a)(1), first clause; probation completed 2019-06 with no violations reported, and the person is not serving a sentence, on probation or charged. Declaration usually needed: no. PC 1203.4(b) and (c) checks run: PC 484(a) is not listed and is not a Vehicle Code offense. AUTOMATIC RELIEF MAY APPLY: probation completed without revocation, so check the RAP sheet for a "relief granted" note before filing (PC 1203.425(a)(1)(B)(iv)(I)(ia), (a)(2)(B)). `[statute / regulator site]`

**Next steps**
- C1: petition for dismissal in the court of conviction, in person, by attorney or by the authorized probation officer (PC 1203.4(a)(1)); 15 days' notice to the prosecuting attorney before relief (PC 1203.4(d)(1)); unpaid restitution is not a bar (PC 1203.4(c)(3)).
- Filing in Sample County (not in the county table, statewide default): CR-180 petition, CR-181 order, proof of service per the court's local practice (CR-106 by mail or POS-050 electronic); serve the prosecuting attorney at least 15 days before relief; confirm the forms on the court's website. `[county filing guide, imported 2026-10-04 — verify with the court]`
- Not evaluated: Proposition 47, Proposition 64 and PC 17(b); full analysis decides them.

**One question I'd ask that isn't in my checklist:** Has anything been charged since 2019, given the seven-year gap the record leaves unconfirmed?

**What next?** 1 draft the attorney review memo · 2 queue for [supervising attorney] with the flags up front · 3 get more facts as plain-language questions for the applicant · 4 filing checklist for Sample County · or tell me what you'd do. Say `--full` for the long report.
```
