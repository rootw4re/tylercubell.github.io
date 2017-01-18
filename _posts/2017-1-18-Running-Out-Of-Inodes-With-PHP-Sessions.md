---
title: 'Running Out of Inodes With PHP Sessions'
author: Tyler
layout: post
permalink: /running-out-of-inodes-with-php-sessions
categories:
  - Posts
---

As I found out, a little known "feature" of PHP is that a custom location can be set for session files that isn't automatically cleared by a cron job. If the PHP application is [crappy enough](http://www.concrete5.org/) *cough cough*, it won't clear them on its own but blindly trust PHP's garbage collection to do so, which is often times disabled by default -- at least on Debian systems. After a while these session files pile up until all inodes are used up and the application fails. Read more about this [here](http://php.net/manual/en/function.session-save-path.php#98106) and [here](http://www.laurencegellert.com/2012/08/php-sessions-not-being-deleted-on-ubuntu-server/).

Here's how to solve it:

Create a shell script, `clearTempFiles.sh`.

```sh
find /var/www -type f -name "sess_*" -delete
```

Make it executable:

```sh
chmod +x clearTempFiles.sh
```

Create a cronjob. In this example, `clearTempFile.sh` is set to run every Sunday at 4AM. 

```
0 4 * * 0 /home/yourname/clearTempFiles.sh
```

After running this script once, my inode usage dropped from nearly 100% to 20% and over 100k inodes were freed. Alternatively, you could set the `session.gc_probability` in `php.ini`.
