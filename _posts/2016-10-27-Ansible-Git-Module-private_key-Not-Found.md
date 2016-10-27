---
title: 'Ansible Git Module - Private Key Not Found'
author: Tyler
layout: post
permalink: /ansible-git-module-private-key-not-found
categories:
  - Posts
---

Ansible's [git module](http://docs.ansible.com/ansible/git_module.html) fails when given a relative path for `key_file`. The [realpath](http://docs.ansible.com/ansible/playbooks_filters.html#other-useful-filters) jinga2 filter solves this problem.

{% highlight yaml %}
- name: Deploy using Git over SSH
  git: repo=ssh://git@example.com/foo.git
       dest=/bar
       version=master
       key_file={% raw %}{{ "ssh_keys/id_rsa" | realpath }}{% endraw %}
       accept_hostkey=true
       force=yes
{% endhighlight %}
