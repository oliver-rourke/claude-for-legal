# [CODE] [SECTION]: [short name of the relief]

**Screened in v[version]: [yes | no | flag only].** Record fields used: [list the `intake-schema.md` fields the rules read].

## What it is

[One paragraph. What the court or agency does, in the statute's own words where precision matters, with subdivision pinpoints.]

## Who may qualify

- [Threshold 1 with subdivision pinpoint]
- [Waiting period, if any, with the exact triggering event the statute names]
- [Status requirements: supervision, pending charges]

## Who is excluded

- [Each exclusion with its pinpoint]

## What the relief does and does not do

- [Effect on later prosecutions]
- [Disclosure duties that survive]
- [Firearms, public office, protective orders, licensing]

## Procedure notes

- [Notice periods, who may apply, forms the statute itself mentions]

## Facts the screen needs

[The record fields, and what staff must confirm when a form answer is ambiguous.]

## Plain-language explanation

[Sixth-grade reading level. What it is, who it helps, what the wait is, what it does not do. Fresh wording.]

## Sources

- [Code and section], text fetched from [URL] on [YYYY-MM-DD] `[statute / regulator site]` `[settled — last confirmed YYYY-MM-DD]`.
- Amendment line as displayed: "[the line]".
- [Any secondary guide used for topic coverage, with the note that the statute text controls.]

---

## How to add a relief type or a state

1. Copy this file. Name it `[code]-[section].md` for the same state, or put it under a state folder (`relief/NV/[code]-[section].md`) for a new state.
2. Fetch the current statute text from the legislature's own site and date the fetch. Do not write a card from memory.
3. Fill every heading. Leave nothing in brackets.
4. Add a row to `references/currency-watch.md` with the amendment line and the verify-at link.
5. For a new state, add the state to the profile's `## Jurisdiction` section and update `screening-bands.md` with that state's routing rules. Until a state has at least one card and a bands section, the screen refuses to run for it; that refusal is deliberate.
6. Turn the row in the profile's `## Relief types enabled` to "yes".
7. Re-run `/record-clearance-legal:eligibility-screen` on the sample intakes and compare with `references/sample-intakes/EXPECTED.md`; extend EXPECTED.md with a fixture for the new card.
