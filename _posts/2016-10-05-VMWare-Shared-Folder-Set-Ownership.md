---
title: 'VMWare Shared Folder Set Ownership'
author: Tyler
layout: post
permalink: /vmware-shared-folder-set-ownership
categories:
  - Posts
---

The problem with VMWare shared folders is that you can't specify the owner/group so the default ends up being `root` for both. This isn't ideal, for instance, if we're sharing web files that need to be owned by Apache in order to be served. Piggybacking off a comment from this [post](http://viraj-workstuff.blogspot.com/2013/07/vmware-fusion-permissions-on-shared.html), here's how to specify the owner/group of a shared folder:

1. Ensure [VMWare Tools](https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=1022525) is installed.
2. Execute:

{% highlight bash %}
    sudo echo ".host:/ /mnt/hgfs vmhgfs rw,uid=33,gid=33" >> /etc/fstab
    sudo umount /mnt/hgfs
    sudo mount /mnt/hgfs
{% endhighlight %}
    
What this does is append `/etc/fstab` to specify how `/mnt/hgfs` should be mounted in terms of ownership. `uid` and `gid` `33` is Apache's `www-data` user on Debian-based distrobutions. `/mnt/hgfs` is then re-mounted with new ownership.
