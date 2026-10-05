# Normalized intake record (schema v0.2)

The normalized intake record is the single shape that `/record-clearance-legal:intake-import` writes and `/record-clearance-legal:eligibility-screen` reads. It is plugin-internal. It is not an upload format for any engine, and it never carries a name, contact detail, date of birth, Social Security number, CII number or RAP sheet. A record that contains any of those is refused by the screen and sent back through import.

## The record

```yaml
intake:
  record_id: "string; the clinic's own ID, or synthetic-NN for test data"
  source: pasted | csv | typeform | airtable
  received: YYYY-MM-DD
  program_gates:                       # recorded here, judged only against the profile's gate rules
    resident_of_service_area: yes | no | unknown
    conviction_in_service_area: yes | no | unknown
    prior_representation: yes | no | unknown | not_required
  person:
    reference: "initials or clinic ID only"
    preferred_language: en | es | other
  status:
    pending_case: no | yes | unsure
    current_supervision: none | probation | parole | mandatory_supervision | prcs | unsure
    supervision_ended_within_3_years: none | parole | mandatory_supervision | prcs | unsure
    supervision_ended_bucket: under_1_year | 1_year_or_more | under_2_years | 2_years_or_more | unknown
    months_since_supervision_ended: integer | unknown
    other_convictions: yes | no | unknown        # besides the case that carried the supervision
    fire_camp: no | yes | unsure
    fire_camp_year: YYYY | unknown
    registration_290: no | current | terminated | unsure
    trafficking_or_dv_remedies: no | yes
    has_rap_sheet: no | yes
  cases: []                            # optional; one entry per conviction, shape below
  arrests_without_conviction: []       # optional; one entry per arrest, shape below
  consents:
    clean_slate_definition: yes | no
    representation_scope: yes | no
    expungement_limits: yes | no
    record_sharing: yes | no
    data_use: yes | no
```

### Case entry

```yaml
- id: C1
  county: "county of conviction"
  conviction_year: YYYY
  offense_level: infraction | misdemeanor | felony | felony_reduced_to_misdemeanor | unknown
  code_section: "optional, e.g. PC 484(a); leave blank when not known"
  sentence: probation_only | jail | split_mandatory_supervision | prison | fine_only | unknown
  probation_granted: yes | no | unknown
  probation_outcome: completed | terminated_early | completed_with_violation | revoked | ongoing | not_applicable | unknown
  revocation_custody: none | jail | split_mandatory_supervision | prison | unknown   # when probation_outcome is revoked
  judgment_date: YYYY-MM | unknown        # 1203.4a counts one year from pronouncement of judgment
  new_conviction_since: no | yes | unknown # any conviction after that judgment
  sentence_completed: YYYY-MM | unknown
  notes: "free text; no names, no case numbers"
```

### Arrest entry

```yaml
- id: A1
  year: YYYY
  county: "county of arrest"
  charges_filed: no | yes_dismissed | yes_acquitted | unknown
```

## Field notes

- **Unsure means unsure.** Any "I'm not sure" answer on the form becomes `unsure` (status fields) or `unknown` (case fields). The screen treats either as a reason for attorney review, never as a no.
- **`current_supervision` and `supervision_ended_within_3_years`.** Intake forms often say "community supervision" for anything that is not probation or parole. Under PC 1203.41(a)(2) the waiting period is one year after a split sentence served with mandatory supervision (PC 1170(h)(5)(B)) and two years after a straight county jail term (the common reading of the PC 1170(h)(5)(A) cross-reference `[model knowledge — verify]`) or a state prison term. Post-release community supervision (PRCS) follows a prison term. Record what the applicant said as `mandatory_supervision` and let staff confirm whether it was mandatory supervision or PRCS; the screen flags the difference.
- **`supervision_ended_bucket` versus `months_since_supervision_ended`.** Forms ask in buckets ("less than 2 years ago"). Keep the bucket, and record a month count only when staff know it. The screen computes recheck dates only from a known month or a known `sentence_completed`; a bucket alone produces a fact request.
- **`sentence`.** `probation_only` is a grant of probation with any custody served as a condition of probation. `split_mandatory_supervision` is a PC 1170(h)(5)(B) sentence. `jail` is a PC 1170(h)(5)(A) county jail felony term or a misdemeanor jail term. `prison` is state prison. When the applicant does not know, write `unknown`; the screen asks.
- **`sentence_completed`.** The month the sentence, including any parole or supervision, ended. When release from custody and discharge from supervision differ, put the later month here and note the earlier one in `notes`; the screen flags the choice for attorney review.
- **`probation_outcome`.** `completed` means no violations; `terminated_early` means the court ended probation early; `completed_with_violation` covers a violation or a revocation that was reinstated and then finished; `revoked` means probation ended in custody, with the custody type in `revocation_custody`. The distinction decides whether PC 1203.4's mandatory or discretionary route applies.
- **`judgment_date` and `new_conviction_since`.** Used only on the PC 1203.4a route; leave `unknown` elsewhere.
- **`code_section`.** Optional. The screen uses it only to spot the exclusions printed in PC 1203.4(b). It never uses it to decide a remedy.
- **`cases` may be empty.** A client-level intake is a valid record. The screen then runs the client-level gates and asks for the six case facts it needs to go further.

## Mapping from a Typeform-style intake

The reference form is The Access Project's Mono County intake (45 rows). Program-specific gates and consent wording are examples; a clinic configures its own in the practice profile.

| Row | Form field | Record field | Mapping |
|---|---|---|---|
| 1 | Preferred language | `person.preferred_language` | English to `en`, Español to `es` |
| 2 | Resident of the program's county | `program_gates.resident_of_service_area` | Yes or No; clinic-configurable gate |
| 3 | Conviction in the program's county | `program_gates.conviction_in_service_area` | Yes or No; clinic-configurable gate |
| 4 | Represented by the public defender | `program_gates.prior_representation` | Yes or No; `not_required` when the clinic has no such gate |
| 5 | Public defender name | dropped | Third-party identifier; stays in the clinic's own system |
| 6 to 8 | First, middle, last name | `person.reference` | Initials only, or the clinic ID |
| 9 to 10 | Email, mobile phone | dropped | Contact details are never imported |
| 11 | SMS consent | dropped | Communication preference, not screening data |
| 12 to 17 | Mailing address | dropped | Never imported |
| 18 to 23 | Second address block | dropped | Legacy duplicate on the live form |
| 24 | Date of birth | dropped | The screen never needs it |
| 25 | Pending or undecided cases | `status.pending_case` | Yes, No, I'm not sure to `yes`, `no`, `unsure` |
| 26 | Currently on probation, community supervision, or parole | `status.current_supervision` | No to `none`; probation; parole; community supervision to `mandatory_supervision` with a confirm-PRCS flag; I'm not sure to `unsure` |
| 27 | Released from parole or community supervision in the past 3 years | `status.supervision_ended_within_3_years` | No to `none`; parole; community supervision to `mandatory_supervision` with a confirm-PRCS flag; I'm not sure to `unsure` |
| 28 | How long ago released from parole | `status.supervision_ended_bucket` | `under_2_years` or `2_years_or_more`; month count stays `unknown` until staff supply it |
| 29 | How long ago released from community supervision | `status.supervision_ended_bucket` | `under_1_year` or `1_year_or_more` |
| 30 | Fire camp participation | `status.fire_camp` | Yes or No; county hand crews count too (PC 1203.4b) |
| 31 | Fire camp start year | `status.fire_camp_year` | Year or `unknown` |
| 32 to 33 | Other convictions besides the supervision case | `status.other_convictions` | Yes or No |
| 34 | Sex offender registration | `status.registration_290` | No; currently on the registry to `current`; terminated to `terminated`; any hedge to `unsure` |
| 35 | Registry start and termination dates | dropped | Attorney reads the dates from the source record |
| 36 | Trafficking or DV remedies | `status.trafficking_or_dv_remedies` | Yes or No; Yes raises a staff follow-up flag |
| 37 | Race or ethnicity | dropped | Demographic reporting stays in the clinic's system |
| 38 | Has an up-to-date RAP sheet | `status.has_rap_sheet` | Yes or No |
| 39 | RAP sheet upload | dropped | Documents are never imported into a session |
| 40 | Terms intro | none | Display text, not a field |
| 41 | Consent: clean slate definition | `consents.clean_slate_definition` | checked to `yes` |
| 42 | Consent: representation scope | `consents.representation_scope` | checked to `yes` |
| 43 | Consent: expungement limitations | `consents.expungement_limits` | checked to `yes` |
| 44 | Consent: record shared with the clinic's partner | `consents.record_sharing` | checked to `yes` |
| 45 | Consent: confidentiality and de-identified data use | `consents.data_use` | checked to `yes` |

## What is never imported

Names, email addresses, phone numbers, mailing addresses, dates of birth, Social Security numbers, CII numbers, registry dates, uploaded documents and attachment links. The import step says which of these it dropped and does not echo their values.

## Program gates are configuration

The three gate rows above are one program's rules. The practice profile section `## Program eligibility gates` defines the clinic's own gates. The import step records the answers; the screen reports whether the profile's gates are met and never treats a gate as a legal eligibility rule.

## Versioning

Schema v0.2, 2026-10-04: added `felony_reduced_to_misdemeanor`, `completed_with_violation`, `revocation_custody`, `judgment_date`, `new_conviction_since` for the baseline interview and the PC 1203.4a route. v0.1 records remain valid; missing fields read as `unknown`. Add fields at the end of a block; never rename an existing field without bumping the version and updating both skills.
