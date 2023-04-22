---
date: 2023-04-22
title: Econometrics Glossary
subtitle: Definitions, Terminology, and Assumptions to Remember
summary: Descriptions of selection on observables assumptions, treatment effects terminology, and sources of endogeneity.
format: hugo
draft: false
slug: econ-glossary
categories:
- TIL
---

## Selection on Observables Assumptions

**Stable Unit Treatment Value Assumption (SUTVA):** There are no spill-overs (i.e., interference) between units that may affect a unit's treatment effect value.

**Conditional Independence (Unconfoundedness):** Treatment assignment is independent of potential outcomes when conditioned on all observable confounders. There is no other unobservable confounder that affects the relationship between treatment and outcome. (*Note: This assumption rarely holds in observational data as it would assume that units make decisions (i.e., treatments) that are entirely independent of potential outcomes which would imply complete irrationality.*)

**Common Support:** Units in the treatment and control group share the same covariate space in all relevant dimensions to effectively control for observable confounders. Otherwise, when covariate distributions are non-overlapping, specific covariate values may give away treatment and control units cannot serve as plausible counterfactuals.

## Treatment Effects

`\(D_i\)` is the treatment, `\(Z_i\)` is a vector of grouping/heterogeneity variables, `\(X_i\)` is the full vector of covariates.
It holds: `\(D_i \subset Z_i \subset X_i\)`

**Average treatment effect (ATE):**

$$ ATE = \theta = E(Y_i^{(1)} - Y_i^{(0)}) $$
Average treatment effect of treatment `\(D_i = 1\)` relative to `\(D_i = 0\)` (which can also reflect absence of treatment) across all units.
<br><br>

**Average treatment effect on the treated (ATET):**

$$ ATET = \theta(d) = E(Y_i^{(1)} - Y_i^{(0)} | D_i=1) $$
Average treatment effect of treatment `\(D_i = 1\)` relative to `\(D_i = 0\)` across all units that received the treatment, i.e., for which `\(D_i = 1\)`.
<br><br>

**Group average treatment effect (GATE):**

$$ GATE = \theta(z) = E(Y_i^{(1)} - Y_i^{(0)} | Z_i=z) $$
Average treatment effect of treatment `\(D_i = 1\)` relative to `\(D_i = 0\)` for each grouping along `\(Z_i\)` (heterogeneous effects).
<br><br>

**Individualized average treatment effect (IATE):**

$$ IATE = \theta(x) = E(Y_i^{(1)} - Y_i^{(0)} | X_i=x) $$
Average treatment effect of treatment `\(D_i = 1\)` relative to `\(D_i = 0\)` based on the full set of covariates `\(X_i\)` (heterogeneous effects).
<br><br>

**Dynamic Treatment Effects:**

Effect sizes vary, i.e., can increase or decrease over time

## Endogeneity

**Selection bias:** Firm self-selects into the sample or into the treatment

**Omitted variable bias (OVB):** Unobservable confounder enters the error term which is then correlated with both treatment and outcome

**Reverse Causality:** Outcome affects treatment status

**Measurement Error:** Error in measuring the treatment can lead to attenuation bias

**Simultaneity:** Treatment and Outcome are jointly determined, i.e., X causes Y and Y causes X

**Common method bias:** X and Y result from the same measurement
