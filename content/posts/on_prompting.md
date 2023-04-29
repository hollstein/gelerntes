---
title: "On prompting"
date: 2023-04-29T16:56:41+02:00
draft: false
tags: ["python", "llm"]
---
 
Prompting instruction fine-tuned LLMs allows getting NLP tasks done deemed infeasible some years ago and prompt hacking (trial and error until good enough on some examples) and prompt engineering (same as hacking but with tracking and metrics) become an interesting programming paradigm. 

## What works for me

### Models

A good publicly available model is an instruction fine-tuned flan-t5 [model](https://huggingface.co/declare-lab/flan-alpaca-gpt4-xl) from the hub. This space evolves fast, I consider this outdated after Q2/2023.

### Creating prompts

Delimiters are the clean code version from prompting. I found that they will not be the deal maker or breaker, but the code creating the prompt looks so much nicer and cleaner this way.

```python
prompt = f"""
[Detailed and specific task description] the text delimited by triple backticks.
```{text}```
"""
```


## Resources

- Video lectures by [deeplearing.ai](https://learn.deeplearning.ai/chatgpt-prompt-eng/lesson/1/introduction): Good intro, videos better watched on 2x, nice demo notebooks
- [dair-ai](https://github.com/dair-ai/Prompt-Engineering-Guide) open book on github
- [Awesome-Prompt-Engineering](https://github.com/promptslab/Awesome-Prompt-Engineering) GitHub list

