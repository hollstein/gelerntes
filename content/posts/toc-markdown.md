---
title: "Generating a table of content for mardown files"
date: 2023-04-22T13:22:39+02:00
draft: false
tags: ["markdown", "tools"]
ShowToc: false
---

Markdown documents for technical documentation can quickly get out of control and lose their value if one just can't find relevant content. Having a table of content helps to navigate and parse files, even if they become longer and more complex. Markdown itself doesn't create auto-generating and auto-updating table of contents and doing this manual will just not work with real people and real live scenarios where time is always in short supply, especially then documentation is an afterthought to getting features shipped. Automating this is a great way to save time and keep these documents useful. 

One way that worked for me is the tool [markdown-toc](https://github.com/ekalinin/github-markdown-toc), install it via:

```bash
npm install --save markdown-toc
````

Add `<!-- toc -->` to a markdown file (e.g. `Readme.md`) and run:

```bash
npx markdown-toc -i Readme.md
```

on the file. Done.