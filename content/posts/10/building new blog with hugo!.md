---
title: building new blog with hugo!
date: 2025-01-04
description: ""
toc: true
math: true
draft: true
categories: 
tags:
---
*and now - to the story of how this blog was born!*

> *(it's nothing special, but I thought I'd document it for myself when i inevitably forget how i did it in the future, as well as any other wandering lost souls out there!)*

i've been meaning to re-jig my tech blog for a while now. for the last year and a bit, I experimented with the static site generator (SSG) [jekyll](https://jekyllrb.com/). jekyll is essentially a tool built in [ruby](https://jekyllrb.com/docs/ruby-101/) that combines **blog posts** (typically written in markdown, `.md` files) with **themes/config files** to generate browser-renderable code (`HTML`, `CSS` and `JS`).

this way, you can streamline your workflow, embed all sorts of cool features (comments, reactions, reading times, table of contents, automatic [rss feeds](https://en.wikipedia.org/wiki/RSS), post dating etc.), and most importantly **avoid the horror of writing blog posts in raw HTML**... but still being able to dabble in it when you please (providing your markdown-to-html renderer permits that).

![markdown vs html](/posts/10/Pasted%20image%2020250104220631.png)
...and all of this within a [static site](https://www.geeksforgeeks.org/static-vs-dynamic-website/) (all files pre-built on web-server, no databases) that is lightweight, responsive, maintainable and (relatively) quick to spin up.

### why did i move away from jekyll?
for three simple reasons:
1. i'd been meaning to try the SSG [hugo](https://gohugo.io/).
2. hugo is built in [golang](https://go.dev/), and i'd been wanting to poke around with go for a while now.
3. i found (and confirmed, after trying hugo) ruby & jekyll to be a bit more onerous to work with & overly-verbose in both site layout & base code. also - i noticed that jekyll had [much slower build times](https://css-tricks.com/comparing-static-site-generator-build-times/).

### getting up and running with hugo:

there are endless tutorials for this, and my pipeline is probably most similar to that of NetworkChuck in the [recent video](https://www.youtube.com/watch?v=dnE7c0ELEH8&t=907s) he released (not even a week before I went in on my own build, after sitting on the idea for ages haha - twas kinda spoopy :3). 

#### setting up the hugo site: 
<< to do >>

#### my final note-taking process:
1. open my Obsidian "blog" vault, and create a new note within a folder in `posts`:
   ![](/posts/10/Screenshot%202025-01-04%20at%2010.27.21%20pm.png)
   the [Templater](https://silentvoid13.github.io/Templater/introduction.html) plugin auto-generates the hugo-formatted frontmatter you see above in every new note, using the code block below inside the `template` file.
``` yaml
---
title: ""
date: <% tp.file.creation_date("YYYY-MM-DD") %>
description: ""
toc: true
math: true
draft: true
categories: 
tags:
---
```
2. write the post :3 - drag & drop / copy-paste images as needed, after making sure the `Absolute path in vault` option is selected in your vault's **Files and links** settings. This may need to be tweaked depending on your site's layout later, but it worked for me, and is easily changed in bulk in VSCode or a similar editor via **find & replace**.
   ![](/posts/10/Screenshot%202025-01-04%20at%2010.28.55%20pm.png)
3. once I'm finished writing, I switch to my full website directory tree in VSCode (my Obsidian "blog" vault is just the website's `content` folder, hence the `.obsidian` folder inside it). 
   ![](/posts/10/Screenshot%202025-01-04%20at%2010.33.17%20pm.png)
   i run the build command `hugo server -t [theme-name-here]` in the VScode terminal to start a live server, and visit `http://localhost:1313/` to double check that the changes have been formatted properly.
   ![](/posts/10/Screenshot%202025-01-04%20at%2010.41.47%20pm.png)
4. then a simple `git commit -m "new blog post: hugo site build" -a && git push origin main` pushes the changes to my site (whose architecture... I will save for another time, lol), where it's rebuilt & served as new HTML pages! 


![](/posts/10/ib2.jpg)