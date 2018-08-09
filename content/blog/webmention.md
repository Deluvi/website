---
title: "Implementing Webmention on a static website"
date: 2018-08-09T14:00:00+02:00
summary: "An explanation of Webmentions and an implementation on Hugo"
---
On my first blog post, I mentioned that my first project for this website is to experiment with some technologies from [the Indieweb](https://indieweb.org/). The technology that attracted me the most is [Webmention](https://indieweb.org/Webmention).  
In this article, I will first elaborate on what are webmentions. Then, I will explain how to send them. Finally, I will show the different ways to receive them when using a static website.

## What is Webmention?

Webmention is a web standard that allows having interactions in a decentralized way. For example, if you link someone on one of your articles, you can inform that person of the mention. That person can then display your mention on his article page. Mentions can be of different types: comment, like, repost, mention, bookmark, etc. As long as there is a link to the source on the page, the webmention is valid.

The big advantage of Webmention is that the protocol is very simple: it only relies on an HTTP POST request.

## Sending webmentions

Sending a webmention to someone is easy. As detailed in [Sending your First Webmention from Scratch](https://aaronparecki.com/2018/06/30/11/your-first-webmention) (which I recommend you to read to have more information about Webmention and implementing microformats), you just need to create an HTML document accessible on the web with some basic content and notify the person. To notify a website, you first have to find a link tag in the head of the page with the attribute `rel="webmention"` (here, it is `<link rel="webmention" href="https://webmention.io/deluvi.com/webmention"/>`). Then, you do an HTTP POST request on that URL with two fields: `source` which contains the URL of your post and `target` which contains the URL of the page you are mentioning. Once notified, the server checks if the target is linked by the source anywhere on the page and adds a new entry internally if that's the case so he can retrieve it later. Some websites (like mine, see below) have a form to submit your webmention via a browser. In theory, the HTTP POST request should be automatic according to [the W3C recommendation](https://www.w3.org/TR/2017/REC-webmention-20170112/#protocol-summary).

On most webmention implementations, the server gets more information about the post like the title, the content of the post and the author info by parsing the [microformats](https://indieweb.org/microformats) contained in the HTML of the article. Microformats are simply some class attributes added on some HTML tags. The most important microformats to implement are [h-card](http://microformats.org/wiki/h-card) which represent your identity and [h-entry](http://microformats.org/wiki/h-entry) which contains the information for a particular post. You can test the microformats of your website [here](https://indiewebify.me/).

As a side note, you don't have to write an official blog post to answer someone. For example, I have a section /replies/ where I put all my replies. Just do not forget to add the microformat class attribute `u-in-reply-to` to a link of the post you are responding to somewhere on your page.

## Receiving webmentions in a static context

In this article, we are assuming that we don't have any dedicated server available to us, only a static webpage provider. First issue, we need something to receive those HTTP POST requests on our behalf. Thankfully for us, there are [some free services](https://indieweb.org/Webmention#Services) that can help us receiving and storing webmentions so we can query them later. One of those services, the one I am using, is [webmention.io](https://webmention.io/).

Once you have a backend, you have mainly two options to display the webmentions to the client when you are working with a static website. You can make your readers query the server and display the comments using JavaScript. This option is the most dynamic one as the commenter can see his reply as soon as the backend processed his notice. It is also very simple to set up since you just have to include a JavaScript file on the page and let it do its job. However, it means using JavaScript when it is not necessary and also adds more load to the webmention backend that has to be used every time you load the page.  
The other option is to include the webmention directly in the HTML when the website is generated. With that alternative, you don't need any javascript nor external query on the client side. However, this comes with a cost of interactivity and complexity, since you have to rebuild and reupload your website as you receive new mentions.  
As an experiment, I chose the second option.

To be able to query the webmention server and store the mentions so that they can be accessed by my static blog generator, I developed a small tool in Rust called [getwms](https://github.com/Deluvi/getwms). This tool retrieves the webmentions from webmention.io, convert them into a standard JSON format and store them into separate files for each post. In the future, I intend to improve this program by having it only retrieving the newest webmentions, not all of them, building the JSON files incrementally as new mentions appear. Additionally, I will make sure that it notifies if there are no new webmentions available.

Once the webmentions have been pulled, we just have to use the data to generate the website's webmention sections using our static blog generator functionalities. Hugo has some good functionalities to load and use JSON data. If you want to see the source code to set up your Hugo website to process the JSON, you can check the bottom of [single.html](https://github.com/Deluvi/website/blob/master/themes/deluvi/layouts/_default/single.html) and [webmention.html](https://github.com/Deluvi/website/blob/master/themes/deluvi/layouts/partials/webmention.html) to have an idea how I am achieving that. Here is the most important part of the code:
```
{{ $pathJSON := (print "data" (strings.TrimSuffix "/" .URL) ".json") }}
{{ if fileExists $pathJSON }}
<div class="webmentions">
    {{ $mJSON := getJSON $pathJSON }}
    {{ partial "webmention.html" $mJSON }}
</div>
{{ end }}
```
It generates the path where the JSON file should be, then gives the JSON to the partial template if the file exists. Once this is included in your article template and you defined a partial template to turn the JSON into displayable HTML, you should start to see your webmention on your website! It should be possible to do a similar thing in any static website generator that supports JSON files.

The process has been automated for my website. Right now, I execute a script every hour on my PC. My script:

1. Runs getwms to get the JSON.
2. Builds the website with Hugo.
3. Tries to commit all changed files on my website repository: it will fail if there were no new changes.
4. Tries to push: it will fail if there were no new changes.

Here is a copy of my batch script:
```
getwms -u "https://webmention.io/api/mentions?domain=deluvi.com&token=NOTHINGTOSEEHERE"
hugo
cd public
git commit -a -m "Update"
git push
```

And that's it! This is my current setup to receive webmentions. Feel free to inspire yourself from this method if you think this is suitable for your needs.

If you are using a more conventional website engine like WordPress, you will probably have a webmention plugin available to you: no need to put so much effort into it. If you want to receive webmentions of likes and replies from social networks, you can syndicate them back to your website using [Bridgy](https://brid.gy/). Syndicating the reaction of your content is important as it makes you more independent from centralized social websites.  
If you want to send me a webmention, go ahead! The form is just below. Just be patient if you want to see the result, the webmentions will be refreshed as soon as possible.

As a bonus, here is an interesting conference about Indieweb and some technologies you can use (Webmention, rel="me", IndieAuth, Micropub...): [Taking Back The Web](https://vimeo.com/265121482)

<a href="https://news.indieweb.org/en" class="u-syndication">Posted on IndieNews</a>