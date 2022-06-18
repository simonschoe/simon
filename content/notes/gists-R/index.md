---
title: R Gists
subtitle: A place for handy R code snippets
summary: A collection of useful `R` code snippets that I rarely use and frequently escape my mind. <i class="fab fa-r-project"></i>
author: Simon Schölzel
date: 2022-04-15
slug: r-gists
categories:
  - R
tags:
  - R
---

### `blogdown`

Update `hugo-apero` theme
```r
blogdown::install_theme
  theme = "hugo-apero/hugo-apero",
  update_config = F, force = T
)
```

