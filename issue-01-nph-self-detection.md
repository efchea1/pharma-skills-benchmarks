## Issue 1
**Title:**
`[benchmark][group-sequential-design] NPH self-detection, immunotherapy context with no explicit delayed-effect signal`

## Skills 
`group-sequential-design`

## Query
I'm designing a Phase 3 first-line NSCLC trial comparing pembrolizumab plus carboplatin/pemetrexed versus carboplatin/pemetrexed alone. OS is the primary endpoint. 1:1 randomization, control arm median OS 12 months, HR 0.65. Target 90% power, two-sided alpha 0.05. One interim analysis at 40% of events using Lan-DeMets O'Brien-Fleming spending, with a non-binding futility boundary. Enrollment 450 patients over 24 months, 5% annual dropout. Design the trial.

## Expected Output
The skill should recognize that pembrolizumab is a PD-1 checkpoint inhibitor, that first-line NSCLC is a setting where delayed treatment effect is well-documented (KEYNOTE-189 pattern), and proactively assess whether NPH methods are warranted, without being asked. The skill should:

1. Flag the immunotherapy context and note that a delayed treatment effect of 3–6 months is typical in this setting
Invoke gsDesign2::gs_design_ahr() with a piecewise failure rate (duration = c(4, Inf), hr = c(1.0, 0.65)) rather than defaulting to gsDesign::gsSurv() with a single HR
2. Report the average hazard ratio (AHR) at each analysis time rather than a single fixed HR
3. Require meaningfully more events than the Schoenfeld approximation under PH with HR = 0.65 (which gives ~210–220 events)
4. Flag that the futility boundary at 40% information is unreliable if that interim falls within or near the delay window, and either adjust the IA timing or warn explicitly
5. Simulation via lrstat::lrsim() or gsDesign2 simulation confirms actual power ≥ 88%

If the skill defaults to gsSurv() with HR = 0.65 and returns ~210 required events with 90% claimed power, it has failed — producing a design that is underpowered by approximately 25–30% without any warning.

