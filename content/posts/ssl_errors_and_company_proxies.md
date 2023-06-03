---
title: "SSL errors and company proxies"
date: 2023-06-03T13:52:10+02:00
draft: false
tags: ["SSL", "python"]
---

Developing AI / ML application behind company IT infrastructure can be challenging. When developing prototypes or quickly checking the latest models from the Huggingface hub, I'm often struck by SSL errors. Often, the only way out for the moment it to disable SSL locally for specific actions, also: Don't do this outside of quick experiments or in production systems.

An excellent resource for dealing with these errors  is this Stack Overflow thread: https://stackoverflow.com/questions/15445981/how-do-i-disable-the-security-certificate-check-in-python-requests

Just for reference, I'm reproducing Blenders answer here and show how to use it. What makes this one so good, is that it implements a solution as a context manager which isolates turning off SSL to specific code blocks:

```python
import warnings
import contextlib

import requests
from urllib3.exceptions import InsecureRequestWarning

old_merge_environment_settings = requests.Session.merge_environment_settings

@contextlib.contextmanager
def no_ssl_verification():
    opened_adapters = set()

    def merge_environment_settings(self, url, proxies, stream, verify, cert):
        # Verification happens only once per connection so we need to close
        # all the opened adapters once we're done. Otherwise, the effects of
        # verify=False persist beyond the end of this context manager.
        opened_adapters.add(self.get_adapter(url))

        settings = old_merge_environment_settings(self, url, proxies, stream, verify, cert)
        settings['verify'] = False

        return settings

    requests.Session.merge_environment_settings = merge_environment_settings

    try:
        with warnings.catch_warnings():
            warnings.simplefilter('ignore', InsecureRequestWarning)
            yield
    finally:
        requests.Session.merge_environment_settings = old_merge_environment_settings

        for adapter in opened_adapters:
            try:
                adapter.close()
            except:
                pass
```

An example of using transformers:

```python
from transformers import AutoTokenizer, AutoModelForSeq2SeqLM

with no_ssl_verification():
    tokenizer = AutoTokenizer.from_pretrained("[MODEL NAME]")
    model = AutoModelForSeq2SeqLM.from_pretrained("[MODEL NAME]")
```