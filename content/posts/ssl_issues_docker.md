---
title: "SSL issues when baking LLMs into docker images"
date: 2023-05-08T18:40:20+02:00
draft: false
tags: ["docker", "ssl"]
---

I'm building some docker images to deploy a data science use-case to AWS as ECS task. One trick to reduce startup-time and external dependencies is to include all needed models into the image. This increases the image size but decreases the time spent getting the models from an external dependency and is safeguarding the service for external resources becoming unavailable. In this project, I can't rely on some other internal way of fetching the models.

This is how I do it for SentenceTransformer:

```python
RUN python -c "import os; os.environ['CURL_CA_BUNDLE'] = ''; from sentence_transformers import SentenceTransformer; SentenceTransformer('sentence-transformers/paraphrase-mpnet-base-v2)"
```


spaCy models

```python
RUN python -m spacy download en_core_web_sm
```

and Hugginface models:

```python
RUN python -c "import os; os.environ['CURL_CA_BUNDLE'] = ''; from transformers import pipeline; model = pipeline(task='text2text-generation', model='declare-lab/flan-alpaca-gpt4-xl'); print(model)"
```

I was fighting SSL errors, and the only way of getting this done was to downgrade `requests<2.27.1` (details [here](https://github.com/huggingface/transformers/issues/17611#issuecomment-1323272726)) and to include `import os; os.environ['CURL_CA_BUNDLE'] = ''`. This is certainly a bad idea in production and should be used with extreme case.