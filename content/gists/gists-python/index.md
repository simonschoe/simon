---
title: Python Snippets
subtitle: Subtitle
summary: <i class="fab fa-python"></i>
author: Simon Schölzel
date: 2022-04-15
slug: python-gists
categories:
  - Python
tags:
  - Python
---

### Working with `.venv` 

Create a new virtual environment
```bash
python -m venv .venv
```

Activate virtual environment
```bash
.\.venv\Scripts\activate
```

Check installed packages
```bash
pip list
```

Create `requirements.txt`
```bash
pip freeze > requirements.txt
```

Quit the virtual environment
```bash
deactivate
```

Install packages from `requirements.txt`
```bash
python -m pip install -r requirements.txt
```
