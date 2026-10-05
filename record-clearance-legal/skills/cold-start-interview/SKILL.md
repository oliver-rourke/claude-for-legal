---
name: cold-start-interview
description: >
  Supervising attorney's one-time setup for a record clearance program: who
  runs the plugin, ethical preconditions, program gates, jurisdiction, relief
  types enabled, review model, intake source and field map, referral targets.
  Writes the practice profile every other skill reads. Use on fresh install,
  when the profile has placeholders, when redoing setup with --redo, or when
  re-checking connectors with --check-integrations.
argument-hint: "[--redo] [--check-integrations] [--full]"
---

# /cold-start-interview

1. Check `~/.claude/plugins/config/claude-for-legal/record-clearance-legal/CLAUDE.md`. If populated and no `--redo`, confirm before overwriting.
2. Run the interview below, starting with Part 0 (supervising-attorney role check, ethical preconditions, what's connected). If the user is not the supervising attorney, stop and redirect.
3. Quick start (2 minutes): organization, supervising attorney, state and counties, review model, intake source. Full setup (10 minutes) adds program gates, field map confirmation, relief-type referral targets, referral links, plain-language standards, seed documents.
4. Migration: if a populated profile (no `[PLACEHOLDER]`) exists at `~/.claude/plugins/cache/claude-for-legal/record-clearance-legal/*/CLAUDE.md` but not at the config path, copy it forward and show what was migrated.
5. Write the profile from the template shipped with the plugin (`CLAUDE.md` in the plugin root), filling every section; mark skipped answers `[DEFAULT]`.
6. Offer a test run on `references/sample-intakes/01-likely-eligible.md`.

```
/record-clearance-legal:cold-start-interview
```

**`--check-integrations`:** re-run only the Part 0 connector check and update `## Available integrations`, leaving every other section alone. Use after adding or removing an MCP connector. Report ✓ only when a tool call actually succeeded this session; configured-but-untested connectors are ⚪ with a one-line how-to; never report ✓ from `.mcp.json` alone.

---

# Cold-Start Interview: Record Clearance Program

## Purpose

Every screen this plugin produces reads the practice profile: who supervises, which program gates apply, which state's statutes govern, which relief types are screened, how review works, where intakes come from, and where to refer what the plugin does not handle. A generic profile gives generic screens, and the first week is spent correcting what the tool assumed. This interview writes the profile once; `/record-clearance-legal:customize` changes one thing later.

**Audience: the supervising attorney.** Staff and volunteers do not run setup. The profile records decisions the attorney is accountable for: the role gate, the review model, and the data-handling preconditions.

## Cold-start check

Read `~/.claude/plugins/config/claude-for-legal/record-clearance-legal/CLAUDE.md`:

- **Does not exist** → start the interview.
- **Contains `<!-- SETUP PAUSED AT: -->`** → greet the user and offer to resume from that section.
- **Contains `[PLACEHOLDER]` but no pause comment** → the template was never completed; offer to start fresh or resume where the placeholders begin.
- **Populated** → already configured; skip unless `--redo`.

## Check for the shared company profile

Look for `~/.claude/plugins/config/claude-for-legal/company-profile.md`. If it exists, read it and confirm in one line ("You're [organization], a [setting] in [jurisdictions]. Right?"). If it does not, ask the organization questions (name, setting, what you do, jurisdictions, regulators, risk posture, escalation names) and write them there per `references/company-profile-template.md` at the marketplace root, then continue with the plugin-specific questions.

## Install scope check

If the working directory is inside a project rather than the home directory, say once that a project-scoped install can only read files in that directory, and ask whether to continue or pause to reinstall user-scoped (see the marketplace QUICKSTART). Skip silently when the working directory is the home directory.

## Before the interview starts

Show this preamble and nothing more:

> **`record-clearance-legal` is for supervising attorneys setting up a record clearance program.** Not your area? `/legal-builder-hub:related-skills-surfacer`.
>
> **2 minutes** gets you the organization, the supervising attorney, your state and counties, the review model, and where intakes come from, with working defaults for everything else. **10 minutes** adds your ethical-preconditions record, program gates, the intake field map, referral targets for the relief types this version does not screen, plain-language standards, and seed documents.
>
> Quick or full? (Upgrade any time with `/record-clearance-legal:cold-start-interview --full`.)

After the choice, orient the attorney in your own voice: what the profile holds, what reads it, that setup builds the profile only from their answers and uploads (not from other conversations or the home-directory CLAUDE.md), and that Part 0 comes first.

## Interview pacing

- **Assume the answer exists somewhere.** For anything longer than a sentence (consent texts, a referral list, an export header), ask for a paste or a link before asking the attorney to type it.
- **Pause for real answers.** Ethical preconditions and the field map need typed answers. Say "this one needs a typed answer, I'll wait," and wait.
- **Two or three answerable prompts per turn,** counting subparts. Prefer tap-through options.
- **Pause and resume.** On "pause" or "stop," write a partial profile with `<!-- SETUP PAUSED AT: [section] -->` at the top and `[PENDING]` on unanswered fields; on re-run, greet and resume without re-asking.
- **Verify user-stated legal facts as they come up.** If the attorney states a waiting period, a statute or a form, sanity-check it against the relief cards before writing it into the profile; surface conflicts with `[premise flagged — verify]`.
- **Before writing,** list every skipped or placeholder answer and ask whether to fill it now. Never write silent gaps.

## The interview

### Part 0: Who's running this setup, ethical preconditions, and what's connected

**Role.**

> Are you the supervising attorney for this program? Setup writes the program's governing context (review model, data-handling rules, ethical preconditions) and must be done by the licensed attorney accountable for the work.
> 1. **Yes, I'm the supervising attorney.** Continue.
> 2. **No, I'm staff, a volunteer, or an administrator.** Stop. Ask the supervising attorney to run `/record-clearance-legal:cold-start-interview`. Staff and volunteers use `/record-clearance-legal:intake-import` and `/record-clearance-legal:eligibility-screen` once setup is done.

If 2, stop. If 1, record name, bar jurisdiction and bar number under `## Who's using this`.

**Ethical and confidentiality preconditions.** Confirm each, and record the answers:

1. **Account tier and data handling.** Which Claude plan the program is on and what its retention and training terms say about client data.
2. **AI-use practice.** Whether and how the program discloses AI-assisted screening to applicants, per ABA Formal Opinion 512 (2024), the state bar's guidance, and Rules of Professional Conduct 1.1, 1.4, 1.6 and 5.3.
3. **RAP sheets and intake data.** RAP sheets, court records and identifiers never enter a session; the import step drops them. Where normalized records may be saved, who sees them, and how long they are kept.
4. **Heightened sensitivity.** Criminal records, immigration exposure, registration status, and trafficking or domestic violence flags carry heightened confidentiality expectations. Confirm whether any of these require extra safeguards or exclusion from the plugin.

If any item is unresolved, flag it in the profile and note that staff should not use the plugin on real applicants until it is resolved.

**What's connected.** For Typeform, Airtable, Google Drive, Slack and CourtListener: ✓ only after a successful call this session; ⚪ configured but not verified, with a one-line how-to; ✗ not found, with the fallback (paste or CSV for intakes; local files for documents; manual copy for Slack; `[model knowledge — verify]` tags for citations). Core features work with paste alone.

### Part 1: Organization and program (quick and full)

- Organization and type: legal aid office, public defender program, law school clinic, pro bono program, reentry nonprofit.
- Program name applicants see; client population; languages beyond English.
- Who does full RAP sheet analysis and petitions: an in-house attorney, a partner organization, or a case analysis engine. This becomes the referral target for everything the screen does not decide.

### Part 2: Jurisdiction (quick and full)

- State. Say plainly: "Version 0.2 ships relief cards for California only. If your state is not CA, setup will complete, but every screen will stop with instructions for adding a state card set until one exists."
- Counties served; local practice notes (service methods, hearing practice, declaration expectations).

### Part 3: Review model (quick and full)

Offer the three models from the profile (formal review queue, configurable flags, lighter-touch) with one sentence each, and ask where the queue lives (a Slack channel, an Airtable view, a folder). There is no right answer; it can change later.

### Part 4: Intake source (quick and full)

- Primary source: Typeform, Airtable, CSV export, or paper and paste. Form or base IDs if any.
- Identifier policy: initials or clinic ID.
- Confirm the default field map in `skills/intake-import/references/intake-schema.md`; capture overrides for fields that differ. Ask for an export header row to check the names.

### Part 5: Program gates (full)

For each of residency, conviction in service area, prior representation, and any other gate: required or not, and the rule in one line. Then "when a gate fails": referral list, stop, or screen anyway for the attorney's information.

### Part 6: Relief types and referral targets (full)

Confirm the two screened types (PC 1203.4, PC 1203.41). For every other row in `## Relief types enabled`, name the referral target. Capture the county referral list link, the out-of-county guidance, and the RAP sheet acquisition page.

### Part 7: Plain-language standards and seed documents (full)

Reading level, prohibited jargon, required elements for any client-facing text drafted after review. Seed documents: consent texts, a scrubbed example intake, the referral list, local practice notes. Fewer than three is a LIMITED DATA profile; say what that means (defaults for gates and referrals).

## Write the profile

Write to `~/.claude/plugins/config/claude-for-legal/record-clearance-legal/CLAUDE.md`, creating directories as needed, using the structure of the template shipped with the plugin. Fill every section. Replace every `[PLACEHOLDER ...]` marker in the template: an answered question gets the answer; a skipped or quick-start question gets `[DEFAULT — <the default value or "not set; add via /record-clearance-legal:customize">]`. Never leave a `[PLACEHOLDER` marker behind, because every other skill stops when it sees one. Before showing the confirmation, search the written file for `[PLACEHOLDER` and fix any hit. Create an empty `verification-log.md` next to it. Then show a short confirmation: role and attorney, state, review model, intake source, gates, and which sections carry defaults.

Quick-start close: "Done. You can run `/record-clearance-legal:intake-import` and `/record-clearance-legal:eligibility-screen` now. I used defaults for gates, referral targets and plain-language standards; when a screen's output feels off, that is usually a default to tune. Run `/record-clearance-legal:cold-start-interview --full` anytime, or `/record-clearance-legal:customize` to change one section."

## Offer a test run

> Want to see a screen end to end? I'll run `/record-clearance-legal:eligibility-screen` on the synthetic intake at `references/sample-intakes/01-likely-eligible.md` under your profile, so you can see the header, the reviewer note, the bands and the decision tree before staff use it on a real applicant.

## What this skill does NOT do

- **Configure a non-California screen.** It records the state and warns; the card set is a separate contribution.
- **Accept setup from staff or volunteers.** The role gate is the first question.
- **Read RAP sheets or client files as seed documents.** Scrubbed examples only.
