## Issue 3
**Title:** `[benchmark][group-sequential-design] Front-loaded enrollment — information fraction vs calendar time decoupling`

## Skills
`group-sequential-design`

## Query
I'm designing a Phase 3 renal cell carcinoma trial, second-line after PD-1 failure. OS primary endpoint, control arm median 14 months, HR 0.72. 1:1 randomization. Target 90% power, two-sided alpha 0.05, O'Brien-Fleming spending, one interim at 40% of events with non-binding futility. Here's the enrollment situation: we have a big site network ready to go, so we expect very aggressive enrollment at the start, around 60 patients per month for the first 6 months, then it drops to about 15 per month for months 7-18 as most sites hit capacity, then a slow tail of about 5 per month for months 19–24. Total enrollment period is 24 months. Dropout 1% per month. Minimum follow-up 12 months for the last patient enrolled. Design the trial and tell me when the interim analysis will occur.

## Expected Output
The skill must use piecewise enrollment specification, `gamma = c(60, 15, 5)` and `R = c(6, 12, 6)` in `gsDesign::gsSurv()`, or `define_enroll_rate()` with three segments in `gsDesign2`, not a single constant rate. Because enrollment is heavily front-loaded, approximately 52% of all patients are enrolled in the first 25% of the enrollment period. Those patients have substantially longer follow-up and contribute events earlier than a uniform enrollment model predicts.

The correct output under piecewise enrollment:
* Total required events: ~310-330
* Calendar time to 40% events (IA): approximately month 21-23 from first patient in
* Information fraction at calendar month 28: approximately 0.52-0.58, not 0.40
* OBF efficacy boundary at actual 40% information fraction: z ≈ 2.96

If the skill uses uniform enrollment (approximately 29 patients/month constant), it will project the IA at approximately calendar month 27-29, 5-7 months later than correct, and any conditional power calculation at the IA will use the wrong information fraction. The downstream consequence: if the monitoring committee receives an interim report at the actual month 22, the skill's boundary is wrong because it was computed assuming the interim falls at 40% information when the actual fraction may be 30-35%.

## Rubric Criteria (Assertions)
* `gsd_design.R` uses piecewise enrollment: calls `gsSurv()` with `gamma = c(60, 15, 5)` and `R = c(6, 12, 6)`, or calls `gsDesign2::define_enroll_rate()` with three duration/rate segments, a single constant rate is a failing answer
* `gsd_results.json` reports calendar time to interim analysis ≤ 25 months from first patient in (uniform enrollment produces ~28 months, that is a failing answer)
* `gsd_results.json` reports total required events in the range 300-340
* `gsd_results.json` information fraction at the IA is computed as events-at-IA divided by total-planned-events, and is consistent with the calendar time reported (if IA is at month 22 and total events require month 36, the information fraction cannot be exactly 0.40)
* `gsd_verification_log.md` confirms simulated power ≥ 88% using the piecewise enrollment model
* Word report explicitly states the enrollment pattern (three-segment ramp) and its effect on IA timing, not a summary that treats enrollment as uniform
