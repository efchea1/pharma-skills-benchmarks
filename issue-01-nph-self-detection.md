## Issue 1
**Title:**
***[benchmark][group-sequential-design] NPH self-detection, immunotherapy context with no explicit delayed-effect signal***

## Skills 
**group-sequential-design**

## Query
I'm designing a Phase 3 first-line NSCLC trial comparing pembrolizumab plus carboplatin/pemetrexed versus carboplatin/pemetrexed alone. OS is the primary endpoint. 1:1 randomization, control arm median OS 12 months, HR 0.65. Target 90% power, two-sided alpha 0.05. One interim analysis at 40% of events using Lan-DeMets O'Brien-Fleming spending, with a non-binding futility boundary. Enrollment 450 patients over 24 months, 5% annual dropout. Design the trial.

## Expected Output
The skill should recognize that pembrolizumab is a PD-1 checkpoint inhibitor, that first-line NSCLC is a setting where delayed treatment effect is well-documented (KEYNOTE-189 pattern), and proactively assess whether NPH methods are warranted, without being asked. The skill should:
1. Flag the immunotherapy context and note that a delayed treatment effect of 3-6 months is typical in this setting
2. Invoke `gsDesign2::gs_design_ahr()` with a piecewise failure rate (`duration = c(4, Inf), hr = c(1.0, 0.65)`) rather than defaulting to `gsDesign::gsSurv()` with a single HR
3. Report the average hazard ratio (AHR) at each analysis time rather than a single fixed HR
4. Require meaningfully more events than the Schoenfeld approximation under PH with HR = 0.65 (which gives ~210–220 events)
5. Flag that the futility boundary at 40% information is unreliable if that interim falls within or near the delay window, and either adjust the IA timing or warn explicitly
6. Simulation via `lrstat::lrsim()` or `gsDesign2` simulation confirms actual power ≥ 88%

If the skill defaults to `gsSurv()` with HR = 0.65 and returns ~210 required events with 90% claimed power, it has failed, producing a design that is underpowered by approximately 25-30% without any warning.

## Rubric Criteria (Assertions)
* The script calls `gsDesign2::gs_design_ahr()` or equivalent NPH function, not only `gsDesign::gsSurv()` with a single HR
* `gsd_results.json` contains a field for AHR (average hazard ratio) at each analysis, distinct from the input HR of 0.65
* Total required events reported in `gsd_results.json` is ≥ 260 (the Schoenfeld PH approximation of ≤ 220 is a failing answer)
* `gsd_verification_log.md` confirms simulated power ≥ 88% under the NPH piecewise model
* The Word report or verification log contains language flagging immunotherapy context and delayed treatment effect, the skill must surface this reasoning, not silently switch methods
* `gsd_results.json` total_N is ≤ 450 (enrollment cap respected)
* The futility look timing is flagged if the interim is projected to fall within 6 months of the delay window, OR the IA is placed post-delay
