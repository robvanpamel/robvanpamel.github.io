---
title: The next steps in Github Pages
date: 2022-10-15   
excerpt_separator: <!--more-->
tags: Github Jekyll
---
After the first steps follows ... the next steps. I created the first blogpost and made it public, I already realised that it wasn't ready yet to be launched. It was way too early, but at least it was something. 

So while waiting  <!--more--> in the airport on my next flight I continued to work on it. What needed to be done ? 
A lot of things! 
* The title of the site was the one from the repository
* I didn't had the social media links
* I couldn't get an excerpt of my blogpost on the main page
* ... 

## Adding social media links 
Adding these social media links didn't went as fast as I expected it. It seemed that I was using an 'old' version of the minima theme. Due to this, the settings which i applied in the ``_config.yml`` didn't apply. 

You can always refer to the remote theme from github by using a plugin. After changing it to use that remote theme, I got a step further and after tweaking a bit more the media links worked. See the `_config.yml` below for more information

````
minima:
  social_links:
    twitter: youraccount
    github: youraccount
    linkedin: youraccount-878796454

remote_theme: jekyll/minima
plugins:
  - jekyll-remote-theme

````

## Adding titles
By default the title and description of your page are the name and description of the repository. Which is mostly not what you want. So lets change this as well
I saw multiple options online, but not all of them worked. This one worked for me in the `_config.yml`
````
title: Rob Van Pamel
description: Rob Van Pamel
```` 

## Adding Excerpts 

Another default setting of the minima theme is that there is no excerpt shown on your post page, but only the title. I like it that you can already see a bit of the post, so I enabled that as well in the  
`_config.yml`

````
show_excerpts: true
````
Afterwards add the excerpt seperator in your blogpost header and off course also in the blogpost. Here is an example 

    title: The next steps in Github Pages
    date: 2022-10-15   
    excerpt_separator: <!--more-->
    ---
    After the first steps follows ... the next steps. I created the first blogpost and made it public, I already realised that it wasn't ready yet to be launched. It was way too early, but at least it was something. 

    So while waiting  <!--more--> 

That will be it for now, it the next one, i would like to change some more smaller things, like a favicon, look at SEO, add tags to the blogposts, and so on. 