---
name: customize
description: >
  Guided customization of the record clearance practice profile: change one
  thing without re-running the whole cold-start interview. Adjust the
  organization, supervising attorney, program gates, jurisdiction, relief
  types enabled, review model, intake source mapping, referral targets,
  plain-language standards, or integrations. Use when the user says "change
  my [thing]", "add a gate", "switch the review model", "update the referral
  list", or "customize".
argument-hint: "[section name, or describe what you want to change]"
---

# /customize

## When this runs

The user typed `/record-clearance-legal:customize`. They want to change something in the practice profile without re-running setup and without hand-editing the file.

## What to do

1. **Read the config.** Read `~/.claude/plugins/config/claude-for-legal/record-clearance-legal/CLAUDE.md` (and `~/.claude/plugins/config/claude-for-legal/company-profile.md`). If the profile does not exist or still contains `[PLACEHOLDER]`, say:

   > You haven't run setup yet. Run `/record-clearance-legal:cold-start-interview` first; customize is for adjusting a profile you already have.

2. **Check who is asking.** Read `## Who's using this`. Changes to the role, the supervising attorney, the review model, the ethical preconditions, or the identifier policy are the supervising attorney's to make. If the user is staff or a volunteer, make the change only after they confirm the attorney asked for it, and note that in the profile next to the change.

3. **Show the customizable map,** with the current value of each in one line:

   - **Who's using this**: role, supervising attorney, ethical preconditions
   - **Available integrations**: Typeform, Airtable, Google Drive, Slack, CourtListener status (re-probe with `/record-clearance-legal:cold-start-interview --check-integrations`)
   - **Organization profile**: name, type, program name, population, languages, partners
   - **Program eligibility gates**: the gates and the "when a gate fails" rule
   - **Jurisdiction**: state, counties, local practice notes
   - **Relief types enabled**: which rows are screened, flag only, or referred, and to whom
   - **Review model**: formal queue, flags, lighter-touch; triggers; queue location
   - **Intake source mapping**: source, IDs, field-map overrides, identifier policy
   - **Referral targets**: full analysis, county list, out-of-county guidance, RAP sheet page
   - **Plain-language standards**: reading level, jargon, required elements

4. **Ask what they want to change.**

   > What would you like to adjust? Pick a section, or describe the change in your own words.

5. **Make the change.** Show the current value, ask for the new value, explain what changes downstream, confirm, write it. Examples:

   - *Turning a relief type to "screened":* "I can only mark a row screened when a card exists under `skills/eligibility-screen/references/relief/` and a row exists in `references/currency-watch.md`. There is no card for [type] yet. Want to keep it as a referral, or add a card first using `_template.md`?"
   - *Changing the state:* "Version 0.2 screens California only. If I set the state to [state], every screen will stop with instructions for adding a state card set. Do that?"
   - *Review model formal queue → flags:* "Screens will carry CHECK WITH [attorney] BEFORE ACTING when a trigger fires instead of QUEUED. Which triggers?"
   - *Adding a gate:* "The import step will record it; the screen will report it against the rule you give me, separately from the legal bands."

6. **Close.**

   > Done. The next import or screen will reflect the change. You can run `/record-clearance-legal:customize` anytime.

## Guardrails

- **Never delete a section.** Offer to mark a gate or a referral `[Archived]` instead.
- **Flag internal inconsistency.** A screened relief type with no card; a formal queue with no queue location; a non-CA state with screened rows; an identifier policy of "full name" (refuse that one: the schema never carries names).
- **Flag guardrail degradation.** These are load-bearing and are not removed through customize: the AI-assisted header on every output, the supervising-attorney role gate, the California hard stop, the "routes, does not decide" framing on the screen, the identifier drop at import. Explain the trade-off and, if the user is not the attorney, ask them to raise it with the attorney.
- **One change at a time.** Don't re-ask the whole interview.
