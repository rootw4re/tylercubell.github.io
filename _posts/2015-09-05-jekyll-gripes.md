---
title: 'Jekyll Gripes'
author: Tyler
layout: post
permalink: /jekyll-gripes
categories:
  - Posts
---
Over the past week or so I launched [jekyllthemes.io](http://www.jekyllthemes.io) and I've been working on a Jekyll theme for sale on Themeforest. The reason why I prefer Jekyll over a more traditional platform, like Wordpress, is it's simplicity. All you need to do to get up and running is pick a theme, change a few configuration variables, and drop some posts into a folder. It can't get any simpler than that.

However, from the perspective of developing a theme designed to work on GitHub pages without any additional plugins, it leaves a few things to be desired.

## 1. Categories Support

It just doesn't work out of the box. The only [solution](http://primalivet.com/2013/11/simple-category-pages-with-vanilla-jekyll/) I came across is having the end-user create a folder for every category which is stupid. Also there might be a few 3rd party plugins but that would kill support for GitHub pages. 

The way it should work is having a `/categories/foo` structure and each post should appear under the `foo` category with pagination support. It's a no brainer.

## 2. Full Liquid Implementation

The problem I ran into was creating a next post preview. When Jekyll creates a `post.excerpt` it will render markdown and strip HTML, but it won't get rid of Liquid tags. There was a solution I was experimenting with that would have worked if Jekyll supported regex in [replace](https://docs.shopify.com/themes/liquid-documentation/filters/string-filters#replace). That didn't work because replace doesn't work, so I moved onto my backup plan, slice. [That also didn't work](https://github.com/jekyll/jekyll/issues/3260) along with a bunch of other filters. Why does it say on the [official Jekyll wiki](http://jekyllrb.com/docs/templates/) that it supports all of the standard Liquid tags and filters then? Isn't replace pretty standard?
