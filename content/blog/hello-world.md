---
title: "Hello World! 🎉"
date: 2018-07-29T14:23:10+02:00
---
Hello people! This is Deluvi speaking. Welcome to my first blog post!

I have been inspired by the [Indieweb movement](https://indieweb.org/) to make my own website. I really like their ideas of owning our own data and content and not letting companies owning them for convenience's sake, since it can be as simple to share on our own website with the help of modern tooling.

In this first blog post, I will share my current setup of this static website. Mostly, to make a static blogging website, you will need three things:

- A **static site generator**: this is a software that you use to generate your html that you will feed to your provider. To generate my whole website, I use [Hugo](https://gohugo.io). I made my own minimal theme just for funsies, but there are a lot of already made themes that looks good. Once you have a theme, you can start producing content right away. The posts are written in Markdown (a very simple language to write content). You can divide your blog content in different sections (here for example, there is a [blog](/blog) section and a [games](/games) section).
- A **hosting solution**: you need a server somewhere in the world to serve your html content. For this end, I am using [Github Pages](https://pages.github.com/). The main selling point of this solution is that it allows custom domain names for free unlike some other solutions.
- A **domain name**: this part is not mandatory to be able to publish, but if you are farsighted, it is probably more than recommended to have one. Say you don't take one and you are currently hosted on Github pages, if someone wants to link one of your blog post, they will use something like "username.github.io/blog/my-awesome-post.html". The day you change your hosting solution, those links will be broken. If you had one domain name, everything will be redirected automatically. The domain name is also your main identity on the web: it is something that you own and that is only for you.  
As for my setup, I use [Gandi](https://www.gandi.net/) as a domain name service for the [deluvi.com](https://deluvi.com) name mainly because it has been recommanded to me by someone else; just use one that you trust.

And that's it! With those things, you have a website and are ready to publish on the internet, independently of any entity!

In the next weeks, I will be trying to implement some of the Indieweb standards into this static website. Currently, I am aiming to implement [Webmentions](https://indieweb.org/Webmention) so you can leave me a comment below this article later (maybe you just did, person from the future). I will share my technique on how to implement them in Hugo and the principles to implement them in any static website generator.