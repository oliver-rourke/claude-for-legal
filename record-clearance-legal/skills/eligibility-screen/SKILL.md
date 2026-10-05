---
name: eligibility-screen
description: >
  Screen a normalized record clearance intake against California dismissal
  statutes (PC 1203.4 and PC 1203.41) and classify each conviction as LIKELY
  ELIGIBLE, NOT ELIGIBLE NOW with a recheck date, or NEEDS ATTORNEY REVIEW,
  with an AUTOMATIC RELIEF MAY APPLY flag where PC 1203.425 may already have
  acted. Also screens PC 1203.4a. With no record, walks an attorney through a
  short set of questions from memory or the court file. Produces an
  attorney-review packet, never a determination. Use when staff ask "is this
  person eligible", "screen this intake", "walk me through one", or right
  after /record-clearance-legal:intake-import.
argument-hint: "[path to a normalized intake record, paste it, or --interactive]"
---

# /eligibility-screen

1. Load `~/.claude/plugins/config/claude-for-legal/record-clearance-legal/CLAUDE.md` (and `~/.claude/plugins/config/claude-for-legal/company-profile.md`). Stop if the profile is missing or still has `[PLACEHOLDER]` markers.
2. Step 0 stop conditions: `## Jurisdiction` State must be CA; read `references/currency-watch.md` and announce staleness past 90 days; probe the research connector.
3. Read `references/screening-bands.md` and the cards in `references/relief/`. The bands file is the rulebook; apply it exactly.
4. Read the record, or, when there is none, run the baseline interview in `references/baseline-questions.md`. Refuse any record that carries a name, contact detail or date of birth.
5. Program gates, then client-level gates, then per-item screen, automatic relief flag, not-screened lines, roll-up.
6. Output with `references/report-template.md`. Close with the decision tree from the profile's `## Outputs`.

```
/record-clearance-legal:eligibility-screen references/sample-intakes/01-likely-eligible.md
```

```
/record-clearance-legal:eligibility-screen
(then paste the normalized record)
```

---

# Eligibility Screen

## Purpose

Staff at a record clearance program spend their first hour with every applicant answering one question: is there anything here worth an attorney's time, and what is missing before the attorney can say so? This skill answers that question from the intake alone, in the attorney's language, with every rule pinned to a subdivision of the statute and every uncertainty flagged where the attorney will see it.

**What it doesn't do:** decide eligibility. It routes. It does not read a RAP sheet, does not evaluate code-section lists, does not consider Proposition 47, Proposition 64 or PC 17(b), does not draft a petition, and does not talk to the applicant. With no record it asks the attorney questions instead. Those belong to the attorney, to full analysis, or to a later version.

**Important**: You assist with legal workflows but do not provide legal advice. All analysis should be reviewed by qualified legal professionals before being relied upon.

## Load context

`~/.claude/plugins/config/claude-for-legal/record-clearance-legal/CLAUDE.md` sections `## Who's using this`, `## Program eligibility gates`, `## Jurisdiction`, `## Relief types enabled`, `## Review model`, `## Referral targets`, `## Outputs`, `## Shared guardrails`.

Plugin references: `references/screening-bands.md` (the rulebook), `references/baseline-questions.md` (the interview and route names), `references/relief/pc-1203-4.md`, `references/relief/pc-1203-4a.md`, `references/relief/pc-1203-41.md`, `references/relief/not-screened.md`, `references/report-template.md`, and `../../references/currency-watch.md`.

## Workflow

### Step 0: Preconditions

Three checks, in this order. Each one can end the run.

**Profile.** If the config file is missing or contains `[PLACEHOLDER]`, print the setup message from the profile's header comment and stop.

**Jurisdiction hard stop.** Read `## Jurisdiction` → State. If it is anything other than `CA`, print this and stop:

> This plugin ships relief cards for California only. Your profile's state is [state]. I will not run a California screen on [state] facts; the waiting periods, exclusions and procedures would be wrong while looking right. To screen [state] convictions, add a card set under `skills/eligibility-screen/references/relief/` using `_template.md` (one card per relief type, written from [state]'s current statute text), add a `[state]` routing section to `references/screening-bands.md`, and re-run. Until then, route [state] matters to a practitioner there.

Do not fall back to California rules. Do not screen "provisionally." The silent-degradation case is the failure this stop exists to close.

**Currency.** Read `references/currency-watch.md`. Compute the days since its `Last verified` date. If more than 90, add to the reviewer note: "currency-watch last verified [date], [N] days ago; stale, treated as a checklist only," and add one line under the bottom line saying the statute cards may be out of date.

**Research connector.** If CourtListener (or another research MCP) is configured, make one cheap call to see whether it responds. Record the result in the reviewer note's **Sources** line: `research connector: CourtListener ✓ verified` only if a call succeeded this session; otherwise `not connected, cites from the relief cards and training knowledge, verify before relying`.

### Step 1: Read the record

Accept a file path or pasted text. The record must have the shape in `../intake-import/references/intake-schema.md`: `record_id`, `source`, `program_gates`, `person`, `status`, `cases`, `arrests_without_conviction`, `consents`.

- If the record contains a name, email address, phone number, street address, date of birth, Social Security number or CII number, stop and say: "This record contains [field]. The screen never reads identifiers. Run `/record-clearance-legal:intake-import` first; it drops them and keeps initials or a clinic ID." Do not echo the value.
- A missing `status` field is `unsure`; a missing case field is `unknown`. Both are review triggers under the bands file, and both go in the facts-to-confirm list.
- `cases: []` is valid. The run then produces a client-level pre-screen (Step 7, rule 4) and a fact request. Never invent a case.

Record what you read for the reviewer note: record ID, number of cases, number of arrests, missing fields.

**Interactive path.** When the user gives no record, pastes prose about a person's convictions, or asks to be walked through (or passes `--interactive`), run the baseline interview in `references/baseline-questions.md`. First, map everything the prose already said onto the record fields and show that draft; then ask only the questions whose fields are still `unknown` or `unsure`, in the reference's order, at most two per turn, offering the listed options and stopping as soon as a route is settled. Do not re-ask a question the prose answered, and do not ask a question whose answer cannot change the route. Build the record as YAML in the exact shape, field names and vocabularies of `../intake-import/references/intake-schema.md` (copy the schema's block and fill it; do not invent keys), echo it in a fenced block, get a yes, then continue from Step 2. Never ask for a RAP sheet or a document; if one is offered, say the screen does not read documents and continue with the questions. "Not sure" becomes `unsure` or `unknown` and the bands flag it.

### Step 2: Program gates

Compare `program_gates` with `## Program eligibility gates`. Report each gate as met, not met, or not required. A failed required gate is a program decision, not a legal one: say so first, apply the profile's "When a gate fails" rule, and if the profile is silent, run the legal screen anyway and mark the packet "program gate not met" so the attorney can decide on a referral with the screen in hand.

### Step 3: Client-level gates

Apply `references/screening-bands.md` section 1 to `status`, field by field, and fill the Client-level gates table in the report: gate, answer, effect on the packet, rule with subdivision. These run before any case is looked at. An `unsure` on `pending_case` or `current_supervision` makes every later band provisional; say so in the bottom line.

### Step 4: Per-item screen

Route each case with section 2 of the bands file, then apply section 3 (PC 1203.4), section 3a (PC 1203.4a) or section 4 (PC 1203.41). For each case write one row: item, the facts used (field values, nothing else), band, the rule applied with its subdivision, the source tag from the card, and flags. Rules:

- Use only the five band strings. Never write "eligible" or "ineligible" as a conclusion.
- Name the route in the Route and rule cell using the names in `references/baseline-questions.md` (for example "PC 1203.4 mandatory route: probation fulfilled"), followed by the subdivision, and state in Next steps by item whether a declaration is usually needed.
- Quote the card, not memory. If a rule you need is not on a card, say so, tag the item `[model knowledge — verify]`, and route it to `NEEDS ATTORNEY REVIEW`.
- When `code_section` is blank on a PC 1203.4 or PC 1203.4a item, say the exclusion check (1203.4(b) and (c), or 1203.4a(d)) could not be run and list it under facts to confirm. On a PC 1203.41 item there is no such check; the only exclusion is registration on a state prison felony (1203.41(a)(6)), so do not list one.
- Arrests in `arrests_without_conviction` are `NOT SCREENED`; print the PC 851.91 and PC 851.93 referral lines from `not-screened.md`. Never state a limitations period or a number of years for one; PC 851.91 turns on it and the attorney determines it. Write "the attorney confirms whether the limitations period has run."
- A conviction from another state or a federal court is `NOT SCREENED`, with the jurisdiction named, and the packet notes it.
- For every `LIKELY ELIGIBLE` item, copy the card's next-step text (petition in the court of conviction, 15 days' notice to the prosecutor, restitution not a bar) into the report's **Next steps by item** section, with the subdivision cites.
- Never list restitution as a fact to confirm; an unpaid order is not a ground for denial (PC 1203.4(c)(3)(A), PC 1203.41(d)).
- Any Vehicle Code `code_section` on a 1203.4 item routes to `NEEDS ATTORNEY REVIEW` (PC 1203.4(c)(1), (c)(2)); when the code section is blank, ask whether it was a Vehicle Code offense.

### Step 5: Automatic relief flag

Apply section 5 of the bands file. The flag attaches to an item; it is never the item's band. Print the standard text about the "relief granted" note.

### Step 6: Recheck arithmetic

When a waiting period has not run:

- Compute the recheck date as the completion month plus the period, from `sentence_completed`, never from a bucket alone. Show the arithmetic ("2025-08 + 24 months = 2027-08") and tag it `[model calculation — verify]`.
- When `notes` give two candidate dates (release from custody, discharge from supervision), compute both, show both, and add `[review]` asking which one is "completion of the sentence" under PC 1203.41(a)(2). If the two readings give different bands (one period has run, the other has not), the item is `NEEDS ATTORNEY REVIEW`; if both give the same band, keep it.
- When the sentence type could be mandatory supervision or PRCS, compute the recheck date under each reading, put both dates in the item's Flags cell, and add `[review]` on the supervision type. If both readings give the same band, keep that band; if they diverge, use `NEEDS ATTORNEY REVIEW`.
- Use the session date as today. Say what date you used.

### Step 7: Not screened and roll-up

List the relief types the facts suggest from `## Relief types enabled` rows marked "no" or "flag only," each with its referral line from `not-screened.md` and the referral target from the profile. List only the types the facts suggest (a fire camp answer, an arrest, a misdemeanor without probation, a trafficking flag); do not enumerate types the record gives no reason to raise. Always add the standing line: "Proposition 47, Proposition 64 and PC 17(b) were not evaluated; full analysis decides them."

Then roll up with section 6 of the bands file: `NEEDS ATTORNEY REVIEW` if any client-level gate was unsure or flagged; else `NOT ELIGIBLE NOW` only if a client-level disqualifier applies (`pending_case: yes` or `current_supervision: probation`); else the count summary in exactly the form "LIKELY ELIGIBLE n of m, NOT ELIGIBLE NOW k, NEEDS ATTORNEY REVIEW j"; or, with no cases, the client-level pre-screen plus the fact request for the six case fields. A per-item `NOT ELIGIBLE NOW` never becomes the packet band.

### Step 8: Review model routing

Read `## Review model`. Formal review queue: add `QUEUED for [supervising attorney]` under the bottom line and name the queue location. Configurable flags: when a trigger in the profile fires, add `CHECK WITH [attorney] BEFORE ACTING`. Lighter-touch: no extra line.

### Step 9: Write the report

Use `references/report-template.md` exactly: the header line first, the reviewer note next, then the report. Keep the body clean (no narration of what you read; that lives in the reviewer note). Every date you computed carries `[model calculation — verify]`; every judgment call carries `[review]`; every cite carries its source tag.

### Step 10: Close

End with "**One question I'd ask that isn't in my checklist:**" when you have a real one, then the decision tree from the profile's `## Outputs`, with all five screening options in the template's order; do not reorder or drop options. When the user picks one, do that thing; do not re-explain the screen.

## Output

See `references/report-template.md`. The first line of every report is the work-product header from the profile's `## Outputs`. The last block is the decision tree.

## Worked rows

From `references/sample-intakes/01-likely-eligible.md` and `02-not-yet.md`, the per-item table looks like this (today taken as 2026-10-04):

| Item | Facts used | Band | Rule applied | Source | Flags |
|---|---|---|---|---|---|
| C1 (01) | misdemeanor; probation_granted yes; probation_outcome completed; code_section PC 484(a) | LIKELY ELIGIBLE | PC 1203.4 mandatory route: probation fulfilled; (a)(1) first clause; not serving, on probation or charged; declaration usually needed: no | `[statute / regulator site]` `[settled — last confirmed 2026-10-04]` | AUTOMATIC RELIEF MAY APPLY (PC 1203.425(a)(1)(B)(iv)(I)(ia)); PC 1203.4(b) check run: PC 484(a) not listed |
| C2 (02) | felony; sentence prison; sentence_completed unknown; notes: released 2024-02, parole discharged 2025-08 | NEEDS ATTORNEY REVIEW | PC 1203.41(a)(2): two years after completion of a state prison sentence; the two readings of completion diverge (2026-02 has passed, 2027-08 has not) | `[statute / regulator site]` `[settled — last confirmed 2026-10-04]` | recheck 2025-08 + 24 months = 2027-08, or 2024-02 + 24 months = 2026-02 `[model calculation — verify]`; `[review]` which date is completion of the sentence; PC 1203.41(a)(6) satisfied by registration_290 no |
| A1 (02) | arrest 2015; charges_filed no | NOT SCREENED | PC 851.91 petition if the limitations period has run; PC 851.93 automatic relief may already appear | `[statute / regulator site]` | referral per profile |
| C2 (03) | felony; sentence split_mandatory_supervision; sentence_completed 2026-02; notes: applicant called it community supervision | NOT ELIGIBLE NOW (same band under both readings) | PC 1203.41(a)(2): one year after a PC 1170(h)(5)(B) sentence, two years after a prison term followed by PRCS | `[statute / regulator site]` `[settled — last confirmed 2026-10-04]` | as mandatory supervision: 2026-02 + 12 months = 2027-02; as PRCS after prison: 2026-02 + 24 months = 2028-02 `[model calculation — verify]`; `[review]` which supervision type it was |

## What this skill does NOT do

- **Decide eligibility.** Bands are routing for the supervising attorney.
- **Read RAP sheets or documents.** It reads the normalized record only.
- **Evaluate code-section lists.** Proposition 47, Proposition 64, PC 17(b) and any "is this offense a wobbler" question go to full analysis.
- **Ask for documents.** The interactive path asks questions; it never asks for a RAP sheet.
- **Compute from buckets.** A "less than 2 years ago" answer produces a fact request, not a date.
- **Communicate with the applicant.** Plain-language questions it drafts go to the attorney first.
- **Screen outside California.** The hard stop is not overridable from the conversation.

## Close with the next-steps decision tree

End with the decision tree per the profile's `## Outputs`. The five default branches are: draft the attorney review memo, queue for the supervising attorney, get more facts, log recheck dates, something else. The tree is the output; the attorney picks.
