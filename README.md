# pharma-skills-benchmarks

Statistical correctness benchmarks for the RConsortium [pharma-skills](https://github.com/RConsortium/pharma-skills) project, covering the **group-sequentialdesign**, **admiral-adsl**, **admiral-bds**, **clinical-trial-simulation**, and **r2rtf** AI agent skills.

20 benchmark cases submitted upstream · 2 merged pull requests fixing gaps the benchmarks exposed · 1 open infrastructure issue.

## What this is

The pharma-skills project is an open-source collection of AI agent skills for pharmaceutical statisticians, built by the R Consortium and BBSW. Skills wrap trusted R packages, `gsDesign`, `gsDesign2`, `lrstat`, `{admiral}`/`{admiralonco}`, `r2rtf`, with curated guidance so an AI agent calls the correct idiomatic function instead of reimplementing clinical trial statistics from scratch.

This repo documents 20 benchmark test cases submitted as GitHub issues to the upstream project, each one a **silent failure mode**: a scenario where the skill produces clean, professional, numerically plausible output that is statistically or structurally wrong in a way a non-expert would not detect.

## Curation logic

A benchmark earns its place here if it satisfies all three criteria:

1. The skill produces output confidently and without error
2. A non-expert user would accept the output as correct
3. A statistician would catch that it is wrong

These are harder to write than complexity benchmarks (co-primary endpoints, alpha recycling, multi-region timelines) because the failure is invisible in the formatted report. They require knowing specifically which R function the skill will call, what default behavior that function has, and exactly where the statistical error enters.

## Wave 1 - group-sequential-design (Issues #36-40, April 2026)

| File | Failure mode | Why it matters | Upstream issue |
|------|-------------|----------------|---------------|
| `issue-05-adaptive-enrichment-refusal.md` | Skill produces `gsDesign` output for adaptive enrichment scenario outside its valid scope | Scientifically invalid design delivered with false confidence | [RConsortium #36](https://github.com/RConsortium/pharma-skills/issues/36) |
| `issue-01-nph-self-detection.md` | Skill defaults to `gsSurv()` on immunotherapy trial without flagging delayed treatment effect | Underpowers trial by ~30% with no warning | [RConsortium #37](https://github.com/RConsortium/pharma-skills/issues/37) |
| `issue-03-front-loaded-enrollment.md` | Skill uses uniform enrollment instead of piecewise, misplaces IA by 5-7 months | Wrong information fraction, wrong boundary at IA | [RConsortium #38](https://github.com/RConsortium/pharma-skills/issues/38) |
| `issue-02-competing-risks-cvot.md` | Skill ignores non-CV death as competing event, uses CV death rate alone for event projection | Wrong event timeline, estimand mismatch | [RConsortium #39](https://github.com/RConsortium/pharma-skills/issues/39) |
| `issue-04-subgroup-futility-binding.md` | Skill uses binding futility for subgroup-only look, equates subgroup and ITT information fractions | FWER not controlled if trial continues past futility | [RConsortium #40](https://github.com/RConsortium/pharma-skills/issues/40) |

### Wave 1 results

Evaluated April 21, 2026 using gemini-3-flash-preview against skill version 2c0eaec.

| Issue | With Skill | Without Skill | Finding |
|-------|-----------|---------------|---------|
| [#36 Adaptive enrichment](https://github.com/RConsortium/pharma-skills/issues/36) | 0% | 57% | Critical: skill produced scientifically invalid design without warning |
| [#37 NPH self-detection](https://github.com/RConsortium/pharma-skills/issues/37) | 25% | 13% | Domain knowledge gap: both agents missed immunotherapy delayed-effect context |
| [#38 Front-loaded enrollment](https://github.com/RConsortium/pharma-skills/issues/38) | 71% | 7% | Skill wins: piecewise enrollment correctly implemented, IA at 14.9 months |
| [#39 Competing risks](https://github.com/RConsortium/pharma-skills/issues/39) | 31% | 13% | Both agents treated non-CV death as independent censoring |
| [#40 Subgroup futility](https://github.com/RConsortium/pharma-skills/issues/40) | 6% | 75% | Skill used binding futility, base model correctly used non-binding |

The 25% → 100% gap between gemini-3-flash-preview and Claude Opus 4.7 on Issue #37 confirmed that NPH self-detection is a **capability-dependent behavior**, the clinical domain knowledge exists in more capable models but is not reliably triggered by the skill in smaller ones. Agent A's 390-event design vs Agent B's 229-event design under Opus 4.7 is not a small numerical difference: it is the difference between a trial that hits 89% power and one that delivers 28% power under a realistic 6-month delay.

## Wave 2 - admiral & clinical-trial-simulation expansion (Issues #165-169, June 2026)

Motivated by Jeff Dickinson's pharmaverse blog post, 
["Why we still need {admiral} in an age of AI"](https://pharmaverse.github.io/blog/posts/2026-06-14_admiral_in_age_of_ai/admiral_in_age_of_ai.html) 
(June 14, 2026), and Yilong Zhang's follow-up request for more benchmark data to support those findings.

| Issue | Skill | Failure mode tested |
|-------|-------|---------------------|
| [#165](https://github.com/RConsortium/pharma-skills/issues/165) | admiral-adsl | SAFFL/ITTFL/PPROTFL population flags, "N" instead of NA for screen failures; SAFFL derived from `!is.na(TRTSDT)` instead of confirmed dosing |
| [#166](https://github.com/RConsortium/pharma-skills/issues/166) | admiral-bds | ADTTE time-to-event, manual date arithmetic silently fails to populate CNSR/CNSDTDSC/EVNTDESC correctly |
| [#167](https://github.com/RConsortium/pharma-skills/issues/167) | admiral-bds | ADRS/RECIST 1.1 tumor response, confirmation logic silently bypassed, inflating confirmed ORR |
| [#168](https://github.com/RConsortium/pharma-skills/issues/168) | clinical-trial-simulation | Bayesian adaptive dose-finding, frequentist decision logic masquerading as Bayesian posterior probability |
| [#169](https://github.com/RConsortium/pharma-skills/issues/169) | group-sequential-design | Platform trial with shared control arm, multiplicity inflation from concurrent sub-trial comparisons |

### Wave 2 results

| Issue | With Skill | Without Skill | Finding |
|-------|-----------|---------------|---------|
| [#165](https://github.com/RConsortium/pharma-skills/issues/165) | 82.6%* | 45.7%* | PPROTFL was dead commented-out code in SKILL.md, fixed in **PR #187**; PPROTFL assertion specifically confirmed Fail → Partial → **Pass** across 3 reruns** |
| [#166](https://github.com/RConsortium/pharma-skills/issues/166) | 84.1% | 65.9% | Unskilled agent independently converged on `derive_param_tte()`, skill's value is structural discipline (DOMAIN removal, date idioms, QC annotation), not function selection |
| [#167](https://github.com/RConsortium/pharma-skills/issues/167) | 77% / 65.9%*** | 59% / 63.6%*** | `{admiralonco}` availability in the runtime swings results significantly, see infra issue **#186** |
| [#169](https://github.com/RConsortium/pharma-skills/issues/169) | 81.8% | 72.7% | Both agents independently derived the correct correlation-aware α* ≈ 0.0188, more refined than this benchmark's original expected Bonferroni value |

\* Original benchmark scores at time of submission (before the PR #187 fix existed).  
\*\* Three independent reruns against the PR #187 fix branch used varying assertion subsets (23, then 23, then 18, likely different eval-runner configurations), so raw scores aren't directly comparable across runs. The PPROTFL `derive_vars_merged()` assertion itself was tracked consistently and is the cleanest signal: Fail (pre-fix) → Partial (pre-filtered, not inline `filter_add`) → **Pass** (inline `filter_add`, exactly the documented idiom).  
\*\*\* Two independent runs (jeffreyad, then ttt-77) produced different margins depending on whether `{admiralonco}` happened to be installed in that run's environment; see [#186](https://github.com/RConsortium/pharma-skills/issues/186).

**Issue #165 → PR #187:** Reading the admiral-adsl SKILL.md source directly showed the PPROTFL guidance was never implemented, only a commented-out stub referencing a placeholder variable that was never connected to any real data source. PR #187 implemented it using `derive_vars_merged()` with `filter_add` (an exclusion-flag pattern, distinct from SAFFL's inclusion-flag `derive_var_merged_exist_flag()`), added a `stop()` guard distinguishing "DV domain absent" from "DV present with zero major deviations," and was merged after a review round addressing SAS XPT 8-character variable-name compliance.

**Issue #169** surfaced a recurring pattern worth naming: when a task scope-shifts outside the skill's standard workflow (here, into `mvtnorm`/correlation territory for platform trials), the skill's otherwise-reliable `# REVIEW:` QC-comment habit doesn't reliably persist. Both the skilled and unskilled agent missed all four required QC comments on this task, despite the skilled agent nailing them on in-scope GSD benchmarks.

## Wave 3 - interim analysis, PK, SSR, KM tables, non-inferiority (Issues #171-175, June 2026)

| Issue | Skill | Failure mode tested |
|-------|-------|---------------------|
| [#171](https://github.com/RConsortium/pharma-skills/issues/171) | group-sequential-design | Conditional power / predictive probability of success computed at the wrong information fraction for DMC decision support |
| [#172](https://github.com/RConsortium/pharma-skills/issues/172) | admiral-bds | ADPX pharmacokinetics, AUC/Cmax/Tmax with BLQ imputation and linear-up/log-down trapezoidal rule |
| [#173](https://github.com/RConsortium/pharma-skills/issues/173) | clinical-trial-simulation | Sample size re-estimation at interim, blinded internal pilot (Wittes-Brittain) without type I error inflation |
| [#174](https://github.com/RConsortium/pharma-skills/issues/174) | r2rtf | Kaplan-Meier survival table, median CI method, number-at-risk integers, censoring annotation |
| [#175](https://github.com/RConsortium/pharma-skills/issues/175) | group-sequential-design | Non-inferiority margin justification via the M1/M2 framework and assay sensitivity |

Results pending: evaluation pipeline still processing this wave as of the last update to this README. #171's first run is the one exception: an instructive **negative result** is already in, see below.

### #171 - when the skill underperformed

| Metric | With Skill | Without Skill |
|--------|-----------|---------------|
| Score | 59.6% | **73.1%** |
| Cond. power @ HR=0.72 (target 52–60%) | 74.6% ❌ | 57.7% ✅ |
| Cond. power @ HR=0.81 (target 20–30%) | 45.6% ❌ | 28.2% ✅ |

This is the only benchmark in the suite so far where the unskilled agent scored higher. The decisive difference was the central quantitative target: the skilled agent computed conditional power outside the reference range and disputed the range rather than meeting it, using a calibrated hazard rate (7.93/mo) rather than the raw events-per-month rate (148/28 ≈ 5.3/mo) the benchmark specifies. Both agents missed all four required QC `# REVIEW:` comments, a shared gap, not a skill 
advantage either way. Root-cause investigation (whether the skill's calibrated rate is actually more correct, or a genuine error) is ongoing.

## Merged fixes

| PR | What it fixed | Confirmed by |
|----|---------------|--------------|
| [**#152**](https://github.com/RConsortium/pharma-skills/pull/152) | `derive_vars_merged_lookup()` enforcement for PARAMCD assignment in admiral-bds, skill was using the non-lookup variant, silently dropping unmatched records | 4 independent re-runs; score improved 94.4% → 98.9% |
| [**#187**](https://github.com/RConsortium/pharma-skills/pull/187) | PPROTFL derivation in admiral-adsl, guidance existed only as dead commented-out code; implemented using the correct exclusion-flag idiom | 2 independent reruns against #165; PPROTFL assertion improved Fail → Partial → **Pass** (fully validated, skilled agent now uses `derive_vars_merged()` with inline `filter_add`, exactly matching the documented idiom) |

## Open infrastructure issue

[**#186**](https://github.com/RConsortium/pharma-skills/issues/186), `{admiralonco}` is not consistently installed in the benchmark runtime, which has produced materially different results across repeat runs of the same oncology benchmark (#167) depending on which agent happened to have the package available.

A second independent run confirmed the asymmetry in reverse: in the first run, the skilled agent lacked `{admiralonco}` and fell back to manual base-`{admiral}` reimplementation; in the second run, it was the *unskilled* agent that successfully self-installed the package from CRAN and used the full idiomatic function suite, while the skilled agent again fell back to manual derivation. Across both runs, no agent, skilled or unskilled, has yet attempted `install.packages("admiralonco")` as part of the *skilled* workflow itself, despite the unskilled agent demonstrating twice that this is a viable, low-effort fix. Proposed as a SKILL.md addition: attempt package installation before falling back to manual implementation when `{admiralonco}` is absent.

## Methodology note: self-correcting benchmark design

Across waves 2 and 3, several of my own benchmark assertions turned out to be miscalibrated rather than the skill being wrong, and I've tried to be transparent about that rather than only reporting wins:

- **#165**: "PPROTFL count ≤ SAFFL count" is structurally false under the stated flag definitions (Placebo subjects can qualify for PPROTFL without qualifying for SAFFL); rescoped rather than treated as a skill failure.
- **#166**: "nrow(adtte) == nrow(adsl)" failed for both agents because `derive_param_tte()` correctly excludes screen failures and subjects with incomplete treatment dates by design, not by error.
- **#167**: the CBRESPFL "Y"/NA-only assertion was wrong; `derive_param_clinbenefit()`'s output is explicitly binary "Y"/"N", and both agents producing "N" was correct admiralonco behavior. Separately, the confirmed-vs-unconfirmed ORR comparison print (assertion #11, the benchmark's core sanity check) has now failed 
  identically across three independent runs regardless of skill, pointing to a genuine missing instruction rather than benchmark miscalibration.
- **#169**: the benchmark's own expected Bonferroni value (≈0.0083) was more conservative than necessary; both agents correctly produced a more statistically refined, less conservative, still-valid correlation-aware answer (≈0.0188).

This pattern, repeatedly assuming 1:1 structural parity with ADSL where the actual admiral function has principled exclusion logic, is itself a useful finding about benchmark design, not just about the skill.

## Cross-project contributions

Beyond the 20 benchmark cases authored directly, contributed analytical synthesis on [**#183**](https://github.com/RConsortium/pharma-skills/issues/183) (`clinical-trial-ipd-sim` feedback cluster, #179-184, authored by lengning), identifying a shared root cause across three separate issues, AE onset timing (#182), visit timing (#183), and discontinuation timing, all stemming from the same architectural pattern: the forward simulation evaluates state only at discrete visit timepoints when the underlying clinical process is continuous-time. Proposed a single shared spec pattern (continuous-time event sampling within visit windows) rather than three separate fixes, connecting it to the same discrete-grid-vs-continuous-time tension seen in the ADTTE (#166) and conditional-power (#171) benchmarks above.

## Upstream submissions

- Wave 1 (Issues #36-40): filed April 19, 2026
- Wave 2 (Issues #165-169): filed June 16, 2026
- Wave 3 (Issues #171-175): filed June 16, 2026

All filed at [RConsortium/pharma-skills/issues](https://github.com/RConsortium/pharma-skills/issues).

## Recognition

Acknowledged by project lead Yilong Zhang (Health Quantitative Scientist, Meta): *"Much appreciate the contributions! Eval results are under the way."*, and later, in response to wave 2: *"Wish we can have more benchmark data to support the findings."*

Merged contributor with repo access; invited to the RConsortium Pharma Skills core team Slack channel and weekly Pilot 7 standups following benchmark evaluation results, April 2026.

PR #152 and #187 reviewed by core contributors **jeffreyad** and **elong0527**.

## Author

**Emmanuel Fle Chea, MPH**  
Founder & CEO, Lexify Health | Clinical Data Scientist  
[github.com/efchea1](https://github.com/efchea1) · 
[lexify.health](https://lexify.health) · 
[LinkedIn](https://linkedin.com/in/emmanuel-fle-chea)

## License
MIT License - see [LICENSE](LICENSE) for details.
