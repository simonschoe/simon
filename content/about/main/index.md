---
## Configure page content in wide column
# use custom css to write `title` using the Typwriter Effect
title: |
  <style>
    .typewriter {
      overflow: hidden; /* Ensures the content is not revealed until the animation */
      border-right: .15em solid orange; /* The typewriter cursor */
      white-space: nowrap; /* Keeps the content on a single line */
      letter-spacing: .0em; /* Adjust as needed */
      animation:
        typing 4s steps(44) 1s 1 normal both,
        blink-caret .75s steps(44) infinite normal;
    }

    /* The typing effect */
    @keyframes typing {
      from { width: 0 }
      to { width: 7.8em } /* Manually set to stop after last character */
    }

    /* The typewriter cursor effect */
    @keyframes blink-caret {
      from, to { border-color: transparent }
      50% { border-color: orange }
    }
  </style>

  <h1 class="f2 f1-m f-headline-l fw5-ns mv4 lh-solid tracked-tight-ns typewriter" style="font-family: inherit;">Hi, I’m Simon. 👋</h1>
number_featured: 2 # pulling from mainSections in config.toml
use_featured: false # if false, use most recent by date
number_categories: 10 # set to zero to exclude
show_intro: true
intro: |
  The solution to many contemporary problems, big or small, begins with the question of *measurement*. My [research](/research) focuses broadly on measurement problems in accounting. These include the prediction of accounting estimates, the analysis of corporate narratives, and the estimation of plausibly causal effects. In my works, I pair traditional quantitative methods with novel techniques from machine learning, natural language processing, interpretable machine learning, and causal machine learning. I am driven by an intense curiosity and approach my work with a [scientific mindset](https://hbr.org/2022/05/act-like-a-scientist). I enjoy deep work and alternating between R and Python to harness the best of both worlds. 
  
  My research explores
  - how machine learning can help managers provide decision-useful accounting estimates and reduce human bias,
  - how natural language processing can be used to meter complex phenomena in firms' capital market communications with financial analysts, and
  - how transfer learning can advance the state-of-the-art in textual analysis in accounting research.
  
  My tech stack includes
  - [pandas](https://pandas.pydata.org/) and the [tidyverse](https://www.tidyverse.org/) for tabular data wrangling,
  - [ggplot2](https://ggplot2.tidyverse.org/) for data visualization,
  - [rvest](https://rvest.tidyverse.org/), [Beautiful Soup](https://beautiful-soup-4.readthedocs.io/en/latest/), and [Selenium](https://selenium-python.readthedocs.io/) for web scraping,
  - [PyMuPDF](https://pymupdf.readthedocs.io/en/latest/module.html), [gensim](https://radimrehurek.com/gensim/), [spacy](https://spacy.io/), and [LLMs](https://github.com/openai/openai-python) for NLP, 
  - [prodigy](https://prodi.gy/) for data annotation,
  - [tidymodels](https://www.tidymodels.org/), [sklearn](https://scikit-learn.org/stable/), and [DALEX](https://dalex.drwhy.ai/) for machine learning,
  - [pytorch](https://pytorch.org/) and [transformers](https://huggingface.co/docs/transformers/index) for deep learning,
  - [Weights & Biases](https://wandb.ai/site) for experiment tracking,
  - [fixest](https://lrberge.github.io/fixest/) and [did](https://bcallaway11.github.io/did/) for empirical modeling,
  - [DoubleML](https://docs.doubleml.org/stable/index.html) for causal machine learning,
  - [gradio](https://www.gradio.app/) for web applications,
  - [rmarkdown](https://rmarkdown.rstudio.com/), [xaringan](https://github.com/yihui/xaringan), and [Jupyter Notebooks](https://jupyter.org/) for literate coding, and
  - [Git](https://git-scm.com/)+[GitHub](https://github.com/) for version control.
  
show_outro: false
outro: 
---

\*\* index doesn't contain a body, just front matter above. See about/list.html in the layouts folder \*\*
