---
title: The first steps in Github Pages
date: 2022-10-14    
excerpt: A small excerpt
---
# Working with Github pages

_"Are you serious, Rob, you don't know how to create Github pages?"_

Yes, I never looked into to be honest. I know you can create them with regular HTML, CSS and some JS, but I want to work on something new, so I'll start with Jekyll and markdown. 

I want something that is easy to maintain, so I am able to proceed with it with not too much effort and overhead. Jekyll and markdown seems to be a good fit. But as I've never used github pages before so, it's all new for me. As the purpose of the github pages is to serve as a blog, it's a very good starting point. Allright lets dive into it. 

## Specify a theme

The first thing you need to do is add a _config.yml, which looks like this in my case. Which just specifies the theme and the installed plugins. As a menu is required, the Jekyll menus are added over here. 

````
theme: minima
plugins:
- jekyll-menus
````

This gives already a clean look to your page, although it isn't fancy yet. But that is something that wil be fixed later, content over presentation, right? For those who know me, I'm not that frontend guy which works on that slick webpage. I'm the one sitting quietly in the back trying to figure out why we experience a too high latency :smirk: . 

Okay, next step is adding the blogs itself, which is actually pretty easy. 

## Adding blog posts

A blog post can be added very easily by adding a new markdown page in a folder called ``_posts``. The only convention you need to be aware of is the name of your file. This requires the format of ``<YYYY-MM-DD-TitleOfYourBlogPost>``. In this example, it is  ``2022-10-15-CreatingGitHubPages``, so that aint't too hard, is it? 

At the top of your blogpost, you can add more metadata, for example title and date, which can be used by some plugins. See below for the example. 

````
---
title: "The first steps in Github Pages"
date: 2022-10-14
---
````

## About page
Every blog needs an ``about`` page, so this one as well. This can be done, by adding a new markdown page where you specify all the contents. 
If you want to be page to be accessable, you need to tell Jekyll in which menu you want it to appear. That can be done, by adding some metadata at the top of your page again. See the example below.

````
---
title: About
menus: header
---

I am a .NET consultant with a focus on architecture and cloud. I ....
````
That's it for now, and I'll explore some more about it, and will tell you more in an upcoming post! To be continued...