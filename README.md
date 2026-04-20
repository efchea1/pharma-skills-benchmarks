# pharma-skills-benchmarks

Statistical correctness benchmarks for the RConsortium 
[pharma-skills](https://github.com/RConsortium/pharma-skills) 
group-sequential-design AI skill.

## What this is

The pharma-skills project is an open-source collection of AI agent 
skills for pharmaceutical statisticians, built by the R Consortium 
and BBSW. The group-sequential-design skill uses gsDesign, gsDesign2, 
and lrstat to design clinical trials, verify via simulation, and 
generate Word reports.

This repo contains five benchmark test cases submitted as GitHub issues 
to the upstream project. Each benchmark targets a **silent failure mode**, a scenario where the skill produces clean, professional, numerically 
plausible output that is statistically wrong in a way a non-expert 
would not detect.

## Curation logic

A benchmark earns its place here if it satisfies all three criteria:

1. The skill produces output confidently and without error
2. A non-expert user would accept the output as correct
3. A statistician would catch that it is wrong

These are harder to write than complexity benchmarks (co-primary 
endpoints, alpha recycling) because the failure is invisible in the 
formatted report. They require knowing specifically which R function 
the skill will call, what default behavior that function has, and 
exactly where the statistical error enters.

## The five benchmarks

| File | Failure mode | Why it matters | Upstream issue |
|------|-------------|----------------|---------------|
| `issue-05-adaptive-enrichment-refusal.md` | Skill produces `gsDesign` output for adaptive enrichment scenario outside its valid scope | Scientifically invalid design delivered with false confidence | [RConsortium #36](https://github.com/RConsortium/pharma-skills/issues/36) |
| `issue-01-nph-self-detection.md` | Skill defaults to `gsSurv()` on immunotherapy trial without flagging delayed treatment effect | Underpowers trial by ~30% with no warning | [RConsortium #37](https://github.com/RConsortium/pharma-skills/issues/37) |
| `issue-03-front-loaded-enrollment.md` | Skill uses uniform enrollment instead of piecewise, misplaces IA by 5-7 months | Wrong information fraction, wrong boundary at IA | [RConsortium #38](https://github.com/RConsortium/pharma-skills/issues/38) |
| `issue-02-competing-risks-cvot.md` | Skill ignores non-CV death as competing event, uses CV death rate alone for event projection | Wrong event timeline, estimand mismatch | [RConsortium #39](https://github.com/RConsortium/pharma-skills/issues/39) |
| `issue-04-subgroup-futility-binding.md` | Skill uses binding futility for subgroup-only look, equates subgroup and ITT information fractions | FWER not controlled if trial continues past futility | [RConsortium #40](https://github.com/RConsortium/pharma-skills/issues/40) |

## Upstream submissions

All five issues were filed at 
[RConsortium/pharma-skills/issues](https://github.com/RConsortium/pharma-skills/issues) 
on April 19, 2026.

## Author

**Emmanuel Fle Chea, MPH**  
Founder & CEO, Lexify | Clinical Data Scientist | Healthcare AI  
[github.com/efchea1](https://github.com/efchea1) · 
[lexify.health](https://lexify.health) · 
[LinkedIn](https://linkedin.com/in/emmanuel-fle-chea)
