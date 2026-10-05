---
name: intake-import
description: >
  Check 1 of 2 for a record clearance program: import an intake (pasted
  answers, a CSV export row, or a Typeform or Airtable record) into the
  normalized record with identifiers dropped, then triage it: proceed to the
  screen, needs attorney review, not eligible now with a recheck, or not
  eligible for the program. Use when staff say "import this intake", "triage
  this applicant", "normalize this Typeform response", "load the Airtable
  record", or before /record-clearance-legal:eligibility-screen. For one
  applicant the screen can run both checks from a paste.
argument-hint: "[--paste | --csv <path> | --airtable <record id> | --typeform <response id>]"
---

# /intake-import

Paths below are relative to the plugin root. The profile is `~/.claude/plugins/config/claude-for-legal/record-clearance-legal/CLAUDE.md`.

1. Load the profile → `## Intake source mapping`, `## Program eligibility gates`, `## Who's using this`. Missing: print the setup message from the template's header comment and stop. Leftover `[PLACEHOLDER` markers: use the template's defaults for those sections and say so in the reviewer note.
2. **Step a, import.** Read `skills/intake-import/references/intake-schema.md`; choose the source; map every field ("I'm not sure" becomes `unsure`; anything the form did not ask becomes `unknown`); drop identifiers and say which categories.
3. **Step b, triage.** Apply `skills/intake-import/references/triage-rules.md` in order and write its one-line result: PROCEED TO SCREEN, NEEDS ATTORNEY REVIEW, NOT ELIGIBLE NOW with the recheck, or NOT ELIGIBLE FOR THE PROGRAM with the reason.
4. Emit the triage line, the record, a completeness table and the flags, then hand off to `/record-clearance-legal:eligibility-screen` for check 2 when triage says proceed or review.

```
/record-clearance-legal:intake-import
(then paste the form answers or an export row)
```

```
/record-clearance-legal:intake-import --csv ./exports/intake-2026-10-04.csv
```

---

# Intake Import and Triage

## Purpose

Intake forms collect what a program needs to run: contact details, consents, demographics, program gates, and a handful of screening answers. The eligibility screen needs a small, fixed subset of that, with no identifiers, in one shape, no matter which form or vendor produced it. This skill is that seam, and it runs the first of the plugin's two checks: is this applicant worth an attorney's time right now, or is there an instant stop, a wait, or something to follow up first? Most intake forms collect these answers; few carry the routing logic. The triage rules supply it for any source.

It also catches the two things forms get wrong for screening: "I'm not sure" answers that a spreadsheet silently treats as "no," and supervision labels ("community supervision") that hide a waiting-period question.

**One applicant, one paste.** `/record-clearance-legal:eligibility-screen` runs the same import and triage inline on a paste and continues into check 2, so staff with one intake in hand can use the screen alone. Use this skill for CSV exports, Typeform and Airtable records, batches, and whenever the triage result is the deliverable.

**What it doesn't do:** band a conviction (check 2 does), ask for case facts (the screen does), store anything, or write back to a form or a database.

**Important**: You assist with legal workflows but do not provide legal advice. All analysis should be reviewed by qualified legal professionals before being relied upon.

## Load context

The profile → `## Intake source mapping` (primary source, IDs, field-map overrides, identifier policy, prior review decision field), `## Program eligibility gates` (which gates exist, whether consents are required), `## Available integrations`, `## Who's using this`, `## Referral targets` (county referral list, RAP sheet guidance).

`skills/intake-import/references/intake-schema.md` → the record shape, allowed values, the minimal record the screen reads, the 45-row mapping from the reference form, and the list of fields that are never imported.

`skills/intake-import/references/triage-rules.md` → the order of evaluation and the four outcomes; it cites `skills/eligibility-screen/references/screening-bands.md` section 1 for the legal effect of each answer.

## Workflow

### Step 1: Choose the source

| Flag | What you read | Fallback |
|---|---|---|
| `--paste` (default) | Form answers pasted as question-and-answer lines, a JSON object, or one export row with its header | none needed |
| `--csv <path>` | The header row and one data row. With several rows, list the `record_id` or submission time of each and ask which one; never import a whole file in one turn without asking | ask for a paste if the file cannot be read, per the file-access rule in the profile |
| `--typeform <response id>` | One response through the Typeform connector, read-only (list or get tools only) | if the connector is not configured or not responding, say so and ask for a paste or a CSV export |
| `--airtable <record id>` | One record through the Airtable connector, read-only (list or get tools only) | same |

Never call a tool that creates, updates or deletes anything in Typeform or Airtable. If the only tools offered are write tools, do not use them.

### Step 2: Map the fields

Use the mapping table in `intake-schema.md`, then the profile's field-map overrides. For each record field write the value in the schema's vocabulary:

- "I'm not sure" (or any hedge: "maybe," "not sure," blank on a required screening question) becomes `unsure` on a `status` field and `unknown` on a case field.
- "Yes, on community supervision" becomes `current_supervision: mandatory_supervision` with the flag "confirm mandatory supervision versus post-release community supervision." The same applies to the past-three-years question.
- A bucketed release answer ("less than 2 years ago") goes to `supervision_ended_bucket`; the release month goes to `months_since_supervision_ended` or the notes only when staff supply it.
- A field the form did not ask is `unknown`, never a guessed `no`. For the five consents: a consent question that was on the form but answered no or left blank is `no`; a source with no consent questions at all leaves them `unknown`, and triage then asks for them as a follow-up instead of stopping.
- A staff field recording a prior review decision maps to `program_gates.prior_review_decision` (`cleared`, `not_eligible`, `none`).
- Program-gate answers are recorded as given. Triage judges them against the profile; this step does not.

### Step 3: Drop identifiers

Remove name, email, phone, mailing address, date of birth, Social Security number, CII, SID, FBI or driver's license number, case, docket or booking number, attorney or public defender names, registry dates, attachment links and uploaded files. Set `person.reference` per the profile's identifier policy (initials from the name fields, or the clinic ID). Then say, in one line, which categories were dropped, for example: "Dropped at import: name, email, phone, address, date of birth, attachment link." Do not print the dropped values anywhere, including in that line.

This rule covers everything you write, not only the record: your reply, your summary of what you did, and any file you save refer to the applicant only as `person.reference` (for example "the applicant, P.E."). Writing the name once in a summary sentence is the same failure as writing it in the record.

A pasted RAP sheet or docket excerpt is acceptable input once identifiers are gone. Map its lines to case facts: the court name gives the county (and the courthouse, which some county rules use); a date gives `sentencing_date` and `conviction_year`; MISDEMEANOR, FELONY or INFRACTION gives `offense_type`; the section with its code prefix gives `code_section` (for example `HS 11377(a)`); the sentence line ("002 YEARS PROBATION", "016 MONTHS JAIL", "003 YEARS PRISON") gives `sentence_term` and the sentence type; the disposition ("successfully completed", "probation revoked") gives `probation_outcome`. Never store or echo the raw lines, never ask for the sheet, and say that the attorney still reads the full RAP sheet. A whole uploaded document is still refused: ask for the conviction lines as text.

### Step 4: Triage (check 1)

Apply `triage-rules.md` in its order of evaluation and write its one-line result. Collect every follow-up (gates unknown, trafficking or DV, RAP sheet on hand or not) and every recheck trigger. Where a date can be computed from a known month, show the arithmetic and tag it `[model calculation — verify]`; from a bucket alone, give the trigger in words.

### Step 5: Emit the record

Print, in this order:

1. The work-product header from the profile's `## Outputs`.
2. A one-line reviewer note: source, number of fields mapped, categories dropped, number of flags.
3. The triage line from Step 4, then one line per follow-up and recheck.
4. The normalized record as a fenced YAML block in the schema's shape and order. Include `record_id`, `source`, and `received` (the submission date from the form, date only). `record_id` is the clinic's own ID when the form carries one; otherwise build it as `[source]-[received]-[reference]`, for example `typeform-2026-09-29-PE`. Never reuse a sample fixture's ID. Fields the schema marks optional may be omitted when the form did not collect them.
5. A completeness table:

| Field | Present | Needed by |
|---|---|---|
| `status.pending_case` | yes / unsure / missing | triage, client-level gate |
| `status.current_supervision` | ... | triage, client-level gate |
| `status.registration_290` | ... | triage, client-level gate |
| `consents.*` | n of 5 | triage |
| `cases[]` | n entries / none | check 2 |
| `cases[].sentence_term` or `sentence_completed` | known for n of m | recheck dates |

6. The flags list: every `unsure`, every "confirm mandatory supervision versus post-release community supervision," every bucket without a month, every required field that was missing from the form.

### Step 6: Hand off

- `PROCEED TO SCREEN` or `NEEDS ATTORNEY REVIEW`: "Record ready. Run `/record-clearance-legal:eligibility-screen` on it now for check 2? It asks for the per-case facts (county, year, offense type, sentence type and term, probation grant and how it ended, sentencing month) one or two at a time, or takes them typed, and confirms them as yes-or-no lines before screening."
- `NOT ELIGIBLE NOW`: the recheck trigger or date and the referral (early termination of probation, fire camp), then "Check 2 can still run on request, for the attorney's information."
- `NOT ELIGIBLE FOR THE PROGRAM`: the reason and the profile's "When a gate fails" step (usually the county referral list, or the consent form again), then the same offer.

If the user wants a copy on disk: "I can write it to `./intakes/[record_id].md`; your profile's retention rule applies to that file." Write the file only after an explicit yes. Never write it anywhere that syncs to a shared drive unless the user names that destination and the destination check in the profile's `## Shared guardrails` passes.

## Output

```markdown
[AI-ASSISTED DRAFT — requires attorney review before any client communication]

> ⚠️ Reviewer note: source [pasted | csv path | typeform response id | airtable record id] · [n] fields mapped · dropped at import: [categories] · [k] flags

**Check 1, intake triage:** [OUTCOME]. [reasons]. [follow-ups]. [recheck].

```yaml
intake:
  record_id: ...
  (record in schema order; optional fields omitted when not collected)
```

## Completeness
| Field | Present | Needed by |
|---|---|---|

## Flags
- [flag]

[hand-off line for the outcome]
```

## What this skill does NOT do

- **Band a conviction.** Check 2 does that with the statute; triage routes the applicant.
- **Ask for case facts.** The screen asks, one or two at a time, and never for a document.
- **Store identifiers.** Nothing from the dropped categories survives into the record, the reply, the conversation summary, or a saved file.
- **Write to Typeform or Airtable.** Read-only, one record at a time.
- **Read whole documents.** A RAP sheet excerpt as text, with identifiers removed, is case facts; the uploaded document itself stays out of the session.
- **Guess.** A question the form did not ask is `unknown`; a hedge is `unsure`; a consent left blank on a form that asked it is `no`.
