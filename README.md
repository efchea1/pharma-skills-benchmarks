# pharma-skills-benchmarks

Statistical correctness benchmarks for the [RConsortium/pharma-skills](https://github.com/RConsortium/pharma-skills) project, covering the group-sequential-design, admiral (ADSL, ADTTE, ADRS, BDS), clinical-trial-simulation, and r2rtf AI agent skills.

**15 benchmark cases submitted upstream · 2 merged pull requests fixing gaps the benchmarks exposed · 3 admiral benchmark runs I executed myself (leading to PR #152) · 1 infrastructure issue (resolved)**

*Results below are drawn from the public issue threads and were last checked November 17, 2026.*

## What this is

The pharma-skills project is an open-source collection of AI agent skills for pharmaceutical statisticians, built by the R Consortium and BBSW. Skills wrap trusted R packages, gsDesign, gsDesign2, lrstat, {admiral}/{admiralonco}, r2rtf, with curated guidance so an AI agent calls the correct idiomatic function instead of reimplementing clinical trial statistics from scratch.

This repo documents 15 benchmark test cases submitted as GitHub issues to the upstream project, each one a silent failure mode: a scenario where the skill produces clean, professional, numerically plausible output that is statistically or structurally wrong in a way a non-expert would not detect.

### Curation logic

A benchmark earns its place here if it satisfies all three criteria:

1. The skill produces output confidently and without error
2. A non-expert user would accept the output as correct
3. A statistician would catch that it is wrong

These are harder to write than complexity benchmarks (co-primary endpoints, alpha recycling, multi-region timelines) because the failure is invisible in the formatted report. They require knowing specifically which R function the skill will call, what default behavior that function has, and exactly where the statistical error enters.

## Contribution timeline

| Date (2026) | What happened |
|---|---|
| Apr 19-20 | Wave 1 filed: five group-sequential-design benchmarks (#36–40) |
| Jun 6-7 | Ran the admiral-bds / admiral-adae benchmarks on #147, #148 and #149 (issues authored by Jeff Dickinson) |
| Jun 7-10 | Opened PR #152 from the pattern those runs exposed; merged Jun 10 after review |
| Jun 16 | Waves 2 and 3 filed: ten benchmarks (#165–169, #171–175) |
| Jun 24-25 | Opened infra issue #186; opened PR #187 (PPROTFL); merged Jun 25 |
| Jun 29 | Cross-issue synthesis on the clinical-trial-ipd-sim feedback cluster (#183) |
| Oct 15 | Latest scored runs on #166, #167 and #172 against the registered `admiral-adrs` skill |
| Oct 20 | Speaker, R/Pharma 2026: "Benchmarking AI for Clinical Trial Design: Silent Failures in the RConsortium pharma-skills Skill" |
| Oct 23 | Infra issue #186 closed as completed |

## Results at a glance

Across the 15 benchmarks, 14 now have at least one scored run (#174 has none yet), with 40+ scored runs from five contributors. In each benchmark's **most recent** run:

- **Skill scored higher in 10:** #36, #39, #40, #165, #166, #167, #168, #169, #172, #175
- **Skill scored lower in 4:** #37, #38, #171, #173 (each explained below)
- **No run yet:** #174

Runs differ in model, skill version and rubric, so scores are not directly comparable across runs. Each table shows how a benchmark's result moved over time.

## Wave 1: group-sequential-design (Issues #36-40, April 2026)

| File | Failure mode | Why it matters | Upstream issue |
|---|---|---|---|
| issue-05-adaptive-enrichment-refusal.md | Skill produces gsDesign output for adaptive enrichment scenario outside its valid scope | Scientifically invalid design delivered with false confidence | RConsortium #36 |
| issue-01-nph-self-detection.md | Skill defaults to gsSurv() on immunotherapy trial without flagging delayed treatment effect | Underpowers trial by ~30% with no warning | RConsortium #37 |
| issue-03-front-loaded-enrollment.md | Skill uses uniform enrollment instead of piecewise, misplaces IA by 5-7 months | Wrong information fraction, wrong boundary at IA | RConsortium #38 |
| issue-02-competing-risks-cvot.md | Skill ignores non-CV death as competing event, uses CV death rate alone for event projection | Wrong event timeline, estimand mismatch | RConsortium #39 |
| issue-04-subgroup-futility-binding.md | Skill uses binding futility for subgroup-only look, equates subgroup and ITT information fractions | FWER not controlled if trial continues past futility | RConsortium #40 |

### Wave 1 results (with skill vs. without skill)

The April 21 baseline used gemini-3-flash-preview against skill version 2c0eaec. Later reruns by other contributors used Claude models and newer skill versions.

| Issue | Apr 21 baseline | Later runs | Finding |
|---|---|---|---|
| #36 Adaptive enrichment | 0% vs 57% | 93/57 (May 13), 93/64 (Jun 28), 100/86 (Jul 2), 86/79 (Aug 13), 86/64 (Sep 11) | Critical failure at baseline; the skill led in all five later runs |
| #37 NPH self-detection | 25% vs 13% | Opus 4.7: 100/44 (Apr 22); Sonnet 4.6: 25/25 (May 13), 38/25 (Jun 15); 0/38 (Sep 1) | Domain knowledge gap: capability-dependent and not yet stable. Issue was reopened |
| #38 Front-loaded enrollment | 71% vs 7% | 86/7 (May 17); 43/50 (Jun 30) | Skill wins on piecewise enrollment, but the Jun 30 run reversed |
| #39 Competing risks | 31% vs 13% | 56/38 (May 19), 6/25 (Jul 16), 38/31 (Jul 22) | Both agents treated non-CV death as independent censoring; margins are small and noisy |
| #40 Subgroup futility | 6% vs 75% | 50/13 (May 20), 31/13 (Jul 1) | Skill used binding futility at baseline; later runs favor the skill |

The 25% to 100% gap between gemini-3-flash-preview and Claude Opus 4.7 on Issue #37 confirmed that NPH self-detection is a capability-dependent behavior: the clinical domain knowledge exists in more capable models but is not reliably triggered by the skill in smaller ones. Agent A's 390-event design vs Agent B's 229-event design under Opus 4.7 is the difference between a trial that hits 89% power and one that delivers 28% power under a realistic 6-month delay.

## admiral-bds / admiral-adae runs I executed (Issues #147-149, June 6-11, 2026)

Before authoring Wave 2, I ran the A/B benchmark myself on three admiral issues authored by Jeff Dickinson, using claude-sonnet-4-6. The results exposed the one gap that repeated across runs and led directly to PR #152.

| Issue | Skill | With skill | Without skill | Notes |
|---|---|---|---|---|
| #147 | admiral-bds (ADVS) | 88.2% (skill ec6bbe8) → 94.7% after PR #152 | 35.5% → 46.1% | BASETYPE detection for SYSBP/DIABP/PULSE was the key skill advantage; unskilled agent produced 8,207 duplicate keys |
| #148 | admiral-adae (ADAE) | 93.8% (skill ccd2cd9) | 33.3% | Skill passed all 6 admiral idiom assertions (e.g. `derive_var_trtemfl()`); unskilled agent used `as.Date()` and manual `case_when()` |
| #149 | admiral-bds (ADLB) | 94.4% (skill ec6bbe8) → 96.7%–98.9% in three reruns after PR #152 | 47.8% | Unskilled agent added unrequested DTYPE rows (85,209 vs 59,580 expected) and non-standard PARAMCD names (K→POTAS, LYM→LYMPH) |

The repeated gap: in both #147 and #149 the skill-guided agent built the correct parameter lookup table but then called `derive_vars_merged()` instead of `derive_vars_merged_lookup()` for PARAMCD/PARAM/PARAMN assignment. A `flag_imputation = "date"` vs `"auto"` error seen in #147 had already been corrected by the #149 run.

Open suggestion from the #149 follow-up: have the skill specify STUDYID+USUBJID+PARAMCD+AVISITN as the uniqueness key rather than leaving it to the agent.

## Wave 2: admiral & clinical-trial-simulation expansion (Issues #165–169, June 2026)

Motivated by Jeff Dickinson's pharmaverse blog post, "Why we still need {admiral} in an age of AI" (June 14, 2026), and Yilong Zhang's follow-up request for more benchmark data to support those findings.

| Issue | Skill | Failure mode tested |
|---|---|---|
| #165 | admiral-adsl | SAFFL/ITTFL/PPROTFL population flags, "N" instead of NA for screen failures; SAFFL derived from !is.na(TRTSDT) instead of confirmed dosing |
| #166 | admiral-adtte | ADTTE time-to-event, manual date arithmetic silently fails to populate CNSR/CNSDTDSC/EVNTDESC correctly |
| #167 | admiral-adrs (retargeted from admiral-bds) | ADRS/RECIST 1.1 tumor response, confirmation logic silently bypassed, inflating confirmed ORR |
| #168 | clinical-trial-simulation | Bayesian adaptive dose-finding, frequentist decision logic masquerading as Bayesian posterior probability |
| #169 | group-sequential-design | Platform trial with shared control arm, multiplicity inflation from concurrent sub-trial comparisons |

### Wave 2 results

| Issue | With skill vs. without skill (chronological) | Finding |
|---|---|---|
| #165 | 82.6/45.7 (Jun 23, before PR #187); my rerun on the PR #187 branch 80.4/43.5 (Jun 24); after rescoping to 18 assertions: 75/50, 83/44, 91.7/47.2 (Aug 15), 86.1/41.7 (Sep 5) | PPROTFL was dead commented-out code in SKILL.md, fixed in PR #187. The PPROTFL assertion was confirmed Fail → Partial → Pass across 3 reruns* |
| #166 | 84.1/65.9 (Jun 23), 81.6/60.5 (Aug 6), 88.9/61.1 (Oct 15) | Skill ahead in every run. The unskilled agent independently converged on `derive_param_tte()`; the skill's value is structural discipline (DOMAIN removal, date idioms, QC annotation), not function selection |
| #167 | 77/59 (Jun 23), 65.9/63.6 (Jun 24), 68.2/65.9 (Jul 3), 75.0/50.0 (Oct 15) | {admiralonco} availability swung the first two runs (see infra issue #186). The Oct 15 run is the widest margin, after retargeting to the `admiral-adrs` skill; both agents used {admiralonco} |
| #168 | 70.5/68.2 (Jul 20), 68.2/61.4 (Jul 29) | Skill ahead but fragile: the margin rests on one assertion (probability of correct selection) within simulation noise. The skill-guided agent placed 0/4 `# REVIEW:` markers in the second run |
| #169 | 81.8/72.7 (Jun 22) | Both agents independently derived the correct correlation-aware α* ≈ 0.0188, more refined than this benchmark's original expected Bonferroni value |

\* Three independent reruns against the PR #187 fix branch used varying assertion subsets (23, then 23, then 18, likely different eval-runner configurations), so raw scores aren't directly comparable across runs. The PPROTFL `derive_vars_merged()` assertion itself was tracked consistently and is the cleanest signal: Fail (pre-fix) → Partial (pre-filtered, not inline filter_add) → Pass (inline filter_add, exactly the documented idiom).

Issue #169 surfaced a recurring pattern worth naming: when a task scope-shifts outside the skill's standard workflow (here, into mvtnorm/correlation territory for platform trials), the skill's otherwise-reliable `# REVIEW:` QC-comment habit doesn't reliably persist. Both the skilled and unskilled agent missed all four required QC comments on this task, despite the skilled agent nailing them on in-scope GSD benchmarks.

## Wave 3: interim analysis, PK, SSR, KM tables, non-inferiority (Issues #171–175, June 2026)

| Issue | Skill | Failure mode tested |
|---|---|---|
| #171 | group-sequential-design | Conditional power / predictive probability of success computed at the wrong information fraction for DMC decision support |
| #172 | admiral-bds | ADPX pharmacokinetics, AUC/Cmax/Tmax with BLQ imputation and linear-up/log-down trapezoidal rule |
| #173 | clinical-trial-simulation | Sample size re-estimation at interim, blinded internal pilot (Wittes-Brittain) without type I error inflation |
| #174 | r2rtf | Kaplan-Meier survival table, median CI method, number-at-risk integers, censoring annotation |
| #175 | group-sequential-design | Non-inferiority margin justification via the M1/M2 framework and assay sensitivity |

### Wave 3 results

| Issue | With skill vs. without skill | Finding |
|---|---|---|
| #171 | 59.6/73.1 (Jun 24) | Skill underperformed (details below). Root-cause investigation is ongoing |
| #172 | 97/53 (Jul 2), 78.3/56.5 (Oct 15) | Skill ahead on admiral idioms, DOMAIN cleanup, uniqueness checks and QC discipline; core NCA numerics were equivalent. Exposed that admiral-bds does not formally cover PK/NCA, a useful scope-documentation signal |
| #173 | 65.9/72.7 (Jul 8), 38.6/72.7 (Aug 29) | Unskilled agent ahead in both runs. The first was two mechanical assertions; the second was operational: the skill-guided agent's report-templating step never completed, leaving type I error, power and the FDA-threshold verdict as unfilled placeholders |
| #174 | no scored run yet | |
| #175 | 26.7/31.7 (Jul 14), 50/31 (Aug 13) | Both agents initially failed the M1/M2 derivation (HR-scale CI bound instead of log-HR). A later run reversed the result (+19 pp) but still missed the regulatory narrative and QC layer (constancy, assay sensitivity, biocreep, `# REVIEW:` tags) |

### #171: when the skill underperformed

| Metric | With Skill | Without Skill |
|---|---|---|
| Score | 59.6% | 73.1% |
| Cond. power @ HR=0.72 (target 52–60%) | 74.6% ❌ | 57.7% ✅ |
| Cond. power @ HR=0.81 (target 20–30%) | 45.6% ❌ | 28.2% ✅ |

The decisive difference was the central quantitative target: the skilled agent computed conditional power outside the reference range and disputed the range rather than meeting it, using a calibrated hazard rate (7.93/mo) rather than the raw events-per-month rate (148/28 ≈ 5.3/mo) the benchmark specifies. Both agents missed all four required QC `# REVIEW:` comments, a shared gap, not a skill advantage either way. Root-cause investigation (whether the skill's calibrated rate is actually more correct, or a genuine error) is ongoing. In the earlier Wave 1 baseline, #36 and #40 also scored lower with the skill; #171 was the first such result in the Claude-era reruns, and #173 and #175 later showed similar reversals.

## Merged fixes

| PR | What it fixed | Review | Confirmed by |
|---|---|---|---|
| [#152](https://github.com/RConsortium/pharma-skills/pull/152) | `derive_vars_merged_lookup()` enforcement for PARAMCD assignment in admiral-bds: the skill was using the non-lookup variant, silently dropping unmatched records. Three reinforcements to `admiral/admiral-bds/SKILL.md` (Step 3 prose, Common BDS errors, output checklist); +9/−3 | Changes requested by jeffreyad (Jun 8), approved Jun 10, merged Jun 10 | 4 independent re-runs; #149 score improved 94.4% → 98.9% (96.7%–98.9% across three reruns), #147 improved 88.2% → 94.7% |
| [#187](https://github.com/RConsortium/pharma-skills/pull/187) | PPROTFL derivation in admiral-adsl: guidance existed only as dead commented-out code. Implemented with `derive_vars_merged()` + `filter_add` (an exclusion-flag pattern, distinct from SAFFL's inclusion-flag `derive_var_merged_exist_flag()`), a `stop()` guard separating "DV domain absent" from "DV present with zero major deviations", `mode = "first"` handling, and a `# REVIEW:` note on DVCAT vs DVSCAT; +38/−3 | Changes requested by jeffreyad (Jun 24; SAS XPT 8-character variable-name compliance), approved and merged by elong0527 (Jun 25) | 2 independent reruns against #165; PPROTFL assertion improved Fail → Partial → Pass |

Issue #165 → PR #187: reading the admiral-adsl SKILL.md source directly showed the PPROTFL guidance was never implemented, only a commented-out stub referencing a placeholder variable that was never connected to any real data source. One test-plan item on PR #187 remains open: the `stop()` guard has not been exercised, because the DS/DV domain has been present in pharmaversesdtm in every rerun.

## Infrastructure issue (resolved)

[#186](https://github.com/RConsortium/pharma-skills/issues/186): {admiralonco} was not consistently installed in the benchmark runtime, which produced materially different results across repeat runs of the same oncology benchmark (#167) depending on which agent happened to have the package available. **Closed as completed on October 23, 2026** by jeffreyad, after elong0527 asked whether it was still an issue.

Two independent runs had confirmed the asymmetry in reverse: in the first run the skilled agent lacked {admiralonco} and fell back to manual base-{admiral} reimplementation; in the second run it was the unskilled agent that self-installed the package from CRAN and used the full idiomatic function suite. Proposed as a SKILL.md addition: attempt package installation before falling back to manual implementation when {admiralonco} is absent. From the Jul 3 run onward, both agents used {admiralonco}.

## Methodology note: self-correcting benchmark design

Across waves 2 and 3, several of my own benchmark assertions turned out to be miscalibrated rather than the skill being wrong, and I've tried to be transparent about that rather than only reporting wins:

- **#165:** "PPROTFL count ≤ SAFFL count" is structurally false under the stated flag definitions (Placebo subjects can qualify for PPROTFL without qualifying for SAFFL); rescoped rather than treated as a skill failure.
- **#166:** "nrow(adtte) == nrow(adsl)" failed for both agents because `derive_param_tte()` correctly excludes screen failures and subjects with incomplete treatment dates by design, not by error.
- **#167:** the CBRESPFL "Y"/NA-only assertion was wrong; `derive_param_clinbenefit()`'s output is explicitly binary "Y"/"N" (the function is deprecated as of admiralonco 1.4.0 in favor of `admiral::derive_extreme_event()`), so both agents producing "N" was correct behavior. Planned rubric revision: split into 14a (BESTRSPFL, ANL01FL, CONFIRMED: Y/NA) and 14b (CBRESPFL: Y/N), and accept `derive_extreme_event()` as the preferred idiom. The Oct 15 run also shows three more assertions that conflict with {admiralonco}'s defaults: the exact AVAL numbering (the package's `aval_resp()` keeps the required order but numbers PD=5 and NE=6), the AVALC value set (`MISSING`, `NON-CR/NON-PD`), and BESTRSPFL (the package derives a BESTRESP parameter instead). Separately, the confirmed-vs-unconfirmed ORR comparison print (the benchmark's core sanity check) failed in the first three runs regardless of skill; the Oct 15 skill-guided run was the first to print both counts and add a `stopifnot()` check.
- **#169:** the benchmark's own expected Bonferroni value (≈0.0083) was more conservative than necessary; both agents correctly produced a more statistically refined, less conservative, still-valid correlation-aware answer (≈0.0188).

This pattern, repeatedly assuming 1:1 structural parity with ADSL where the actual admiral function has principled exclusion logic, is itself a useful finding about benchmark design, not just about the skill.

## Cross-project contributions

- **Clinical-trial-ipd-sim feedback (#183):** contributed analytical synthesis on the feedback cluster #179–184 (authored by lengning), identifying a shared root cause across three separate issues, AE onset timing (#182), visit timing (#183), and discontinuation timing, all stemming from the same architectural pattern: the forward simulation evaluates state only at discrete visit timepoints when the underlying clinical process is continuous-time. Proposed a single shared spec pattern (continuous-time event sampling within visit windows) rather than three separate fixes, connecting it to the same discrete-grid-vs-continuous-time tension seen in the ADTTE (#166) and conditional-power (#171) benchmarks above.
- **Benchmark reruns:** posted scored reruns and follow-up analysis on other contributors' issues (#147–149, #165, #166, #167, #169) as described above.
- **YAMAA SDTM/ADaM benchmark (Pilot 7):** volunteer reviewer for the ADaM-side cases (ADSL, ADAE, ADTTE, ADRS), following the same admiral work.

## Upstream submissions

- Wave 1 (Issues #36–40): filed April 19–20, 2026
- Wave 2 (Issues #165–169): filed June 16, 2026
- Wave 3 (Issues #171–175): filed June 16, 2026

All filed at [RConsortium/pharma-skills/issues](https://github.com/RConsortium/pharma-skills/issues).

## Recognition

Acknowledged by project lead Yilong Zhang (Health Quantitative Scientist, Meta): "Much appreciate the contributions! Eval results are under the way.", and later, in response to wave 2: "Wish we can have more benchmark data to support the findings."

Merged contributor with repo access; invited to the RConsortium Pharma Skills core team Slack channel and weekly Pilot 7 standups following benchmark evaluation results, April 2026.

PR #152 and #187 reviewed by core contributors jeffreyad and elong0527.

Speaker, R/Pharma 2026 (October 20, 2026).

## Author

Emmanuel Fle Chea, MPH
Founder & Sole Developer, Lexify Health | Clinical Data Scientist
[github.com/efchea1](https://github.com/efchea1) · [lexify.health](https://lexify.health) · LinkedIn

## License

MIT License - see LICENSE for details.
