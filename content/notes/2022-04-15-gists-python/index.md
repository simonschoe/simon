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
```python
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

logging with `rich`
```python
import logging
from rich.logging import RichHandler

logger = logging.getLogger(__name__)

# specify handlers
cons_handler = RichHandler()
file_handler = logging.FileHandler("debug.log")

# set logging levels
logger.setLevel(logging.DEBUG)
cons_handler.setLevel(logging.WARNING)
file_handler.setLevel(logging.DEBUG)

# format logs
fmt_cons = '%(message)s'
fmt_file = '%(levelname)s %(asctime)s [%(filename)s:%(funcName)s:%(lineno)d] %(message)s'
cons_handler.setFormatter(logging.Formatter(fmt_cons))
file_handler.setFormatter(logging.Formatter(fmt_file))

# add handlers to logger
logger.addHandler(cons_handler)
logger.addHandler(file_handler)
```

Save cookies for use via `selenium` or `requests`
```python
# save cookies
with open("cookie", 'wb') as f:
    pickle.dump(driver.get_cookies(), f)
    
# load cookies (selenium)
with open("cookie", 'rb') as f:
    cookies = pickle.load(f)
    for cookie in cookies:
        driver.add_cookie(cookie)
        
# load cookies (requests)
cookie_jar = requests.cookies.RequestsCookieJar()
with open("cookie", 'rb') as f:
    cookies = pickle.load(f)
    for cookie in cookies:
        cookie_jar.update({cookie['name']: cookie['value']})
        
requests.get(url, cookies=cookie_jar)
```




