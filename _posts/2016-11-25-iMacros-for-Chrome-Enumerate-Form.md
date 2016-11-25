---
title: 'iMacros for Chrome - Enumerate Form'
author: Tyler
layout: post
permalink: /imacros-for-chrome-enumerate-form
categories:
  - Posts
---

This iMacro takes the current loop number, zerofills it, inserts the value into a text box, and then submits the form. It could be useful in testing for [enumeration vulnerabilities](https://www.owasp.org/index.php/Testing_for_User_Enumeration_and_Guessable_User_Account_(OWASP-AT-002)). Requires [iMacros for Chrome](https://chrome.google.com/webstore/detail/imacros-for-chrome/cplklnmnlbnpmjogncfgfijoopmnlemp?hl=en).

```
TAG POS=1 TYPE=INPUT:TEXT FORM=ID:form_id ATTR=ID:input_text_id CONTENT=EVAL("('0000' + {% raw %}{{!LOOP}}{% endraw %} ).slice(-4);")
TAG POS=1 TYPE=INPUT:SUBMIT FORM=ID:form_id ATTR=ID:input_submit_id
WAIT SECONDS=1
```

If zerofilling isn't necessary, replace the text after `CONTENT=` with `{% raw %}{{!LOOP}}{% endraw %}`.
