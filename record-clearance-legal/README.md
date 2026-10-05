# Claude for Record Clearance

*Statute-level eligibility screening for record clearance programs, with the attorney in the loop by design.*

A plugin for the people who run record clearance (expungement) programs: legal aid offices, public defender clean slate programs, law school clinics, pro bono projects and reentry nonprofits. Staff paste an intake, or an attorney describes a conviction from memory, and the plugin returns a short screen against California's dismissal statutes: what looks eligible, what is not eligible yet and when to look again, and what needs a lawyer's judgment before anyone acts.

**Every output is a draft for attorney review, marked, gated and logged. The plugin routes; it does not decide. Nothing goes to an applicant without the review model the supervising attorney set at setup.**

## The problem this solves

The first hour with every applicant goes to the same questions: is there anything here worth an attorney's time, and what is missing before the attorney can say so? Staff answer them from memory of a dozen statutes with waiting periods that changed last year, and the answers drift. Meanwhile the waitlist grows.

This plugin pins every answer to a subdivision of the statute, flags every uncertainty where the attorney will see it, and refuses to guess. The screen runs on client-level answers alone when that is all there is, and gets more precise as case facts arrive. It does not need, and does not read, a RAP sheet.

## Who uses it

| Role | Runs | Gets |
|---|---|---|
| **Supervising attorney** | `/record-clearance-legal:cold-start-interview` once (two minutes quick, ten full), then reviews every screen | A program profile every skill reads; screens with the judgment calls already flagged |
| **Staff and volunteers** | `/record-clearance-legal:eligibility-screen`, one command | A short screen with bands, next steps and the facts still needed; identifiers dropped on the way in |

## Commands

| Command | What it does | What it doesn't do |
|---|---|---|
| `/record-clearance-legal:eligibility-screen` | **Check 2**, with check 1 run inline on a paste. Takes pasted intake answers, a RAP sheet excerpt with identifiers removed, a normalized record, or a sentence or two about a conviction; with nothing, asks four to eight questions per conviction from memory. Drops names and contact details and says so. Before screening, lists the facts it will rely on as numbered yes-or-no lines and waits for a yes or a correction ("just screen" skips it). Screens against PC 1203.4, PC 1203.4a and PC 1203.41; names the route (mandatory, discretionary, split sentence, jail, prison) and whether a declaration is usually needed; bands each conviction; flags where PC 1203.425 automatic relief may already have acted; computes recheck dates with the arithmetic shown; for every item on a petition route, lists the county's forms and how the prosecutor is served. About 300 words for a clean one-case screen; `--full` gives the long form | Doesn't decide; doesn't read RAP sheets or documents; doesn't evaluate code-section lists, Proposition 47, Proposition 64 or PC 17(b); doesn't draft petitions; doesn't run outside California |
| `/record-clearance-legal:intake-import` | **Check 1.** Imports an intake (paste, CSV row, Typeform or Airtable record) with identifiers dropped, then triages it with the routing a mature intake form carries: proceed to the screen, needs attorney review, not eligible now with a recheck, or not eligible for the program (gates, prior review, consents) | Doesn't band a conviction; doesn't ask for case facts (the screen does); doesn't write back to any form or base |
| `/record-clearance-legal:cold-start-interview` | **Attorney.** One-time setup: role gate, jurisdiction, review model, intake source; the full path adds the ethical-preconditions record, program gates, referral targets and plain-language standards | Doesn't configure a screen for any state other than California in this version |
| `/record-clearance-legal:customize` | Change one profile section without re-running setup | Doesn't remove the load-bearing guardrails |

## Two checks

1. **Intake triage.** From the intake answers alone: is this applicant worth an attorney's time now? Program gates, any prior review decision, pending case, supervision today and in the past three years with the fire camp exception, registration, trafficking or DV, and the five consents, in the order a mature intake form routes them. Outcomes: proceed to the screen, needs attorney review (a "not sure" on a field that matters), not eligible now (a wait, with the recheck), or not eligible for the program (with the referral step). Most intake forms collect these answers; this supplies the routing for any source.
2. **Eligibility screen.** Per conviction, under PC 1203.4, 1203.4a and 1203.41, with the automatic relief flag, the recheck arithmetic and the county filing bullet. It runs when check 1 says proceed or review, and on request otherwise.

## What it screens

| Relief | Screened | How |
|---|---|---|
| PC 1203.4 dismissal after probation | yes | mandatory routes (probation fulfilled, early discharge), discretionary route, exclusions in subdivisions (b) and (c), 15-day notice, restitution rule |
| PC 1203.4a dismissal for a misdemeanor without probation, or an infraction | yes | one year from judgment, sentence complied with, new-conviction proxy for "honest and upright life", exclusions in subdivision (d) |
| PC 1203.41 dismissal after a felony jail, split or prison sentence | yes | one-year and two-year waiting periods, supervision bar, registration bar |
| PC 1203.425 automatic relief | flag only | the conditions from the statute; "check the RAP sheet for a relief granted note" |
| Everything else (1203.4b fire camp, 1203.42, 17(b), Proposition 47, Proposition 64, 851.91 and 851.93 arrest sealing, certificates of rehabilitation, trafficking and DV vacatur, early termination of probation) | no | named in the report only when the facts raise it, with a short card and a referral line |

Per-case bands need six facts per conviction: county, year, offense type, sentence type and term, probation grant and outcome, sentencing month; when the actual end date is unknown the screen estimates it from the term and says so. Most intake forms do not collect them, so the screen asks for what the paste left out, one or two questions at a time, and never for a document. With no case facts at all it runs the client-level gates and asks for the six.

The five band strings are `LIKELY ELIGIBLE`, `NOT ELIGIBLE NOW` (with a recheck date), `NEEDS ATTORNEY REVIEW`, `AUTOMATIC RELIEF MAY APPLY` (a flag on an item) and `NOT SCREENED`. The overall line inherits the strictest client-level result: one "I'm not sure" on a pending case makes every band provisional.

## What a screen looks like

Short by default. First a verification checklist: the facts the screen will rely on, numbered, for a yes or a correction. Then the screen: the header, a one-line reviewer note, the check 1 triage line and the check 2 band line, one block per conviction (band, route, the facts that drove it, the flags), the next-step bullets, and a one-line decision tree. Empty sections are left out. `--full` gives the long form with the gate table, the per-item table and the five-option decision tree, for the attorney memo. `skills/eligibility-screen/references/report-template.md` has both forms and a rendered example.

## Filing guidance

For every conviction on a petition route, Next steps carries one "Filing in [county]" bullet: the county's petition and order forms (CR-180 and CR-181, or the county's own versions), the proof-of-service form, who is served and how (the district attorney by mail, email or fax, sometimes probation too), the declaration form when relief is discretionary, and one local point such as a hearing date required before filing or a wet-signature rule. Picking "filing checklist" in the decision tree prints the county's full entry: filing mode, copies, e-filing and case-lookup notes, courthouses, and the court and prosecutor contacts recorded for service. The data is `skills/eligibility-screen/references/county-filing-guide.md`, built from The Access Project's county filing tables for all 58 California counties and dated; every line carries a verify-with-the-court tag because local practice changes. This is guidance about what to file. The plugin does not fill, generate or file petitions.

## Ethical and confidentiality preconditions

Before using this plugin with real applicants, confirm with the supervising attorney and the organization's IT or ethics lead:

1. **Account tier and data handling.** Which Claude plan the program is on and what its retention and training terms say about client data.
2. **AI-use practice.** Whether and how the program discloses AI-assisted screening to applicants, per ABA Formal Opinion 512 (2024), the state bar's guidance, and Rules of Professional Conduct 1.1, 1.4, 1.6 and 5.3.
3. **RAP sheets and intake data.** Full RAP sheets and court records never enter a session; a short excerpt of conviction lines with identifiers removed may be pasted as case facts. Both the screen and the import step drop names, contact details, dates of birth, Social Security, CII and case numbers, registry dates and attachments, and keep initials or a clinic ID.
4. **Heightened sensitivity.** Criminal records, immigration exposure, registration status, and trafficking or domestic violence flags carry heightened confidentiality expectations. Decide whether any of these require extra safeguards or exclusion from the plugin.

The full setup records these decisions as Part 0. The quick start asks one yes or no and writes a default that tells staff not to use the plugin on real applicants until the attorney confirms them.

## Confidence markers

- `[AI-ASSISTED DRAFT — requires attorney review before any client communication]` on every output.
- `[review]` on a judgment call the attorney has to make; `[verify]` on a fact to confirm against a primary source.
- `[model calculation — verify]` on every date the skill computed, with the arithmetic beside it.
- `[statute / regulator site]` on every rule quoted from a dated card, with the cards' last-confirmed date stated once in the reviewer note; `[model knowledge — verify]` on anything else.
- `[partial text, verify]` on cards whose statute text could only be fetched in summary.

Trust the flags more than the absence of flags.

## Built-in safeguards

- **Jurisdiction hard stop.** If the profile's state is not California, every screen stops and explains how to add a state card set. It never applies California rules by default.
- **Currency watch.** `references/currency-watch.md` carries the last-verified date and the last amendment seen for every statute the plugin cites. It is current until the January 1 after that date, when California statutes take effect, or earlier if an entry names a pending change; a clinic's verification-log entry refreshes it.
- **Role gate.** Only the supervising attorney runs setup. Staff and volunteers run the screen under that attorney's review model.
- **Identifier drop.** The normalized record carries no name, contact detail, date of birth or document. The screen drops any it finds in a paste, says which categories it dropped, and never echoes them.
- **Routes, does not decide.** Bands are routing for the attorney. No band is a statement to an applicant.

## Review model

The supervising attorney chooses at setup: a formal review queue (every screen is marked QUEUED and goes to the clinic's own queue), configurable flags (screens that hit a trigger carry CHECK WITH [ATTORNEY] BEFORE ACTING), or lighter-touch (labels and verification prompts only). Changeable later with `/record-clearance-legal:customize`.

## Connectors

Ships with Slack and Google Drive (the suite baseline), CourtListener for citation verification, and two intake connectors new to the suite: **Typeform** (`https://api.typeform.com/mcp`) and **Airtable** (`https://mcp.airtable.com/mcp`), both OAuth and both used read-only here. Nothing requires them: paste and CSV export cover every workflow. Without a research connector, every cite comes from the dated cards or carries `[model knowledge — verify]`.

## How it learns

The practice profile at `~/.claude/plugins/config/claude-for-legal/record-clearance-legal/CLAUDE.md` is written by the cold-start interview and survives plugin updates. Edit it directly for small fixes, run `/record-clearance-legal:customize` for guided changes, or re-run setup when the program changes. A verification log next to it records every rule a person has checked against a primary source, so the next person does not re-verify. A section left at its placeholder does not stop the skills; they apply the template's default and say so in the reviewer note. The state is the exception: it must be set.

## Testing

`references/sample-intakes/` holds twelve synthetic fixtures and `EXPECTED.md`, the acceptance tests: exact band strings, the rules each report must mention, the flags it must raise, and the word caps. `evals/` holds the same tests as a `claude plugin eval` suite. From the plugin directory:

```
claude plugin eval . --ablation none --no-publish --max-cost-usd 5
```

Run it before committing a change to a rule, a card or a template. Add a fixture and an expected row for every new rule.

## Adding a relief type or a state

Copy `skills/eligibility-screen/references/relief/_template.md`, write the card from the current statute text with a dated fetch, add a row to `references/currency-watch.md`, add routing rules to `screening-bands.md`, turn the row on in the profile, and extend `references/sample-intakes/EXPECTED.md` and `evals/` with a fixture. The screen refuses to run for a state with no cards; that refusal is the feature.

## File structure

```
record-clearance-legal/
├── .claude-plugin/plugin.json
├── .mcp.json                              # connectors (see Connectors)
├── CLAUDE.md                              # practice-profile template, written by cold-start
├── README.md
├── hooks/hooks.json                       # empty stub
├── evals/                                 # claude plugin eval suite (one case per fixture)
├── references/
│   ├── currency-watch.md                  # statutes, last amendments, last-verified date
│   ├── plain-language.md                  # staff explanations at a sixth-grade level
│   └── sample-intakes/                    # twelve synthetic fixtures and EXPECTED.md
└── skills/
    ├── cold-start-interview/SKILL.md
    ├── customize/SKILL.md
    ├── intake-import/
    │   ├── SKILL.md
    │   └── references/intake-schema.md, intake-flow.md, triage-rules.md
    └── eligibility-screen/
        ├── SKILL.md
        └── references/
            ├── screening-bands.md         # the rulebook: gates, routes, bands, roll-up
            ├── baseline-questions.md      # the interview questions
            ├── report-template.md         # compact and full forms, with an example
            ├── county-filing-guide.md     # forms, service and local practice for all 58 counties
            ├── engine-crosswalk.md        # optional map to The Access Project's engine labels
            └── relief/pc-1203-4.md, pc-1203-4a.md, pc-1203-41.md, not-screened.md, _template.md
```

## Sources and attribution

Eligibility criteria are written from the current text of the California Penal Code, fetched and dated in `references/currency-watch.md` and on each card. Topic coverage of the plain-language material was informed by public legal information guides, including The Access Project's website content and Root & Rebound's Roadmap to Reentry; the statute text controls wherever they differ. The intake schema and reference flow are adapted from The Access Project's clean slate intake, with program-specific gates moved into configuration. Sample intakes are synthetic.

## Maintainer

The Access Project (accessprojectca.org), a California nonprofit that runs the Clean Slate Engine, a full RAP sheet analysis platform. This plugin is independent of that platform: it screens from intake answers, and names a program's own full-analysis provider, whoever that is, as the referral target.

> **Disclaimer:** Every output from this plugin is a draft for attorney review, not legal advice, not a legal conclusion, not a substitute for a lawyer. The attorney using the plugin, not the plugin and not its maintainers, is responsible for the legal positions taken in their work product.
