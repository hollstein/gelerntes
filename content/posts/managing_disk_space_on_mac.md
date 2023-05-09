---
title: "Managing disk space on Mac"
date: 2023-05-08T18:54:05+02:00
draft: false
tags: ["docker", "mac"]
---

Developing date science use cases on a Mac with 256GB disk space is fun compared to doing this on Windows, but the small disk of my Mac can make things challenging. In case the disk is full again:

- `docker system prune --all`, removes all thing's docker, also everything cached, so initial build times are a price to pay 
- If I have no clue why this disk is full: `find . -maxdepth 1 -type d -mindepth 1 -exec du -hs {} \;`
- `pip cache purge`
- `conda clean --all`
- Removing unused software also helps

