---
name: intake-import
description: >
  Turn a record clearance intake into the normalized intake record the
  eligibility screen reads: from pasted form answers, a CSV export row, or a
  Typeform or Airtable record when those connectors are live. Drops names and
  contact details at import, maps "I'm not sure" answers to flags, and asks
  for the per-case facts the form does not collect. Use when staff say
  "import this intake", "normalize this Typeform response", "load the Airtable
  record", or before /record-clearance-legal:eligibility-screen.
argument-hint: "[--paste | --csv <path> | --airtable <record id> | --typeform <response id>]"
---

# /intake-import

1. Load `~/.claude/plugins/config/claude-for-legal/record-clearance-legal/CLAUDE.md` → `## Intake source mapping`, `## Program eligibility gates`, `## Who's using this`. Stop if the profile is missing or has `[PLACEHOLDER]` markers.
2. Read `references/intake-schema.md`. The record shape and the mapping table there are the contract.
3. Choose the source: pasted text (default), a CSV row, or a Typeform or Airtable record through a connector that is actually responding. Connectors are read-only here.
4. Map every field. "I'm not sure" becomes `unsure`; anything the form did not ask becomes `unknown`.
5. Drop identifiers. Keep initials or the clinic ID as `person.reference`. Say what was dropped; never echo the values.
6. Ask for per-case facts if staff have them. Accept "skip".
7. Emit the record, a completeness table, and the flags. Offer `/record-clearance-legal:eligibility-screen`.

```
/record-clearance-legal:intake-import
(then paste the form response or export row)
```

```
/record-clearance-legal:intake-import --csv ./exports/intake-2026-10-04.csv
```

---

# Intake Import

## Purpose

Intake forms collect what a program needs to run: contact details, consents, demographics, program gates, and a handful of screening answers. The eligibility screen needs a small, fixed subset of that, with no identifiers, in one shape, no matter which form or vendor produced it. This skill is the seam between the two. It also catches the two things forms get wrong for screening: "I'm not sure" answers that a spreadsheet silently treats as "no," and supervision labels ("community supervision") that hide a waiting-period question.

**What it doesn't do:** judge eligibility, store anything, or write back to a form or a database.

**Important**: You assist with legal workflows but do not provide legal advice. All analysis should be reviewed by qualified legal professionals before being relied upon.

## Load context

`~/.claude/plugins/config/claude-for-legal/record-clearance-legal/CLAUDE.md` → `## Intake source mapping` (primary source, IDs, field-map overrides, identifier policy), `## Program eligibility gates` (which gate questions exist), `## Available integrations` (which connectors are configured), `## Who's using this`.

`references/intake-schema.md` → the record shape, allowed values, the 45-row mapping from the reference form, and the list of fields that are never imported.

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

Use the mapping table in `references/intake-schema.md`, then the profile's field-map overrides. For each record field write the value in the schema's vocabulary:

- "I'm not sure" (or any hedge: "maybe," "not sure," blank on a required screening question) becomes `unsure` on a `status` field and `unknown` on a case field.
- "Yes, on community supervision" becomes `current_supervision: mandatory_supervision` with the flag "confirm mandatory supervision versus PRCS." The same applies to the past-three-years question.
- A bucketed release answer ("less than 2 years ago") goes to `supervision_ended_bucket`; `months_since_supervision_ended` stays `unknown` until staff supply a number.
- A field the form did not ask is `unknown`, never a guessed `no`.
- Program-gate answers are recorded as given. Do not judge them here; the screen reports them against the profile.

### Step 3: Drop identifiers

Remove name, email, phone, mailing address, date of birth, Social Security number, CII number, attorney or public defender names, registry dates, attachment links and uploaded files. Set `person.reference` per the profile's identifier policy (initials from the name fields, or the clinic ID). Then say, in one line, which categories were dropped, for example: "Dropped at import: name, email, phone, address, date of birth, attachment link." Do not print the dropped values anywhere, including in that line.

This rule covers everything you write, not only the record: your reply, your summary of what you did, and any file you save refer to the applicant only as `person.reference` (for example "the applicant, P.E."). Writing the name once in a summary sentence is the same failure as writing it in the record.

If the paste contains a RAP sheet or a court document, stop and say: "That looks like a RAP sheet or court record. This plugin never reads those; keep them in the clinic's document system and give me the intake answers only."

### Step 4: Ask for per-case facts

The reference form collects no case-level facts. Ask once:

> Do you have case-level facts for this applicant? For each conviction I need six things: county, year of conviction, offense level (infraction, misdemeanor, felony), sentence type (probation only, jail, split sentence with mandatory supervision, prison, fine only), whether probation was granted and how it ended (completed, terminated early, revoked, ongoing), and the month the sentence was completed. Give me what you have, or say "skip" and the screen will run at client level.

Record what staff give in the case entry shape from the schema. When staff would rather be walked through it, hand off: `/record-clearance-legal:eligibility-screen --interactive` asks the same questions one or two at a time. Anything they do not know is `unknown`. Arrests without conviction go in `arrests_without_conviction` with year, county and whether charges were filed.

### Step 5: Emit the record

Print, in this order:

1. The work-product header from the profile's `## Outputs`.
2. A one-line reviewer note: source, number of fields mapped, categories dropped, number of flags.
3. The normalized record as a fenced YAML block in exactly the schema's shape and field order. Include `record_id`, `source`, and `received` (the submission date from the form, date only). `record_id` is the clinic's own ID when the form carries one; otherwise build it as `[source]-[received]-[reference]`, for example `typeform-2026-09-29-PE`. Never reuse a sample fixture's ID.
4. A completeness table:

| Field | Present | Needed by the screen |
|---|---|---|
| `status.pending_case` | yes / unsure / missing | client-level gate |
| `status.current_supervision` | ... | client-level gate |
| `status.registration_290` | ... | client-level gate |
| `cases[]` | n entries / none | per-item screen |
| `cases[].sentence_completed` | known for n of m | recheck dates |

5. The flags list: every `unsure`, every "confirm mandatory supervision versus PRCS," every bucket without a month count, every required field that was missing from the form.

### Step 6: Offer the next step

> Record ready. Run `/record-clearance-legal:eligibility-screen` on it now? If you want a copy on disk, I can write it to `./intakes/[record_id].md`; your profile's retention rule applies to that file.

Write the file only after an explicit yes. Never write it anywhere that syncs to a shared drive unless the user names that destination and the destination check in the profile's `## Shared guardrails` passes.

## Output

```markdown
[AI-ASSISTED DRAFT — requires attorney review before any client communication]

> **⚠️ Reviewer note**
> - **Source:** [pasted | csv path | typeform response id | airtable record id]; [n] fields mapped; dropped at import: [categories]; [k] flags

```yaml
intake:
  record_id: ...
  (full record in schema order)
```

## Completeness
| Field | Present | Needed by the screen |
|---|---|---|

## Flags
- [flag]

Record ready. Run `/record-clearance-legal:eligibility-screen` on it now?
```

## What this skill does NOT do

- **Judge eligibility.** Program gates are recorded, not applied; legal rules are not touched.
- **Store identifiers.** Nothing from the dropped categories survives into the record, the reply, the conversation summary, or a saved file.
- **Write to Typeform or Airtable.** Read-only, one record at a time.
- **Read documents.** RAP sheets, court papers and attachments stay out of the session.
- **Guess.** A question the form did not ask is `unknown`; a hedge is `unsure`.
