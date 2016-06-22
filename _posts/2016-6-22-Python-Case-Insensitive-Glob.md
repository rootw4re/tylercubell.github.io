---
title: 'Python Case Insensitive Glob'
author: Tyler
layout: post
permalink: /python-case-insensitive-glob
categories:
  - Posts
---

Python 2's [glob](https://docs.python.org/2/library/glob.html) by itself is limited in its pattern matching capability but when combined with `regex` it becomes more useful. Surprisingly, I couldn't find anything concise and practical for selecting files with certain extensions in a case-insensitive manner so I wrote this:

```python
#!/usr/bin/env python

import glob, re

for file in [f for f in glob.glob('*') if re.match('^.*\.zip$', f, flags=re.IGNORECASE)]:
    print file
```
