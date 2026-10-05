# Claude for Record Clearance

*Statute-level eligibility screening for record clearance programs, with the attorney in the loop by design.*

A plugin for the people who run record clearance (expungement) programs: legal aid offices, public defender clean slate programs, law school clinics, pro bono projects and reentry nonprofits. It turns a program's intake into a normalized record, screens that record against California's dismissal statutes, and hands the supervising attorney a packet that says what looks eligible, what is not eligible yet and when to look again, and what needs a lawyer's judgment before anyone acts.

**Every output is a draft for attorney review, marked, gated and logged. The plugin routes; it does not decide. Nothing goes to an applicant without the review model the supervising attorney set at setup.**

## The problem this solves

The first hour with every applicant goes to the same questions: is there anything here worth an attorney's time, and what is missing before the attorney can say so? Staff answer them from memory of a dozen statutes with waiting periods that changed last year, and the answers drift. Meanwhile the waitlist grows.

This plugin pins every answer to a subdivision of the statute, flags every uncertainty where the attorney will see it, and refuses to guess. The screen runs on client-level answers alone when that is all there is, and gets more precise as case facts arrive. It does not need, and does not read, a RAP sheet.

## Who uses it

| Role | Runs | Gets |
|---|---|---|
| **Supervising attorney** | `/record-clearance-legal:cold-start-interview` (once), reviews every packet | A program profile every skill reads; packets with the judgment calls already flagged |
| **Staff and volunteers** | `/record-clearance-legal:intake-import`, then `/record-clearance-legal:eligibility-screen` | A normalized intake record with identifiers dropped; a banded screen with next steps and a fact request |

## Commands

| Command | What it does | What it doesn't do |
|---|---|---|
| `/record-clearance-legal:cold-start-interview` | **Attorney.** One-time setup: role gate, ethical preconditions, program gates, jurisdiction, relief types, review model, intake source | Doesn't configure a screen for any state other than California in this version |
| `/record-clearance-legal:customize` | Change one profile section without re-running setup | Doesn't remove the load-bearing guardrails |
| `/record-clearance-legal:intake-import` | Pasted answers, a CSV row, or a Typeform or Airtable record to the normalized intake record; drops names and contact details; maps "I'm not sure" to flags; asks for per-case facts | Doesn't judge eligibility; doesn't write back to any form or base; doesn't read RAP sheets or attachments |
| `/record-clearance-legal:eligibility-screen` | Screens the record against PC 1203.4, PC 1203.4a and PC 1203.41, or, with no record, walks the attorney through five to eight questions per conviction from memory; names the route (mandatory, discretionary, split sentence, jail, prison) and whether a declaration is usually needed; bands each conviction; flags where PC 1203.425 automatic relief may already have acted; lists what was not screened with a referral; computes recheck dates with the arithmetic shown | Doesn't decide; doesn't evaluate code-section lists, Proposition 47, Proposition 64 or PC 17(b); doesn't draft petitions; doesn't run outside California |

## What v0.2 screens

| Relief | Screened | How |
|---|---|---|
| PC 1203.4 dismissal after probation | yes | mandatory routes (probation fulfilled, early discharge), discretionary route, exclusions in subdivisions (b) and (c), 15-day notice, restitution rule |
| PC 1203.4a dismissal for a misdemeanor without probation, or an infraction | yes | one year from judgment, sentence complied with, new-conviction proxy for "honest and upright life", exclusions in subdivision (d) |
| PC 1203.41 dismissal after a felony jail, split or prison sentence | yes | one-year and two-year waiting periods, supervision bar, registration bar |
| PC 1203.425 automatic relief | flag only | the conditions from the statute; "check the RAP sheet for a relief granted note" |
| Everything else (1203.4b fire camp, 1203.42, 17(b), Proposition 47, Proposition 64, 851.91 and 851.93 arrest sealing, certificates of rehabilitation, trafficking and DV vacatur, early termination of probation) | no | named in the report with a short card and a referral line |

Per-case bands need six facts per conviction (county, year, level, sentence type, probation grant and outcome, completion month). The reference intake form does not collect them; `skills/intake-import/references/intake-flow.md` shows the optional block to add. Until then the import step asks staff for them, the screen runs at client level, or the attorney answers the screen's interactive questions from memory or the court file.

The five band strings are `LIKELY ELIGIBLE`, `NOT ELIGIBLE NOW` (with a recheck date), `NEEDS ATTORNEY REVIEW`, `AUTOMATIC RELIEF MAY APPLY` (a flag on an item) and `NOT SCREENED`. A packet inherits the strictest client-level result: one "I'm not sure" on a pending case makes every band provisional.

## Ethical and confidentiality preconditions

Before using this plugin with real applicants, confirm with the supervising attorney and the organization's IT or ethics lead:

1. **Account tier and data handling.** Which Claude plan the program is on and what its retention and training terms say about client data.
2. **AI-use practice.** Whether and how the program discloses AI-assisted screening to applicants, per ABA Formal Opinion 512 (2024), the state bar's guidance, and Rules of Professional Conduct 1.1, 1.4, 1.6 and 5.3.
3. **RAP sheets and intake data.** RAP sheets, court records and identifiers never enter a session. The import step drops names, contact details, dates of birth, Social Security and CII numbers, registry dates and attachments, and keeps initials or a clinic ID.
4. **Heightened sensitivity.** Criminal records, immigration exposure, registration status, and trafficking or domestic violence flags carry heightened confidentiality expectations. Decide whether any of these require extra safeguards or exclusion from the plugin.

The cold-start interview captures these decisions as Part 0 before any other configuration.

## Confidence markers

- `[AI-ASSISTED DRAFT — requires attorney review before any client communication]` on every output.
- `[review]` on a judgment call the attorney has to make; `[verify]` on a fact to confirm against a primary source.
- `[model calculation — verify]` on every date the skill computed, with the arithmetic beside it.
- `[statute / regulator site]` and `[settled — last confirmed YYYY-MM-DD]` on every statutory rule, pointing at a dated fetch of the text; `[model knowledge — verify]` on anything else.
- `[partial text, verify]` on cards whose statute text could only be fetched in summary.

Trust the flags more than the absence of flags.

## Built-in safeguards

- **Jurisdiction hard stop.** If the profile's state is not California, every screen stops and explains how to add a state card set. It never applies California rules by default.
- **Currency watch.** `references/currency-watch.md` carries the last-verified date and the last amendment seen for every statute the plugin cites; past 90 days the skills announce staleness.
- **Role gate.** Only the supervising attorney runs setup. Staff and volunteers run import and screen under that attorney's review model.
- **Identifier drop.** The normalized record carries no name, contact detail, date of birth or document; the screen refuses a record that does.
- **Routes, does not decide.** Bands are routing for the attorney. No band is a statement to an applicant.

## Review model

The supervising attorney chooses at setup: a formal review queue (every packet is marked QUEUED and goes to the clinic's own queue), configurable flags (packets that hit a trigger carry CHECK WITH [ATTORNEY] BEFORE ACTING), or lighter-touch (labels and verification prompts only). Changeable later with `/record-clearance-legal:customize`.

## Connectors

Ships with Slack and Google Drive (the suite baseline), CourtListener for citation verification, and two intake connectors new to the suite: **Typeform** (`https://api.typeform.com/mcp`) and **Airtable** (`https://mcp.airtable.com/mcp`), both OAuth and both used read-only here. Nothing requires them: paste and CSV export cover every workflow. Without a research connector, every cite carries `[model knowledge — verify]` or points at the dated card.

## How it learns

The practice profile at `~/.claude/plugins/config/claude-for-legal/record-clearance-legal/CLAUDE.md` is written by the cold-start interview and survives plugin updates. Edit it directly for small fixes, run `/record-clearance-legal:customize` for guided changes, or re-run setup when the program changes. A verification log next to it records every rule a person has checked against a primary source, so the next person does not re-verify.

## Adding a relief type or a state

Copy `skills/eligibility-screen/references/relief/_template.md`, write the card from the current statute text with a dated fetch, add a row to `references/currency-watch.md`, add routing rules to `screening-bands.md`, turn the row on in the profile, and extend `references/sample-intakes/EXPECTED.md` with a fixture. The screen refuses to run for a state with no cards; that refusal is the feature.

## File structure

```
record-clearance-legal/
├── .claude-plugin/plugin.json
├── .mcp.json                              # connectors (see Connectors)
├── CLAUDE.md                              # practice-profile template, written by cold-start
├── README.md
├── hooks/hooks.json                       # empty stub
├── references/
│   ├── currency-watch.md                  # statutes, last amendments, last-verified date
│   ├── plain-language.md                  # staff explanations at a sixth-grade level
│   └── sample-intakes/                    # six synthetic fixtures and EXPECTED.md (the test suite)
└── skills/
    ├── cold-start-interview/SKILL.md
    ├── customize/SKILL.md
    ├── intake-import/
    │   ├── SKILL.md
    │   └── references/intake-schema.md, intake-flow.md
    └── eligibility-screen/
        ├── SKILL.md
        └── references/
            ├── screening-bands.md         # the rulebook
            ├── baseline-questions.md      # the interactive interview and route names
            ├── report-template.md
            └── relief/pc-1203-4.md, pc-1203-4a.md, pc-1203-41.md, not-screened.md, _template.md
```

## Sources and attribution

Eligibility criteria are written from the current text of the California Penal Code, fetched and dated in `references/currency-watch.md` and on each card. Topic coverage of the plain-language material was informed by public legal information guides, including Root & Rebound's Roadmap to Reentry, and rewritten in fresh words; the statute text controls wherever they differ. The intake schema and reference flow are adapted from The Access Project's clean slate intake, with program-specific gates moved into configuration. Sample intakes are synthetic.

## Maintainer

The Access Project (accessprojectca.org), a California nonprofit that runs the Clean Slate Engine, a full RAP sheet analysis platform. This plugin is independent of that platform: it screens from intake answers, and names a program's own full-analysis provider, whoever that is, as the referral target.

> **Disclaimer:** Every output from this plugin is a draft for attorney review, not legal advice, not a legal conclusion, not a substitute for a lawyer. The attorney using the plugin, not the plugin and not its maintainers, is responsible for the legal positions taken in their work product.
