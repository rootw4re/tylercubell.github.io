---
title: Google Maps Embed API — Not Centering Solution
author: Tyler
layout: post
permalink: /google-maps-embed-api-not-centering-solution/
categories:
  - Posts
---
If you&#8217;re trying to embed a Google Map using their Embed API and getting this result, it may be caused by jQuery.

## Before:

[<img src="http://i.imgur.com/J1rLAS9l.png" alt="" width="500" height="206" />][1]

## After:

[<img src="http://i.imgur.com/uVgBxMXl.png" alt="" width="500" height="182" />][2]

## Solution

Instead of using jQuery to change the CSS display property, try hiding the map using the left property.

{% highlight css %}
.google-map {
  position:relative;
  width:500px;
  height:500px;
}
.google-map iframe {
  position:absolute;
  top:0;
  left:0;
  width:100%;
  height:100%;
}
{% endhighlight %}

{% highlight html %}
<div class="google-map">
    <iframe frameborder="0" style="border:0" src="https://www.google.com/maps/embed/v1/search?key=#Key&q=#Query&zoom=20"></iframe>
</div>
{% endhighlight %}

jQuery:

{% highlight js %}
var map = $('.google-map');

// Buggy: map.css('display', 'none');
map.css('left', '-9999px');

// Buggy: map.css('display', 'block');
map.css('left', '0px');
{% endhighlight %}

&nbsp;


 [1]: http://i.imgur.com/J1rLAS9.png
 [2]: http://i.imgur.com/uVgBxMX.png