---
date: 2022-04-15
title: Git Commands
summary: <i class="fab fa-git-alt"></i>&emsp;A place for handy Git commands
slug: git-commands
---

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
