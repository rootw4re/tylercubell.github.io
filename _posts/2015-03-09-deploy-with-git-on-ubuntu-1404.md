---
title: 'Deploy with Git on Ubuntu 14.04'
author: Tyler
layout: post
permalink: /deploy-with-git-on-ubuntu-1404
categories:
  - Posts
---
This method will let you deploy with Git over HTTP on a Ubuntu 14.04 system.

Prerequisites:

Install git:

{% highlight bash %}
sudo apt-get update
sudo apt-get install git
git config --global user.name "Your Name"
git config --global user.email "youremail@domain.com"
{% endhighlight %}

Install apache:

{% highlight bash %}
sudo apt-get install apache2
sudo a2enmod cgi alias env
sudo service apache2 restart
{% endhighlight %}

Configure apache:

{% highlight bash %}
cd /etc/apache2/sites-available/000-default.conf
{% endhighlight %}

Add this to the default configuration:

{% highlight apache %}
<VirtualHost *:80>
   ServerName git.yourdomain.com
   DocumentRoot /var/www/git
   SetEnv GIT_PROJECT_ROOT /var/www/git
   SetEnv GIT_HTTP_EXPORT_ALL
   SetEnv REMOTE_USER=$REDIRECT_REMOTE_USER
   ScriptAliasMatch \
      "(?x)^/(.*/(HEAD | \
          info/refs | \
          objects/(info/[^/]+ | \
          [0-9a-f]{2}/[0-9a-f]{38} | \
          pack/pack-[0-9a-f]{40}\.(pack|idx)) | \
          git-(upload|receive)-pack))$" \
       "/usr/lib/git-core/git-http-backend/$1"
   Alias /git /var/www/git
   <Directory /usr/lib/git-core>
      Options +ExecCGI -MultiViews +SymLinksIfOwnerMatch
      AllowOverride None
      Require all granted
   </Directory>
   <LocationMatch "^/.*$">
      AuthType Basic
      AuthName "Git"
      AuthUserFile /etc/apache2/passwd.git
      Require valid-user
   </LocationMatch>
</VirtualHost>
{% endhighlight %}

{% highlight bash %}
htpasswd -c /etc/apache2/passwd.git yourname
cd /var/www
mkdir git && cd git
mkdir yourproject && cd yourproject
git init --bare --shared
git update-server-info
cd hooks
cat post-receieve

#!/bin/sh
git --work-tree=/var/www/yourprojectwww --git-dir=/var/www/git/yourproject.git checkout -f

(Hit Ctrl+D to end file)
chmod +x post-receieve
sudo service apache2 restart
{% endhighlight %}

Configure your Git client. I use GitHub for Windows.

Create a new repo.
Go to Repository Settings.
In 'Primary Remote (origin)' set it to:
http://git.yourdomain.com/yourproject.git.

Don't forget to create a DNS record for the git subdomain.


Sources:

[http://stackoverflow.com/questions/26734933/how-to-set-up-git-over-http](http://stackoverflow.com/questions/26734933/how-to-set-up-git-over-http)

[http://git-scm.com/docs/git-http-backend](http://git-scm.com/docs/git-http-backend)

[https://www.kernel.org/pub/software/scm/git/docs/git-http-backend.html](https://www.kernel.org/pub/software/scm/git/docs/git-http-backend.html)

[http://git-scm.com/2010/03/04/smart-http.html](http://git-scm.com/2010/03/04/smart-http.html)

[http://stackoverflow.com/questions/4372306/git-push-over-http-not-activating-remote-hooks](http://stackoverflow.com/questions/4372306/git-push-over-http-not-activating-remote-hooks)
