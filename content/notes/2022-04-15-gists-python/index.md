---
date: 2022-04-15
title: Python Gists
subtitle: A place for handy Python code snippets
summary: A collection of useful `Python` code snippets that I rarely use and that frequently escape my mind. <i class="fab fa-python"></i>
slug: python-gists
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

`pandas` pipes
```bash
from functools import wraps

def logger(f):
    @wraps(f)
    def wrapper(*args, **kwargs):
        res = f(*args, **kwargs)
        print(f"Step: {f.__name__}, shape={res.shape}")
        return res
    return wrapper
    
@logger
def init_pipe(df):
    return df.copy()

@logger
def func_name(df):
    ...
    return df
    
df.pipe(init_pipe).pipe(func_name)
```
