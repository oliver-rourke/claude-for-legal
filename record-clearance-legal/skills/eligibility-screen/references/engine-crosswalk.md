# Crosswalk to The Access Project's engine labels

Optional. Load this only when someone asks which engine label a route corresponds to. Programs that use the Clean Slate Engine will recognize these labels; everyone else can ignore this file.

| Route (as named in `screening-bands.md` section 2a) | Engine label |
|---|---|
| PC 1203.4 mandatory route: probation fulfilled | 2a |
| PC 1203.4 mandatory route: early discharge | 2b |
| PC 1203.4 discretionary route: interest of justice | 2c |
| PC 1203.4a mandatory route | 3a |
| PC 1203.4a discretionary route | 3b |
| PC 1203.41 route: split sentence | 5a |
| PC 1203.41 route: straight county jail | 5b |
| PC 1203.41 route: state prison | 5c |
| PC 1203.42 pre-realignment prison (not screened; referral line only) | 6 |

## What was not carried over from the engine formulas, and why

- **Code-section lists** (always discretionary, never eligible, realignment-with-prison, wobbler, Prop 47, Prop 64). The statute's own exclusion lists are used instead; everything else is "full analysis decides."
- **Prop 47 and Prop 64 pre-emption of 1203.41.** Program policy, not statute; the report asks full analysis to decide.
- **Registration-date heuristics.** Replaced by the client-level registration question and the (a)(6) flag.
- **Engine date cutoffs.** Replaced by the statutory October 1, 2011 realignment date `[PC 1170(h)(7), verify at the leginfo link]` and the one-year and two-year periods in 1203.41(a)(2).
- **"Felony previously reduced" and Prop 47 or 64 eligibility as 1203.4a entry points.** Kept only as the "felony later reduced" option, tagged verify.
