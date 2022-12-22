---
date: 2022-04-15
title: R Gists
subtitle: A place for handy R code snippets
summary: A collection of useful `R` code snippets that I rarely use and that frequently escape my mind. <i class="fab fa-r-project"></i>
slug: r-gists
---

### `blogdown`

Update `hugo-apero` theme
```r
blogdown::install_theme
  theme = "hugo-apero/hugo-apero",
  update_config = F, force = T
)
```

