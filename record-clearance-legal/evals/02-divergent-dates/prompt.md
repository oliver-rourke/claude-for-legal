---
max_turns: 12
allowed_tools: [Read, Glob, Grep, Skill]
---

No need to confirm the facts back to me; just screen.

Screen this record clearance intake.

```yaml
intake:
  record_id: "synthetic-02"
  source: csv
  received: 2026-10-04
  program_gates:
    resident_of_service_area: yes
    conviction_in_service_area: yes
    prior_representation: not_required
  person:
    reference: "M.T."
    preferred_language: es
  status:
    pending_case: no
    current_supervision: none
    supervision_ended_within_3_years: parole
    supervision_ended_bucket: under_2_years
    months_since_supervision_ended: 14
    other_convictions: yes
    fire_camp: no
    fire_camp_year: unknown
    registration_290: no
    trafficking_or_dv_remedies: no
    has_rap_sheet: no
  cases:
    - id: C1
      county: "Sample"
      conviction_year: 2017
      offense_level: misdemeanor
      code_section: ""
      sentence: probation_only
      probation_granted: yes
      probation_outcome: completed
      sentence_completed: 2019-03
      notes: ""
    - id: C2
      county: "Sample"
      conviction_year: 2021
      offense_level: felony
      code_section: ""
      sentence: prison
      probation_granted: no
      probation_outcome: not_applicable
      sentence_completed: unknown
      notes: "released from prison 2024-02; parole discharged 2025-08"
  arrests_without_conviction:
    - id: A1
      year: 2015
      county: "Sample"
      charges_filed: no
  consents:
    clean_slate_definition: yes
    representation_scope: yes
    expungement_limits: yes
    record_sharing: yes
    data_use: yes
```
