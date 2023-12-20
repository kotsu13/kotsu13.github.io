---
layout: post
title:  "Running Jekyll on a DigitalOcean Droplet"
description: "Deploying jekyll onto an Ubuntu VPS, without using automatic deployment tools"
date: 2016-03-21
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [jekyll, digitalocean]
---

One of my first challenges with using Jekyll was trying to deploy a blog on my Digital Ocean droplet. Initially, I was following the guide at <a href="https://www.digitalocean.com/community/tutorials/how-to-get-started-with-jekyll-on-an-ubuntu-vps">this link</a>, only to realise a few problems:

1. The default Capistrano is now version 3, instead of version 2 on the guide. Unfortunately, the core of Capistrano 3 is completely different from Capistrano 2 so none of the instructions can be used. 
2. Even though I eventually tried to download Capistrano 2, i still ran into issues trying to get it to run. 

I spent a few days trying to get my jekyll up and eventually, I realised that I actually didn't need Capistrano. The reason for this is:  While looking through the Jekyll documentation, I came to the realization of the meaning of "static" - whenever `jekyll serve` is executed, it looks through all the layout/posts/etc... folders, and actually generates a full html for everything into the sub-folder `_site/`. I didn't actually care about anything else besides the webpage being updated as I updated my blogposts, so just `jekyll serve` worked plenty fine for me.

### Setup:
1. OS: Ubuntu 
2. Webserver: nginx 

### Steps: 
<ol>
<li> Download nginx and the ruby virtual machine: 
{% highlight shell %}
curl -L https://get.rvm.io | bash -s stable --ruby=2.0.0
apt-get install nginx 
{% endhighlight %} </li>

<li> Check that nginx is already running by keying in your ip address/domain name into your browser. If the nginx default page appears, it means it is up. Otherwise, it can be started by running this command in a shell:
{% highlight shell %}
nginx
{% endhighlight %} 
or
{% highlight shell %}
service nginx start 
{% endhighlight %} </li>

<li> Download jekyll: 
{% highlight shell %}
gem install jekyll 
{% endhighlight %} </li>

<li> Checking the nginx website, the default location where a nginx webpage is stored at is /usr/share/nginx/html , so let's navigate to that folder: 
{% highlight shell %}
cd /usr/share/nginx/html
{% endhighlight %} </li>

<li> If the folder is not empty, feel free to clear it. Setup a default jekyll blog at that directory, and then run 'jekyll serve' to generate the static files:
{% highlight shell %}
rm -r (whatever files there)
jekyll new .
jekyll serve
{% endhighlight %} </li>

<li> Next, change the nginx configuration file to point to the _site/ sub-directory, because that's where the static files will be generated.
{% highlight shell %}
cd /etc/nginx/sites-available/
vim default
{% endhighlight %} </li>

<li>Look for the line that says:
{% highlight vim %}
root /usr/share/nginx/html 
{% endhighlight %}
and change it to: 
{% highlight vim %}
root /usr/share/nginx/html/_site
{% endhighlight %} </li>

<li>The default jekyll blog should be up now if you refresh your tab. To continuously update the static files whenever any posts are added, just run Jekyll in the background: 
{% highlight shell %}
jekyll serve --detach
{% endhighlight %} </li>

</ol> 

