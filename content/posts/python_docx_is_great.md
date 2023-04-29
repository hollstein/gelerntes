---
title: "python-docx is a great library for writing MS Word files"
date: 2023-04-27T13:25:41+02:00
draft: false
tags: ["python"]
---

Today, I used the great library [python-docx](https://python-docx.readthedocs.io/en/latest/user/install.html) to create a rather complex report in MS Word. Why Word? It was the quickest way to get my results into the hands of colleagues from the business. I was using a not so large LLM locally on my GPU server to generate a report for decision-making around some or our relevant topics. In the mix went the remarkable [transfomrers](https://pypi.org/project/transformers/) library as well as [BERTopic](https://maartengr.github.io/BERTopic/index.html) for clustering. The library is easy to use for getting things done fast. Documentation is a bit lacking, and ChatGPT gets many questions about python-docx wrong. What is possible, but hard, to do is creating links within the document or creating a table of contents.