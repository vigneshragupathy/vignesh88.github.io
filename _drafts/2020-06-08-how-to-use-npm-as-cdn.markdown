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
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.

See `npm help json` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg>` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
package name: (npm) vikki-tools
version: (1.0.0) 1.0.7
description: Libraries for https://tools.vikki.in
entry point: (index.js) dist/index.js
test command: 
git repository: https://github.com/vignesh88/tools.git
keywords: vikki, tools
author: Vignesh Ragupathy
license: (ISC) 
About to write to /home/vikki/npm/package.json:

{
  "name": "vikki-tools",
  "version": "1.0.7",
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


Is this OK? (yes) yes

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
npm notice 
npm notice package: vikki-tools@1.0.7
npm notice === Tarball Contents === 
npm notice 1.1kB   dist/admin/img/LICENSE                                  
npm notice 8.4kB   dist/admin/css/autocomplete.css                         
npm notice 16.4kB  dist/admin/css/base.css                                 
npm notice 698B    dist/base64/css/base64_dark.css                         
npm notice 159B    dist/base64/css/base64_light.css                        
npm notice 7.8kB   dist/epoch/css/bootstrap-datetimepicker.min.css         
npm notice 11.2kB  dist/epoch/css/bootstrap-select.min.css                 
npm notice 187.1kB dist/common/css/bootstrap.css                           
npm notice 199.4kB dist/common/css/bootstrap.min.css                       
npm notice 6.2kB   dist/admin/css/changelists.css                          
npm notice 2.1kB   dist/common/css/dark.css                                
npm notice 1.5kB   dist/common/css/dark.min.css                            
npm notice 412B    dist/admin/css/dashboard.css                            
npm notice 1.2kB   dist/epoch/css/epoch_dark.css                           
npm notice 1.0kB   dist/epoch/css/epoch_light.css                          
npm notice 423B    dist/admin/css/fonts.css                                
npm notice 8.5kB   dist/admin/css/forms.css                                
npm notice 3.1kB   dist/general/css/general_dark.css                       
npm notice 3.1kB   dist/general/css/general_light.css                      
npm notice 617B    dist/geoip/css/geoip_dark.css                           
npm notice 602B    dist/geoip/css/geoip_light.css                          
npm notice 2.0kB   dist/common/css/light.css                               
npm notice 1.5kB   dist/common/css/light.min.css                           
npm notice 1.2kB   dist/admin/css/login.css                                
npm notice 340.4kB dist/common/css/mdb_dark.css                            
npm notice 340.1kB dist/common/css/mdb_light.css                           
npm notice 327.3kB dist/common/css/mdb.css                                 
npm notice 6.4kB   dist/password_generator/css/password_generator_dark.css 
npm notice 6.3kB   dist/password_generator/css/password_generator_light.css
npm notice 1.9kB   dist/admin/css/responsive_rtl.css                       
npm notice 18.1kB  dist/admin/css/responsive.css                           
npm notice 3.8kB   dist/admin/css/rtl.css                                  
npm notice 17.6kB  dist/admin/css/vendor/select2/select2.css               
npm notice 15.2kB  dist/admin/css/vendor/select2/select2.min.css           
npm notice 10.3kB  dist/admin/css/widgets.css                              
npm notice 1.1kB   dist/general/img/favicon.ico                            
npm notice 6.8kB   dist/admin/js/actions.js                                
npm notice 3.2kB   dist/admin/js/actions.min.js                            
npm notice 901B    dist/admin/js/vendor/select2/i18n/af.js                 
npm notice 941B    dist/admin/js/vendor/select2/i18n/ar.js                 
npm notice 1.1kB   dist/admin/js/autocomplete.js                           
npm notice 761B    dist/admin/js/vendor/select2/i18n/az.js                 
npm notice 1.0kB   dist/base64/js/base64.js                                
npm notice 992B    dist/admin/js/vendor/select2/i18n/bg.js                 
npm notice 1.3kB   dist/admin/js/vendor/select2/i18n/bn.js                 
npm notice 38.5kB  dist/epoch/js/bootstrap-datetimepicker.min.js           
npm notice 52.6kB  dist/epoch/js/bootstrap-select.min.js                   
npm notice 67.7kB  dist/common/js/bootstrap.bundle.min.js                  
npm notice 118.9kB dist/general/js/bootstrap.js                            
npm notice 49.0kB  dist/general/js/bootstrap.min.js                        
npm notice 995B    dist/admin/js/vendor/select2/i18n/bs.js                 
npm notice 934B    dist/admin/js/vendor/select2/i18n/ca.js                 
npm notice 7.8kB   dist/admin/js/calendar.js                               
npm notice 409B    dist/admin/js/cancel.js                                 
npm notice 712B    dist/admin/js/change_form.js                            
npm notice 45.4kB  dist/epoch/js/chosen.jquery.js                          
npm notice 2.2kB   dist/admin/js/collapse.js                               
npm notice 1.1kB   dist/admin/js/collapse.min.js                           
npm notice 566B    dist/common/js/common.js                                
npm notice 5.7kB   dist/admin/js/core.js                                   
npm notice 1.3kB   dist/admin/js/vendor/select2/i18n/cs.js                 
npm notice 864B    dist/admin/js/vendor/select2/i18n/da.js                 
npm notice 6.9kB   dist/common/js/darkmode-js.min.js                       
npm notice 20.2kB  dist/admin/js/admin/DateTimeShortcuts.js                
npm notice 915B    dist/admin/js/vendor/select2/i18n/de.js                 
npm notice 1.1kB   dist/admin/js/vendor/select2/i18n/dsb.js                
npm notice 1.2kB   dist/admin/js/vendor/select2/i18n/el.js                 
npm notice 879B    dist/admin/js/vendor/select2/i18n/en.js                 
npm notice 5.2kB   dist/epoch/js/epoch.js                                  
npm notice 956B    dist/admin/js/vendor/select2/i18n/es.js                 
npm notice 831B    dist/admin/js/vendor/select2/i18n/et.js                 
npm notice 902B    dist/admin/js/vendor/select2/i18n/eu.js                 
npm notice 1.1kB   dist/admin/js/vendor/select2/i18n/fa.js                 
npm notice 839B    dist/admin/js/vendor/select2/i18n/fi.js                 
npm notice 946B    dist/admin/js/vendor/select2/i18n/fr.js                 
npm notice 1.0kB   dist/general/js/general.js                              
npm notice 666B    dist/general/js/general.min.js                          
npm notice 113B    dist/geoip/js/geoip.js                                  
npm notice 948B    dist/admin/js/vendor/select2/i18n/gl.js                 
npm notice 1.0kB   dist/admin/js/vendor/select2/i18n/he.js                 
npm notice 1.2kB   dist/admin/js/vendor/select2/i18n/hi.js                 
npm notice 892B    dist/admin/js/vendor/select2/i18n/hr.js                 
npm notice 1.1kB   dist/admin/js/vendor/select2/i18n/hsb.js                
npm notice 867B    dist/admin/js/vendor/select2/i18n/hu.js                 
npm notice 1.1kB   dist/admin/js/vendor/select2/i18n/hy.js                 
npm notice 22.9kB  dist/common/js/iconify.min.js                           
npm notice 804B    dist/admin/js/vendor/select2/i18n/id.js                 
npm notice 566B    dist/index.js                                           
npm notice 13.8kB  dist/admin/js/inlines.js                                
npm notice 4.8kB   dist/admin/js/inlines.min.js                            
npm notice 833B    dist/admin/js/vendor/select2/i18n/is.js                 
npm notice 937B    dist/admin/js/vendor/select2/i18n/it.js                 
npm notice 917B    dist/admin/js/vendor/select2/i18n/ja.js                 
npm notice 363B    dist/admin/js/jquery.init.js                            
npm notice 280.4kB dist/admin/js/vendor/jquery/jquery.js                   
npm notice 88.1kB  dist/admin/js/vendor/jquery/jquery.min.js               
npm notice 95.9kB  dist/common/js/jquery.min.js                            
npm notice 1.5kB   dist/common/js/js.cookie.min.js                         
npm notice 12.1kB  dist/epoch/js/jstz.min.js                               
npm notice 1.3kB   dist/admin/js/vendor/select2/i18n/ka.js                 
npm notice 1.1kB   dist/admin/js/vendor/select2/i18n/km.js                 
npm notice 910B    dist/admin/js/vendor/select2/i18n/ko.js                 
npm notice 975B    dist/admin/js/vendor/select2/i18n/lt.js                 
npm notice 930B    dist/admin/js/vendor/select2/i18n/lv.js                 
npm notice 1.1kB   dist/admin/js/vendor/select2/i18n/mk.js                 
npm notice 44.1kB  dist/epoch/js/moment-timezone-with-data-2012-2022.js    
npm notice 14.7kB  dist/epoch/js/moment-timezone.js                        
npm notice 51.6kB  dist/epoch/js/moment.min.js                             
npm notice 847B    dist/admin/js/vendor/select2/i18n/ms.js                 
npm notice 814B    dist/admin/js/vendor/select2/i18n/nb.js                 
npm notice 1.4kB   dist/admin/js/vendor/select2/i18n/ne.js                 
npm notice 952B    dist/admin/js/vendor/select2/i18n/nl.js                 
npm notice 3.3kB   dist/password_generator/js/password_generator.js        
npm notice 987B    dist/admin/js/vendor/select2/i18n/pl.js                 
npm notice 569B    dist/admin/js/popup_response.js                         
npm notice 495B    dist/admin/js/prepopulate_init.js                       
npm notice 1.5kB   dist/admin/js/prepopulate.js                            
npm notice 379B    dist/admin/js/prepopulate.min.js                        
npm notice 1.1kB   dist/admin/js/vendor/select2/i18n/ps.js                 
npm notice 911B    dist/admin/js/vendor/select2/i18n/pt-BR.js              
npm notice 917B    dist/admin/js/vendor/select2/i18n/pt.js                 
npm notice 6.9kB   dist/admin/js/admin/RelatedObjectLookups.js             
npm notice 973B    dist/admin/js/vendor/select2/i18n/ro.js                 
npm notice 1.2kB   dist/admin/js/vendor/select2/i18n/ru.js                 
npm notice 167.5kB dist/admin/js/vendor/select2/select2.full.js            
npm notice 76.7kB  dist/admin/js/vendor/select2/select2.full.min.js        
npm notice 5.8kB   dist/admin/js/SelectBox.js                              
npm notice 12.3kB  dist/admin/js/SelectFilter2.js                          
npm notice 1.3kB   dist/admin/js/vendor/select2/i18n/sk.js                 
npm notice 949B    dist/admin/js/vendor/select2/i18n/sl.js                 
npm notice 938B    dist/admin/js/vendor/select2/i18n/sq.js                 
npm notice 1.1kB   dist/admin/js/vendor/select2/i18n/sr-Cyrl.js            
npm notice 1.0kB   dist/admin/js/vendor/select2/i18n/sr.js                 
npm notice 6.2kB   dist/common/js/superplaceholder.js                      
npm notice 841B    dist/admin/js/vendor/select2/i18n/sv.js                 
npm notice 1.1kB   dist/admin/js/vendor/select2/i18n/th.js                 
npm notice 827B    dist/admin/js/vendor/select2/i18n/tk.js                 
npm notice 831B    dist/admin/js/vendor/select2/i18n/tr.js                 
npm notice 1.2kB   dist/admin/js/vendor/select2/i18n/uk.js                 
npm notice 8.9kB   dist/admin/js/urlify.js                                 
npm notice 851B    dist/admin/js/vendor/select2/i18n/vi.js                 
npm notice 128.8kB dist/admin/js/vendor/xregexp/xregexp.js                 
npm notice 62.5kB  dist/admin/js/vendor/xregexp/xregexp.min.js             
npm notice 823B    dist/admin/js/vendor/select2/i18n/zh-CN.js              
npm notice 762B    dist/admin/js/vendor/select2/i18n/zh-TW.js              
npm notice 540B    package.json                                            
npm notice 326.6kB dist/general/js/bootstrap.bundle.js.map                 
npm notice 1.1kB   dist/admin/css/vendor/select2/LICENSE-SELECT2.md        
npm notice 1.1kB   dist/admin/js/vendor/select2/LICENSE.md                 
npm notice 2.1kB   README.md                                               
npm notice 7.3kB   dist/common/js/dark-mode-toggle.min.mjs                 
npm notice 1.2kB   dist/general/img/favicon.png                            
npm notice 33.2kB  dist/common/img/search.png                              
npm notice 87.1kB  dist/general/img/vignesh_ragupathy.png                  
npm notice 1.1kB   dist/admin/img/calendar-icons.svg                       
npm notice 568B    dist/password_generator/img/copy-full.svg               
npm notice 331B    dist/admin/img/icon-addlink.svg                         
npm notice 504B    dist/admin/img/icon-alert.svg                           
npm notice 1.1kB   dist/admin/img/icon-calendar.svg                        
npm notice 380B    dist/admin/img/icon-changelink.svg                      
npm notice 677B    dist/admin/img/icon-clock.svg                           
npm notice 392B    dist/admin/img/icon-deletelink.svg                      
npm notice 560B    dist/admin/img/icon-no.svg                              
npm notice 655B    dist/admin/img/icon-unknown-alt.svg                     
npm notice 655B    dist/admin/img/icon-unknown.svg                         
npm notice 581B    dist/admin/img/icon-viewlink.svg                        
npm notice 436B    dist/admin/img/icon-yes.svg                             
npm notice 560B    dist/admin/img/inline-delete.svg                        
npm notice 1.4kB   dist/common/img/moon_1.svg                              
npm notice 333B    dist/common/img/moon.svg                                
npm notice 1.1kB   dist/admin/img/gis/move_vertex_off.svg                  
npm notice 1.1kB   dist/admin/img/gis/move_vertex_on.svg                   
npm notice 1.2kB   dist/password_generator/img/regenerate.svg              
npm notice 458B    dist/admin/img/search.svg                               
npm notice 3.3kB   dist/admin/img/selector-icons.svg                       
npm notice 1.1kB   dist/admin/img/sorting-icons.svg                        
npm notice 2.0kB   dist/common/img/sun_1.svg                               
npm notice 573B    dist/common/img/sun.svg                                 
npm notice 331B    dist/admin/img/tooltag-add.svg                          
npm notice 280B    dist/admin/img/tooltag-arrowright.svg                   
npm notice 11.6kB  dist/admin/fonts/LICENSE.txt                            
npm notice 1.1kB   dist/admin/js/vendor/jquery/LICENSE.txt                 
npm notice 1.1kB   dist/admin/js/vendor/xregexp/LICENSE.txt                
npm notice 214B    dist/admin/fonts/README.txt                             
npm notice 319B    dist/admin/img/README.txt                               
npm notice 86.2kB  dist/admin/fonts/Roboto-Bold-webfont.woff               
npm notice 85.7kB  dist/admin/fonts/Roboto-Light-webfont.woff              
npm notice 85.9kB  dist/admin/fonts/Roboto-Regular-webfont.woff            
npm notice === Tarball Details === 
npm notice name:          vikki-tools                             
npm notice version:       1.0.7                                   
npm notice package size:  1.1 MB                                  
npm notice unpacked size: 3.9 MB                                  
npm notice shasum:        a9153c3a9bb68bc34d5040d2088a5b95a256e4cc
npm notice integrity:     sha512-zynWl1/pL0Wvk[...]k3yhkCzBz7+0A==
npm notice total files:   188                                     
npm notice 
+ vikki-tools@1.0.7
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