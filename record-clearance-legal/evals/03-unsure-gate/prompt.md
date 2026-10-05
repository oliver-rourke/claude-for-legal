---
max_turns: 12
allowed_tools: [Read, Glob, Grep, Skill]
---

No need to confirm the facts back to me; just screen.

Screen this record clearance intake.

```yaml
intake:
  record_id: "synthetic-03"
  source: typeform
  received: 2026-10-04
  program_gates:
    resident_of_service_area: yes
    conviction_in_service_area: yes
    prior_representation: yes
  person:
    reference: "J.L."
    preferred_language: en
  status:
    pending_case: unsure
    current_supervision: none
    supervision_ended_within_3_years: mandatory_supervision
    supervision_ended_bucket: under_1_year
    months_since_supervision_ended: 8
    other_convictions: yes
    fire_camp: no
    fire_camp_year: unknown
    registration_290: no
    trafficking_or_dv_remedies: no
    has_rap_sheet: yes
  cases:
    - id: C1
      county: "Sample"
      conviction_year: 2019
      offense_level: felony
      code_section: ""
      sentence: probation_only
      probation_granted: yes
      probation_outcome: completed_with_violation
      revocation_custody: none
      judgment_date: unknown
      new_conviction_since: unknown
      sentence_completed: 2022-01
      notes: "probation revoked once in 2020, then reinstated and completed"
    - id: C2
      county: "Sample"
      conviction_year: 2022
      offense_level: felony
      code_section: ""
      sentence: split_mandatory_supervision
      probation_granted: no
      probation_outcome: not_applicable
      sentence_completed: 2026-02
      notes: "applicant called it community supervision"
  arrests_without_conviction: []
  consents:
    clean_slate_definition: yes
    representation_scope: yes
    expungement_limits: yes
    record_sharing: yes
    data_use: yes
```
