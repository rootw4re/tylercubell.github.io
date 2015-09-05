---
title: Tuning LAMP for WordPress
author: Tyler
layout: post
permalink: /tuning-lamp-for-wordpress/
categories:
  - Posts
---
<img src="http://i.imgur.com/pFXkCXC.jpg">

Here are the things you can do to speed up WordPress on LAMP with the greatest ROI for your time. This guide is sort of a compilation of a bunch of tutorials from around the web. This is exactly what I did and I got WordPress down from an average of 250ms to 30ms. It works great.

## 1. APC

[APC][1] is an opcode cache for PHP. It speeds up your PHP scripts. This is probably the easiest thing you can do to increase performance.

{% highlight bash %}
sudo apt-get install php-pear php5-dev make libpcre3-dev
sudo pecl install apc
sudo nano /etc/php5/apache2/php.ini
Add: 
extension = apc.so
apc.shm_size = 64
{% endhighlight %}

[Original source][2].

## 2. mod_pagespeed

[mod_pagespeed][3] does a lot of little things like shrinking images, css, js, etc. Download [mod_pagespeed][4] from Google.

{% highlight bash %}
sudo dpkg -i mod-pagespeed-*.deb
sudo apt-get -f install
{% endhighlight %}


## 3. Memcached

[memcached][5] speeds up common database queries among other things.

{% highlight bash %}
apt-get install php5-memcache memcached php-pear
pecl install memcache
{% endhighlight %}

## 4. Varnish

[Varnish][6] is a HTTP (only) accelerator a.k.a. reverse proxy that caches web pages. From now on Varnish is going to listen on port 80, and Apache is running on port 8080. Stop Apache and make sure you update any [virtual hosts][7].

{% highlight bash %}
apt-get update
apt-get upgrade
apt-get install varnish
nano /etc/default/varnish
{% endhighlight %}

Change to:


{% highlight bash %}
DAEMON_OPTS="-a :80 \
             -T localhost:81 \
             -f /etc/varnish/default.vcl \
             -S /etc/varnish/secret \
             -s malloc,256m"
{% endhighlight %}

{% highlight bash %}cd /etc/varnish cp default.vcl user.vcl nano user.vcl{% endhighlight %}

This is the configuration that works for me. If this one doesn&#8217;t suit you, see some examples on [GitHub][8].

{% highlight bash %}
backend default {
    .host = "127.0.0.1";
    .port = "8080";
}

acl purge {
    "localhost";
}

sub vcl_recv
{
    # Ignore requests with "X-Forwarded-Proto: https" header
    if (req.http.X-Forwarded-Proto !~ "https") {
        remove req.http.X-Forwarded-For;
        set req.http.X-Forwarded-For = client.ip;
    }

    # Allow purging
    if(req.request == "PURGE"){
        if(!client.ip ~ purge){
            error 405 "Purging not allowed.";
        }
        return(lookup);
    }

    # Set Grace Time
    if (req.backend.healthy) {
        set req.grace = 30s;
    } else {
        set req.grace = 1h;
    }

    # Do not cache POST requests
    if (req.request == "POST")
    {
        return (pass);
    }

    # phpMyAdmin is ok
    if (req.url ~ "phpmyadmin") {
        return(pass);
    }

    # Don't served cached pages to logged in users
    if (req.http.Cookie ~ "(comment_author|wordpress_[a-f0-9]+|wp-postpass|wordpress_logged_in)") {
        return (pass);
    }

    # Drop received cookies not related to sessions
    if (!(req.url ~ "(login|logout|log-in|log-out|signin|signout|sign-in|sign-out|admin)")) {
        unset req.http.cookie;
    }
}

sub vcl_fetch
{
    if (req.url ~ "phpmyadmin") {
       return(hit_for_pass);
    }
    
    # Drop cookies not related to sessions
    if (!(req.url ~ "(login|logout|log-in|log-out|signin|signout|sign-in|sign-out|admin)")) {
        unset beresp.http.set-cookie;
    }

    # Set the TTL for cache object to one hour
    set beresp.ttl = 60m;

    # Set Grace Time to one hour
    set beresp.grace = 1h;
}

sub vcl_deliver {
    if (obj.hits &gt; 0) {
        set resp.http.X-Cache = "cached";
    } else {
        set resp.http.x-Cache = "uncached";
    }

    # Remove some headers: PHP version
    unset resp.http.X-Powered-By;

    # Remove some headers: Apache version & OS
    unset resp.http.Server;
    unset resp.http.X-Varnish;
    unset resp.http.Via;
    unset resp.http.Link;

    return (deliver);
}

sub vcl_miss {
    if(req.request == "PURGE"){
        purge;
        error 200 "Purged";
    }
}
 
sub vcl_hit {
    if(req.request == "PURGE"){
        purge;
        error 200 "Purged";
    }
}

sub vcl_init {
    return (ok);
}

sub vcl_fini {
    return (ok);
}
{% endhighlight %}

[Original source.][9]

## 5. Pound

If you&#8217;re running any sites with SSL, you need [Pound][10]. Pound is an SSL [terminator][11] and reverse proxy. You need it because Varnish doesn&#8217;t do SSL. In this configuration, Pound listens on port 443 (HTTPS), sends traffic to Varnish on port 80 (HTTP), takes whatever Varnish gives it, and sends it back out over SSL.

{% highlight bash %}
apt-get install pound
nano /etc/default/pound
Add:
startup=1
nano /etc/pound/pound.cfg
{% endhighlight %}

Change pound.cfg to:

{% highlight bash %}
User            "www-data"
Group           "www-data"
 
ListenHTTPS
        Address 1.2.3.4 # Your Public IP
        Port 443
        Cert "location of your .pem file"
        HeadRemove "X-Forwarded-Proto"
        AddHeader "X-Forwarded-Proto: https"
        Service
                BackEnd
                        Address 127.0.0.1
                        Port 80
                End
        End
End
{% endhighlight %}

To make a .pem file if you have your private key, cert, and CA bundle do this:

{% highlight bash %}
cat a.key b.crt c.bundle &gt; z.pem
{% endhighlight %}

Now restart everything.

{% highlight bash %}
service memcached restart
service apache2 restart
service mysql restart
service varnish restart
service pound restart
curl -silent -X PURGE http://localhost:80 &gt; /dev/null
{% endhighlight %}

## 6. Varnish HTTP Purge Plugin

Install this [plugin][12] for WordPress. It purges Varnish every time you edit or create a new post.

 [1]: http://php.net/manual/en/book.apc.php
 [2]: https://www.digitalocean.com/community/tutorials/how-to-install-alternative-php-cache-apc-on-a-cloud-server-running-ubuntu-12-04
 [3]: https://developers.google.com/speed/pagespeed/module
 [4]: https://developers.google.com/speed/pagespeed/module/download
 [5]: http://php.net/manual/en/book.memcached.php
 [6]: https://www.varnish-cache.org/
 [7]: http://httpd.apache.org/docs/2.2/vhosts/
 [8]: https://github.com/mattiasgeniar/varnish-3.0-configuration-templates
 [9]: https://www.linode.com/docs/websites/varnish/getting-started-with-varnish-cache
 [10]: http://en.wikipedia.org/wiki/Pound_%28networking%29
 [11]: http://i.imgur.com/OM0CSsm.jpg
 [12]: https://wordpress.org/plugins/varnish-http-purge/