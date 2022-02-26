---
title: Machine Learning Workflows with Tidymodels
subtitle: 
author: Simon Schölzel
weight: 4
date: 2021-10-01
draft: false
excerpt: This lecture is part of the "Machine Learning in R" graduate course held at University of Münster, School of Business and Economics (winter term 2021/22).
links:
- icon: door-open
  icon_pack: fas
  name: Slides
  url: https://simonschoe.github.io/ml-with-tidymodels/#1
- icon: github
  icon_pack: fab
  name: code
  url: https://github.com/simonschoe/ml-with-tidymodels
categories:
- Workshop
tags:
---

### Contents

This lecture gives a deep insight into `tidymodels`, a unified framework towards modeling and machine learning in `R` using tidy data principles. It introduces and motivates tools that facilitate every step of the machine learning workflow, from resampling, over preprocessing and model building, to model tuning and performance evaluation.

More specifically, after this lecture you will
- be familiar with the core packages of the `tidymodels` ecosystem and hopefully realize the value of a unified modeling framework,
- know how to design a full-fledged machine learning pipeline for a particular prediction task,
- broaden your technical skills by learning about declarative programming, hyperparameter scales and parallel processing, and
- most importantly, be capable of conducting your own machine learning projects in `R`.

<br>

### Agenda

**1 Learning Objectives**

**2 Introduction to `tidymodels`**

**3 Himalayan Climbing Expeditions Data**

**4 The Core `tidymodels` Packages**

>4.1 `rsample`: General Resampling Infrastructure  
4.2 `recipes`: Preprocessing Tools to Create Design Matrices  
4.3 `parsnip`: A Common API to Modeling and Analysis Functions  
4.4 `workflows`: Modeling Workflows  
4.5 `dials`: Tools for Creating Tuning Parameter Values  
4.6 `tune`: Tidy Tuning Tools  
4.7 `broom`: Convert Statistical Objects into Tidy Tibbles  
4.8 `yardstick`: Tidy Characterizations of Model Performance

**5 Additions to the `tidymodels` Ecosystem**
