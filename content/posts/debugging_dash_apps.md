---
title: "Debugging dash apps"
date: 2023-04-28T11:47:28+02:00
draft: false
tags: ["python", "dash"]
---

Debugging [Dash apps](https://dash.plotly.com/) can be fun and painful at the same time.

For debugging, I often litter the code with these to have the more powerful IPython available for debugging. 

```python
from IPython import embed; embed()
```

Beware of the auto-update in debug mode. If we run the app in debug mode

```python
app.run(host='0.0.0.0', port=8091, debug=True)
````

we get hot reloading which is super useful. However, if this happens while beeing in a debug window, accessing and killing the process becomes tricky. What works for me is this (adjust port accordingly):

```sh
kill $(lsof -t -i :8091)
```