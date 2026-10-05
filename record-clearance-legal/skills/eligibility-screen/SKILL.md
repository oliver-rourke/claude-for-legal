---
name: eligibility-screen
description: >
  Screen a record clearance intake against California dismissal statutes
  (PC 1203.4, PC 1203.4a and PC 1203.41) and band each conviction LIKELY
  ELIGIBLE, NOT ELIGIBLE NOW with a recheck date, or NEEDS ATTORNEY REVIEW,
  with an AUTOMATIC RELIEF MAY APPLY flag where PC 1203.425 may already have
  acted. Takes a normalized record, pasted intake answers or prose, or, with
  nothing, asks an attorney a short set of questions from memory. Produces a
  short attorney-review screen, never a determination. Use when staff ask "is
  this person eligible", "screen this intake", "walk me through one", or paste
  an intake.
argument-hint: "[record path or paste | intake answers or prose | --interactive] [--full] [--no-confirm]"
---

# /eligibility-screen

Paths below are relative to the plugin root. The profile is `~/.claude/plugins/config/claude-for-legal/record-clearance-legal/CLAUDE.md`.

1. Step 0, once per session: profile present (leftover placeholders become named defaults, except State); State is CA or stop; currency check; research connector probe.
2. Read the input: a normalized record, pasted intake answers, a RAP sheet excerpt with identifiers removed, or prose (map them, drop identifiers), or nothing (the interview in `skills/eligibility-screen/references/baseline-questions.md`). Then check 1, triage, from `skills/intake-import/references/triage-rules.md`. Then the verification checklist: numbered yes-or-no lines the user confirms before the screen; `--no-confirm` or "just screen" skips it.
3. Check 2: apply the rulebook, `skills/eligibility-screen/references/screening-bands.md`: per-item routes, automatic relief flag, recheck arithmetic, roll-up.
4. Write the compact report from `skills/eligibility-screen/references/report-template.md`; `--full` gives the long form. Close with the decision tree.

```
/record-clearance-legal:eligibility-screen
(then paste the intake answers, a normalized record, or describe the conviction)
```

```
/record-clearance-legal:eligibility-screen references/sample-intakes/01-likely-eligible.md --full
```

---

# Eligibility Screen

## Purpose

Staff at a record clearance program spend their first hour with every applicant on one question: is there anything here worth an attorney's time, and what is missing before the attorney can say so? This skill answers it from the intake alone, in the attorney's language, with every rule pinned to a subdivision and every uncertainty flagged where the attorney will see it.

**What it does not do.** Decide eligibility (it routes). Read a full RAP sheet or any document (a short excerpt of conviction lines with identifiers removed is fine as case facts). Evaluate code-section lists, Proposition 47, Proposition 64 or PC 17(b). Compute a date from a bucket. Draft a petition. Talk to the applicant. Screen outside California. Those belong to the attorney, to full analysis, or to a later version.

**Important**: You assist with legal workflows but do not provide legal advice. All analysis should be reviewed by qualified legal professionals before being relied upon.

## Load context

Profile sections: `## Who's using this`, `## Program eligibility gates`, `## Jurisdiction`, `## Relief types enabled`, `## Review model`, `## Referral targets`, `## Outputs`, `## Shared guardrails`.

Plugin files, by trigger (paths from the plugin root):

| File | Load when |
|---|---|
| `skills/eligibility-screen/references/screening-bands.md` | always; it is the rulebook |
| `skills/eligibility-screen/references/report-template.md` | always |
| `references/currency-watch.md` | once per session, in Step 0 |
| `skills/eligibility-screen/references/baseline-questions.md` | the input is prose, pasted answers, or nothing |
| `skills/intake-import/references/triage-rules.md` | always; check 1 runs in Step 2 |
| `skills/intake-import/references/intake-schema.md` | mapping pasted answers, an excerpt or prose, or echoing a record |
| `skills/eligibility-screen/references/relief/pc-1203-4.md` | a case routes to rulebook section 3 |
| `skills/eligibility-screen/references/relief/pc-1203-4a.md` | a case routes to section 3a |
| `skills/eligibility-screen/references/relief/pc-1203-41.md` | a case routes to section 4 |
| `skills/eligibility-screen/references/relief/not-screened.md` | a NOT SCREENED line is needed |
| `skills/eligibility-screen/references/county-filing-guide.md` | an item is on a petition route and has a county, or someone asks what to file; read only that county's row and section |
| `skills/eligibility-screen/references/engine-crosswalk.md` | someone asks for an engine label |

Precedence when texts differ: the profile, then the rulebook, then this file, then the interview spine.

## Workflow

### Step 0: Preconditions (once per session)

Run these on the first screen of a session. On later screens in the same session, skip to Step 1 unless the profile changed.

**Profile.** Missing: print the setup message from the profile template's header comment and stop. Present but with `[PLACEHOLDER` markers: list the sections that still carry them, use the template's stated default for each, carry "Defaults applied: [sections]" into the reviewer note, and continue. One exception: a placeholder in `## Jurisdiction` State is a stop; the state must be set before any screen. Test runs: when the session's system prompt names an alternate profile path for an evaluation, read that file instead of the home path and write "test profile" in the reviewer note's Sources line.

**Jurisdiction hard stop.** Read `## Jurisdiction` → State. If it is anything other than `CA`, print this and stop:

> This plugin ships relief cards for California only. Your profile's state is [state]. I will not run a California screen on [state] facts; the waiting periods, exclusions and procedures would be wrong while looking right. To screen [state] convictions, add a card set under `skills/eligibility-screen/references/relief/` using `_template.md` (one card per relief type, written from [state]'s current statute text), add a `[state]` routing section to `screening-bands.md`, and re-run. Until then, route [state] matters to a practitioner there.

Do not fall back to California rules. Do not screen "provisionally." The silent-degradation case is the failure this stop exists to close.

**Currency.** Read `references/currency-watch.md`. The watch is current until the January 1 after its `Last verified` date, because California statutes take effect then; it is stale earlier if an entry names a pending change whose effective date has passed. If the clinic's verification log (next to the profile) has a newer entry for the currency watch, use that entry's date. When stale, add to the reviewer note "currency-watch last verified [date]; stale, treated as a checklist only" and one line under the check lines saying the statute cards may be out of date.

**Research connector.** If CourtListener (or another research MCP) is configured, make one cheap call to see whether it responds. Record the result in the reviewer note's Sources line: `research connector: CourtListener ✓ verified` only if a call succeeded this session; otherwise `not connected, cites from the cards, verify before relying`.

### Step 1: Read the input

Three kinds of input. Each ends with the verification checklist: the facts the screen will rely on, as numbered yes-or-no lines the user confirms before the screen runs. The user can skip it ("just screen", `--no-confirm`).

**A normalized record** (a file path, or a paste in the shape of `skills/intake-import/references/intake-schema.md`): accept the keys in any order. A missing `status` field is `unsure`, a missing case field is `unknown`; both are review triggers and go in facts to confirm. `cases: []` is valid and produces the no-case reply (Step 7). Never invent a case.

**Pasted intake answers, a RAP sheet excerpt or prose** (form question-and-answer lines, an export row with its header, conviction lines copied from a RAP sheet with identifiers removed, or a sentence or two about a person's convictions): map them with the schema's mapping table and the profile's field-map overrides, using the import skill's conventions: "I'm not sure" is `unsure` on a status field and `unknown` on a case field; "community supervision" is `mandatory_supervision` with a flag to confirm mandatory supervision versus post-release community supervision; a question the form did not ask is `unknown`, never a guessed no. Then run the interview in `baseline-questions.md` for whatever the paste left unknown and route-relevant: at most two questions per turn, never a question the paste answered, never a question that cannot change the route. Never ask the trafficking or domestic violence question; when the form did not ask it, record `unknown` and let a one-clause staff follow-up under Next steps carry it. When the input gives no initials or clinic ID, ask for them in the first question turn; that turn may carry three items when one of them is the initials, so county and year still go together. Until then `person.reference` is `unknown`. Build `record_id` as `[source]-[received]-[reference]`.

**Nothing, or `--interactive`**: run the interview from the start, prose first, then only the gaps.

**Identifiers, on every input.** Before echoing anything, look for a name, email address, phone number, street address, date of birth, Social Security number, CII number, attorney name, case number, registry date or attachment link. Drop each one, set `person.reference` to initials or the clinic ID per the profile's identifier policy, and print one line: "Dropped at import: [categories]". Never print a dropped value anywhere: not in the record, the echo, the report or your own summary. From here on the person is `person.reference`. A pasted RAP sheet or docket excerpt is acceptable input once identifiers are gone, and is common: a date, a court name, MISDEMEANOR or FELONY, a section with its code prefix and description, a sentence line such as "002 YEARS PROBATION", a disposition such as "successfully completed". Map it: the court name gives the county (and the courthouse, which some county rules use); the date gives `sentencing_date` and `conviction_year`; the type gives `offense_type`; the section gives `code_section` with its code (for example `HS 11377(a)`); the sentence line gives `sentence_term` and the sentence type; the disposition gives `probation_outcome`. Identifiers that appear in such text (name, date of birth, CII, SID, FBI or driver's license number, case, docket or booking number) are dropped like any other. Never store or echo the raw lines, never ask for the sheet, and note that the attorney still reads the full RAP sheet. A whole uploaded document is still refused: ask for the conviction lines as text.

**Verification checklist.** Before screening, list the facts the screen will rely on as numbered yes-or-no lines in plain words, in this order: the four client-level gates (supervision today, pending case, registration, fire camp), then for each case one line for what it is (section, offense type, county, year) and one for how it ended (probation grant, term and outcome, or custody type and term with the month it ended, actual or estimated from the term; judgment month and any new conviction on the PC 1203.4a route), then the program gates. Close with: "Reply yes to confirm all and I will screen, or give the number and the correction. Say record to see the full YAML." Then stop and wait. On yes (or proceed, go, correct), screen. On a correction, change the field, show only the changed lines, and ask again. On record, show one fenced YAML block in the schema's names with only the fields the screen uses, and ask again. Skip the checklist only when the user asked to ("just screen", "no need to confirm", `--no-confirm`); then go straight to the screen. Never ask for a RAP sheet or a document; if one is offered, say the screen does not read documents and continue. The YAML shown on request uses the schema's names and only the fields the screen uses: `record_id`, `source`, `received`, `person.reference`; the five `status` gates plus `supervision_ended_within_3_years` when it is not `none`; for each case `id`, `county`, `conviction_year`, `offense_type`, `code_section`, `sentence`, `probation_granted`, `probation_outcome`, `revocation_custody` when revoked, `judgment_date` and `new_conviction_since` on the PC 1203.4a route, `sentence_completed`, `notes`; arrests as `id`, `year`, `county`, `charges_filed`.

Record for the reviewer note: record ID, cases, arrests, missing fields, categories dropped.

### Step 2: Check 1, intake triage

Apply `skills/intake-import/references/triage-rules.md` in its order of evaluation: program gates against the profile, the prior review decision, pending case, current supervision with the fire camp exception, supervision ended in the past three years, registration, trafficking or DV, consents, then the not-sure review. It cites rulebook section 1 for the legal effect of each answer. Prose, an excerpt or the interview carries no consent questions, so consents are `unknown` there and give a follow-up, not a stop. Write its one line, `**Check 1, intake triage:** [outcome]. [reasons]. [follow-ups]. [recheck]`, as the first line of the body, followed by the review-model line from Step 8.

- `PROCEED TO SCREEN` or `NEEDS ATTORNEY REVIEW`: continue to check 2. Under review, every band is provisional; say so on the check 2 line.
- `NOT ELIGIBLE NOW` or `NOT ELIGIBLE FOR THE PROGRAM`: stop after the check 1 line, the recheck trigger or the profile's "When a gate fails" step, and the referral, and offer check 2 on request ("for the attorney's information"). Run it when the user asks or when the profile says to screen anyway, and label every band provisional.

### Step 3: Provisional bands

An `unsure` on `pending_case` or `current_supervision`, or a registration answer other than `no`, makes every later band provisional (rulebook section 1); the check 2 line says so. An unknown trafficking or DV answer is a one-clause follow-up, never a gate.

### Step 4: Per-item screen

Route each case with rulebook sections 2 and 2a, then apply section 3 (PC 1203.4), 3a (PC 1203.4a) or 4 (PC 1203.41). Copy the route name, the declaration line, the exclusion checks and the next-step text from the rulebook; do not restate them from memory. Skill rules on top:

- Use only the five band strings. Never write "eligible" or "ineligible" as a conclusion.
- Quote the card, not memory. If a rule you need is not on a card, say so, tag the item `[model knowledge — verify]`, and route it to `NEEDS ATTORNEY REVIEW`.
- The rulebook's standing rules apply every time, and these are the ones runs most often miss: restitution is never a fact to confirm; any Vehicle Code section on a PC 1203.4 item is attorney review; no limitations period is ever stated for an arrest; a PC 1203.41 item has no (b) or (c) check; when `code_section` is blank on a PC 1203.4 or PC 1203.4a item, say which exclusion check could not run and list it under facts to confirm.
- A conviction from another state or a federal court is `NOT SCREENED` with the jurisdiction named.

### Step 5: Automatic relief flag

Apply rulebook section 5. The flag attaches to an item; it is never the item's band. Compact report: one clause with the pinpoint and "check the RAP sheet for a 'relief granted' note before filing", or the condition that fails and when it is met. Full report: the rulebook's standard text.

### Step 6: Recheck arithmetic

When a waiting period has not run:

- Compute the recheck date as the completion month plus the period, from `sentence_completed` (or `judgment_date` on the PC 1203.4a route), never from a bucket alone. Show the arithmetic ("2025-08 + 24 months = 2027-08") and tag it `[model calculation — verify]`.
- When `sentence_completed` is unknown but `sentencing_date` and `sentence_term` are known, estimate completion as sentencing month plus term, show it ("2023-05 + 24 months = 2025-05") and tag it `[estimated from the term — verify against the record]`. For probation that is the probation end. For a jail term it ignores credits, so the real end is usually earlier; for a prison term parole follows release, so the estimate is a floor and the item carries `[review]` whenever its band depends on it.
- When `notes` give two candidate dates (release from custody, discharge from supervision), compute both, show both, and add `[review]` asking which one is "completion of the sentence" under PC 1203.41(a)(2). If the two readings give different bands (one period has run, the other has not), the item is `NEEDS ATTORNEY REVIEW`; if both give the same band, keep it.
- When the sentence type could be mandatory supervision or post-release community supervision, compute the recheck date under each reading, show both, and add `[review]` on the supervision type. Same band under both readings: keep it. Different bands: `NEEDS ATTORNEY REVIEW`.
- Use the session date as today. Say what date you used.

### Step 7: Not screened and roll-up

List the relief types the facts suggest from `## Relief types enabled` rows marked "no" or "flag only", each as one Next-steps bullet with its referral line from `not-screened.md` and the referral target from the profile. List only the types the facts raise (a fire camp answer, an arrest, a trafficking flag, a pre-realignment prison term); do not enumerate types the record gives no reason to raise. Always add the one-line standing bullet: "Not evaluated: Proposition 47, Proposition 64 and PC 17(b); full analysis decides them."

**Filing bullet.** For each item on a petition route (LIKELY ELIGIBLE, or NEEDS ATTORNEY REVIEW on a discretionary route), add one Next-steps bullet "Filing in [county]:" from `county-filing-guide.md`: the petition and order forms, the proof-of-service form, who is served and how, the declaration form only when a declaration is usually needed, and one local point (hearing date before filing, wet signature, copies, e-filing) when the county has one. A county not in the table gets the guide's statewide default and says so. Tag the bullet `[county filing guide, imported 2026-10-04 — verify with the court]`. List forms for relief this plugin does not screen only when the facts raised it (a fire camp answer), and never invent a service address or email; use only what the guide records. This is guidance about what to file; the plugin does not fill or file forms.

Then roll up with rulebook section 6: `NEEDS ATTORNEY REVIEW` if any client-level gate was unsure or flagged; else `NOT ELIGIBLE NOW` only if a client-level disqualifier applies (`pending_case: yes` or `current_supervision: probation`); else the count summary in exactly the form "LIKELY ELIGIBLE n of m, NOT ELIGIBLE NOW k, NEEDS ATTORNEY REVIEW j"; or, with no cases, the template's no-case form with the six-fact request. A per-item `NOT ELIGIBLE NOW` never becomes the overall band.

### Step 8: Review model routing

Read `## Review model`. Formal review queue: add `QUEUED for [supervising attorney]` to the check 1 line and name the queue location. Configurable flags: when a trigger in the profile fires, add `CHECK WITH [attorney] BEFORE ACTING`. Lighter-touch: no extra line.

### Step 9: Write the report

Compact form from the template by default; the full form with `--full`, or when the user asks for the long version afterwards. Header first, reviewer note second (one line when green), then the body. Every computed date carries `[model calculation — verify]`; every judgment call carries `[review]`; each quoted rule carries one source tag. Omit empty sections. Next steps are actions, one per item, never a restatement of the item block; "Program gates met" is three words when they are met; the review-model line is the profile's phrase only. Keep a clean one-case screen near 300 words including its filing bullet; length grows only with cases, flags and review questions. For more than about ten records in a session, offer the summary table from the profile's `## Outputs` instead of ten reports.

### Step 10: Close

Add "**One question I'd ask that isn't in my checklist:**" as one sentence when you have a real one; otherwise omit the line. Then the decision tree: in the compact form the one-line, four-option tree from the template (memo, queue, more facts, filing checklist for the county); in the full form the five-option tree from the profile's `## Outputs`, in order. When the user picks one, do that thing; do not re-explain the screen. The filing checklist is the template's on-request form: the county's row and notes from `county-filing-guide.md` under the header and a one-line reviewer note.

## Worked examples

Compact blocks from the sample intakes, today taken as 2026-10-04.

- **C1 (fixture 01), PC 484(a) misdemeanor, Sample County 2017: LIKELY ELIGIBLE.** PC 1203.4 mandatory route: probation fulfilled; subdivision (a)(1), first clause; probation completed 2019-06, no violations. Declaration usually needed: no. PC 1203.4(b) and (c) checks run: not listed, not a Vehicle Code offense. AUTOMATIC RELIEF MAY APPLY: check the RAP sheet for a "relief granted" note before filing (PC 1203.425(a)(1)(B)(iv)(I)(ia)). `[statute / regulator site]`
- **C2 (fixture 02), felony, state prison, section not known: NEEDS ATTORNEY REVIEW.** PC 1203.41 route: state prison, two years after completion; subdivision (a)(2). The notes give two completion dates and the readings diverge: released 2024-02 + 24 months = 2026-02, which has run; parole discharged 2025-08 + 24 months = 2027-08, which has not `[model calculation — verify]`. `[review]` Which date is "completion of the sentence"? Registration check under (a)(6) satisfied by `registration_290: no`. No (b) or (c) check applies to this section. `[statute / regulator site]`
- **A1 (fixture 02), 2015 arrest, no charges filed: NOT SCREENED.** PC 851.91 petition if the limitations period has run, which the attorney confirms; PC 851.93 relief may already appear on the record. Referral: the target named in the profile. `[statute / regulator site]`
- **Filing in Mono County** (a LIKELY ELIGIBLE PC 1203.4 item, declaration not usually needed): CR-180 petition, CR-181 order, CR-106 proof of service by mail; serve the DA by mail; mail or in-person filing; every case gets a hearing the court sets about a month out. `[county filing guide, imported 2026-10-04 — verify with the court]`
- **C2 (fixture 03), felony, split sentence ended 2026-02, called "community supervision" by the applicant: NOT ELIGIBLE NOW.** PC 1203.41 route; subdivision (a)(2). Same band under both readings: as mandatory supervision, 2026-02 + 12 months = 2027-02; as post-release community supervision after a prison term, 2026-02 + 24 months = 2028-02 `[model calculation — verify]`. `[review]` Which supervision type was it? Log both dates. `[statute / regulator site]`
