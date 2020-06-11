---
layout: post
title: How to use npm as CDN
date: '2020-06-08 12:15:00'
tags:
- linux
- opensource
author: Vignesh Ragupathy
---

![](/content/images/cover/npn.jpg)
*Photo by [Vignesh Ragupathy](https://photography.vikki.in/vikki-photography-budapest-3){:target="_blank"}*

I been using AWS to host my personal blog for few years now. Orinally my blog was hosted in wordpress and then i migrated to ghost. My blog is running in ghost for last 2 years already. I thought of exploring new hosting option and finally decided to migrate to Jekyll.

Jekyll is a ruby based static blog and it has an advantage of hosting in github. The letsencrypt SSL certificate is also provided by github for my custom domain so i don't need to worry about managing it.

I also created a seperate website to showcase the opensource tools i am developing. This also available in github and anyone can fork my project. This project is running in Django framework as containerized application. One of the challenges in Django application is hosting your static content. Django does not support static content in production and we should use a proxy like Nginx to server its static content.

I use my nginx proxy to serve the static content. But due to performance reason , i started to explore the free CDN to serve my static content

Below is the nginx configuration snippet for mapping static content.

{% highlight console %}
location /static/ {
        root /tools.vikki.in/static;
    }
{% endhighlight %}

> unpkg and jsdeliver are global CDN and they can be used to deliver any pacakges hosted in NPN

After doing the reasearch i decided to use unpkg and jsdelivr for my website.
[unpkg](https://unpkg.com/){:target="_blank"} and [jsdelivr](https://www.jsdelivr.com/){:target="_blank"} both provides CDN for the content hosted in NPN.
So first we should have the static content published in [NPN](https://www.npmjs.com/){:target="_blank"}.

## NPN Pacakage creation

### 1. Create the directory for adding packages for NPN

{% highlight bash %}
mkdir npn
mkdir npm/dist
cd npm
{% endhighlight %}

### 2. Create a package.json file for your pacakage

{% highlight bash %}
npm init
{% endhighlight %}

{% highlight json %}
{
  "name": "vikki-tools",
  "version": "1.0.3",
  "description": "Libraries for https://tools.vikki.in",
  "main": "dist/index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/vignesh88/tools.git"
  },
  "keywords": [
    "vikki",
    "tools"
  ],
  "author": "Vignesh Ragupathy",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/vignesh88/tools/issues"
  },
  "homepage": "https://github.com/vignesh88/tools#readme"
}
{% endhighlight %}

### 3. Create a index.js

I added a javascript function that will be used to copy text to clipboard.

{% highlight bash %}
vim dist/index.js
{% endhighlight %}

{% highlight javascript %}
function copyToClipboard(x,y) {
    if( document.getElementById(x).value) {
        data_2_copy = document.getElementById(x).value;
    } else {
        data_2_copy = document.getElementById(x).innerText;
    }

    var e = document.createElement("textarea");
    e.style.opacity = "0", e.style.position = "fixed", e.textContent = data_2_copy;
    var t = document.getElementsByTagName("body")[0];
    t.appendChild(e), e.select(), document.execCommand("copy"), t.removeChild(e), $(y).show(), setTimeout((function () {
        $(y).hide()
    }), 1e3)
}
{% endhighlight %}

Now lets copy all our static content to the <mark>dist</mark>  directory.
I have various css,images,javascript that will be used in various app inside my django application.

Below are the files i copied.

{% highlight bash %}
vikki@vikki-ericsson-210319:/mnt/vikki/github/tools/npm$ tree .
.
├── dist
│   ├── admin
│   │   ├── css
│   │   │   ├── autocomplete.css
│   │   │   ├── base.css
│   ├── geoip
│   │   ├── css
│   │   │   ├── geoip_dark.css
│   │   │   └── geoip_light.css
│   │   └── js
│   │       └── geoip.js
│   ├── index.js
│   └── password_generator
│       ├── css
│       │   ├── password_generator_dark.css
│       │   └── password_generator_light.css
│       ├── img
│       │   ├── copy-full.svg
│       │   └── regenerate.svg
│       └── js
│           └── password_generator.js
├── package.json
└── README.md
{% endhighlight %}

Now we are all set, lets connect to NPN and publish our package.

> You should already have an account in NPN to publish.

{% highlight bash %}
npm login
npm publish
{% endhighlight %}

That's it. Now we have the package published in NPN.

> Unpkg and Jsdelivr provides CDN for NPM packages without any configuration.

Lets try to access it using unpkg. Open your browser and enter the url in the below format.

<mark>https://unpkg.com/pacakage/</mark>

For using specific version <mark>https://unpkg.com/package@version/:file</mark>

My package name is *vikki-tools* so the format will be [https://unpkg.com/vikki-tools<mark>/</mark>](https://unpkg.com/vikki-tools/){:target="_blank"}.

> The leading <mark> / </mark> at the end of the URL is important.

![unpkg screenshot](/content/images/2020/unpkg_vikki_tools.jpg)

## Using Unpkg to serve static content in website

We can now load the static content from NPN in our website.

{% highlight html %}
<script src="https://unpkg.com/vikki-tools@1.0.3/dist/index.js"></script>
<link href="https://unpkg.com/vikki-tools@1.0.3/dist/base64/css/base64_dark.css" rel="stylesheet">
{% endhighlight %}

## Using Jsdelivr to serve static content in website

We can also use Jsdelivr instead of unpkg.

{% highlight html %}
<script src="https://cdn.jsdelivr.net/npm/vikki-tools@1.0.3/dist/index.js"></script>
<link href="https://cdn.jsdelivr.net/npm/vikki-tools@1.0.3/dist/base64/css/base64_dark.css" rel="stylesheet">
{% endhighlight %}

## Auto minified version from jsdelivr

Jsdelivr also provide the auto minified version of the CSS and Javascript from NPN.
If you want to use minified version css and js, just add  <mark>.min</mark> extension to the filename

{% highlight html %}
<script src="https://cdn.jsdelivr.net/npm/vikki-tools@1.0.3/dist/index.min.js"></script>
<link href="https://cdn.jsdelivr.net/npm/vikki-tools@1.0.3/dist/base64/css/base64_dark.min.css" rel="stylesheet">
{% endhighlight %}

## Script to automatically update the static and CDN URL in Django

For ease, i created a script to automatically update all static content in your template directory in Django application.

The code is available in the [Github URL](https://github.com/vignesh88/tools/blob/master/vikki_scripts/update_static_to_cdn.py){:target="_blank"}

## Demo video



<style>.embed-container { position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; } .embed-container iframe, .embed-container object, .embed-container embed { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }</style><div class='embed-container'><iframe src='https://www.youtube.com/embed/XMSV5J4jxSo' frameborder='0' allowfullscreen></iframe></div>

<!--
<object style="width:100%;height:100%;width: 820px; height: 461.25px; float: none; clear: both; margin: 2px auto;" data="https://www.youtube.com/embed/XMSV5J4jxSo">
</object>
<iframe width="560" height="315" src="https://www.youtube.com/embed/XMSV5J4jxSo" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
-->