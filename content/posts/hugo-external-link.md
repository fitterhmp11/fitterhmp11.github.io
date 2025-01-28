---
title: "Working With Hugo: External Links and Shortcodes"
date: 2025-01-28T15:00:00
tags:
  - hugo
categories: 
 - hugo

draft: false
url: "hugo-external-links"
summary: "Working With Hugo: External Links and Shortcodes"
description: "Working With Hugo: External Links and Shortcodes"
---

I was hoping to add some external links to references I used for my content, in particular for my podcast notes pages to the the webpage of the podcast creators.

At first I just looked into adding a link in markdown, that is Hugo compatible that would create the link in html.  ChatGPT showed me the following:

`[Link Text](URL)`

I used that, but it does not automatically open in new tab, which would be nice.  ChatGPT suggested "Shortcodes", because Hugo does not allow for that on its own.

Shortcodes are a Hugo feature that allows for custom or reusable functionality (think variable or function you call).  


ChatGPT said *Create a file in layouts/shortcodes/external-link.html with the following content:*


`<a href="{{ .Get 0 }}" target="_blank" rel="noopener noreferrer">{{ .Inner }}`


I noticed I don't have a "Layouts" folder, but when asked ChatGPT is said: *If your Hugo project doesn't have a layouts folder, that's normal if you're using a pre-built theme. Themes often provide their own layouts, but you can create a layouts folder in your project root to customize or extend your site with your own shortcodes, templates, or partials.*

*In your project root (where config.toml or config.yaml is located), create a new folder called layouts.
Create a shortcodes Subfolder.*

So I created the folders and used the suggested file name `external-links.html`.  


*Why Create the layouts Folder?  
The layouts folder in your project allows you to override or extend the functionality provided by a theme. If a theme doesn't include a specific layout, shortcode, or partial, Hugo will fall back to checking your custom layouts folder.*

here is test of using the external link

Check out {{< external-link "https://gohugo.io" >}}Hugo's official website{{< /external-link >}}.



