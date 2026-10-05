---
max_turns: 12
allowed_tools: [Read, Glob, Grep, Skill]
---

No need to confirm the facts back to me; just screen.

Screen this record clearance intake and give me the attorney review screen.

```yaml
intake:
  record_id: "synthetic-01"
  source: pasted
  received: 2026-10-04
  program_gates:
    resident_of_service_area: yes
    conviction_in_service_area: yes
    prior_representation: not_required
  person:
    reference: "D.R."
    preferred_language: en
  status:
    pending_case: no
    current_supervision: none
    supervision_ended_within_3_years: none
    supervision_ended_bucket: unknown
    months_since_supervision_ended: unknown
    other_convictions: unknown
    fire_camp: no
    fire_camp_year: unknown
    registration_290: no
    trafficking_or_dv_remedies: no
    has_rap_sheet: yes
  cases:
    - id: C1
      county: "Sample"
      conviction_year: 2017
      offense_type: misdemeanor
      code_section: "PC 484(a)"
      sentence: probation_only
      probation_granted: yes
      probation_outcome: completed
      sentence_completed: 2019-06
      notes: "no violations reported"
  arrests_without_conviction: []
  consents:
    clean_slate_definition: yes
    representation_scope: yes
    expungement_limits: yes
    record_sharing: yes
    data_use: yes
```
