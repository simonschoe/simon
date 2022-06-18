---
title: Git Commands
subtitle:  A place for handy Git commands
summary: A collection of useful `Git` commands that I rarely use and frequently escape my mind. <i class="fab fa-git-alt"></i>
author: Simon Schölzel
date: 2022-04-15
slug: git-gists
categories:
  - Git
tags:
  - Git
---

### <i class="fab fa-git-alt"></i>

Update Git remote URL (e.g., after renaming)
```bash
git remote set-url origin <updated-url>
```

Remove Files from Git after adding/updating `.gitignore`
```bash
git rm --cached <filename>
```

Deactivate automatic repository language detection by adding a `.gitattributes` file
```bash
*.html linguist-detectable=false
```
