---
layout: post
title: How to use npm as CDN
date: '2020-06-08 12:15:00'
tags:
- kubernetes
- linux
- opensource
author: Vignesh Ragupathy
categories: [PHP, Laravel]
---

I been using AWS to host my personal blog for few years now. Orinally my blog was hosted in wordpress and then i migrated to ghost. My blog is running in ghost for last 2 years already. I thought of exploring new hosting option and finally decided to migrate to Jekyll. 

Jekyll is a ruby based static blog and it has an advantage of hosting in github. The letsencrypt SSL certificate is also provided by github for my custom domain so i don't need to worry about managing it.

I also created a seperate website to showcase the opensource tools i am developing. This also available in github and anyone can fork my project. This project is running in Django framework as containerized application. One of the challenges in Django application is hosting your static content. Django does not support static content in production and we should use a proxy like Nginx to server its static content.

I use my nginx proxy to serve the static content. But due to performance reason , i started to explore the free CDN to serve my static content


{% highlight console %}
location /static/ {
        root /tools.vikki.in/static;
    }
{% endhighlight %}

unpkg and jsdeliver are global CDN and they can be used to deliver any pacakges hosted in NPN

### Step 1
Create a directory 
mkdir npn
mkdir npm/dist
cd npm
### Step 2
Create a package.json file for your pacakage
npm init

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

### Step 3
Create a index.js

vim dist/index.js 

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

{% highlight console %}
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

{% highlight console %}
#npm login
#npm publish
{% endhighlight %}

unpkg.com/:package@:version/:file
<script src="https://unpkg.com/selector-library@1.0.6/selector.js"></script>
<script src="https://unpkg.com/selector-library"></script>
<script src="https://cdn.jsdelivr.net/npm/package-name"></script>
https://cdn.jsdelivr.net/npm/vikki-tools@1.0.3/dist/index.min.js
https://unpkg.com/vikki-tools/

![unpkg screenshot](/content/images/2020/unpkg_vikki_tools.jpg)