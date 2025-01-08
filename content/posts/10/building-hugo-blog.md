---
title: building & deploying this blog with hugo!
date: 2025-01-08
description: "my process of building this blog with hugo, ft. my cursed website build & deployment pipeline"
toc: true
math: true
draft: false
categories: 
   - blog
   - hugo
   - website
   - obsidian
tags:
   - cloudflare
   - github
   - git
   - go
   - github pages
   - markdown
---

# *and now - to the story of how this blog was born!*

> *(it's nothing special, but I thought I'd document it for myself when i inevitably forget how i did it in the future, as well as any other wandering lost souls out there!)*

i've been meaning to re-jig my tech blog for a while now. for the last year and a bit, I experimented with the static site generator (SSG) [jekyll](https://jekyllrb.com/). jekyll is essentially a tool built in [ruby](https://jekyllrb.com/docs/ruby-101/) that combines **blog posts** (typically written in markdown, `.md` files) with **themes/config files** to generate browser-renderable code (`HTML`, `CSS` and `JS`).

this way, you can streamline your workflow, embed all sorts of cool features (comments, reactions, reading times, table of contents, automatic [rss feeds](https://en.wikipedia.org/wiki/RSS), post dating etc.), and most importantly **avoid the horror of writing blog posts in raw HTML**... but still being able to dabble in it when you please (providing your markdown-to-html renderer permits that).

![markdown vs html](/posts/10/Pasted%20image%2020250104220631.png)

...and all of this within a [static site](https://www.geeksforgeeks.org/static-vs-dynamic-website/) (all files pre-built on web-server, no databases) that is lightweight, responsive, maintainable and (relatively) quick to spin up.

---

## - why did i move away from jekyll?
for three simple reasons:
   1. i'd been meaning to try the SSG [hugo](https://gohugo.io/).
   2. hugo is built in [golang](https://go.dev/), and i'd been wanting to poke around with go for a while now.
   3. i found (and confirmed, after trying hugo) ruby & jekyll to be a bit more onerous to work with & overly-verbose in both site layout & base code. also - i noticed that jekyll had [much slower build times](https://css-tricks.com/comparing-static-site-generator-build-times/).

---

## - getting up and running with hugo:
   there are endless tutorials for this, and my pipeline is probably most similar to that of NetworkChuck in the [recent video](https://www.youtube.com/watch?v=dnE7c0ELEH8&t=907s) he released (not even a week before I went in on my own build, after sitting on the idea for ages haha - twas kinda spoopy :3). 

### - setting up the hugo site: 
   1. Hugo has two simple dependencies: `git` (for code version control), and the `go` compiler toolchain. (here are where you can install [`git`](https://git-scm.com/) and [`go`](https://go.dev/), if needed)
   
   2. after installing these, using the package manager of your choice (for me, `homebrew`), [install hugo](https://gohugo.io/installation/) with `brew install hugo`.
   
   3. choose a directory for your site, and open it in your code editor. ensure hugo is in your system PATH so you can access it via the command line, and run `hugo -v` to ensure you're on the latest.
   
   4. then simply run `hugo new site [SITENAME]`, replacing `[SITENAME]` with whatever you'd like to call the site (& folder it lives in). hugo will then spinup the basic bones of your site, and navigate into the folder it creates with `cd`.
   
   5. initialise an empty git repository in this new folder with `git init`.
   
   6. to install a theme, browse them [here](https://themes.gohugo.io/), and follow the instructions in the theme's description (as some methods vary). 
      however, the most common is installation is via a **git submodule** - which essentially will just pull down an existing git repo containing a pre-built hugo theme, and populate your site's `themes` folder with it. this way, when building your site, hugo will use it as a base layout, and add any changes made to your site on top of it.
      for me, i ran: `git submodule add https://github.com/michaelneuper/hugo-texify3.git themes/hugo-texify3`

   7. now, your site's directory tree should look something like the following:

      ![](/posts/10/Screenshot%202025-01-08%20at%206.35.22%20pm.png)

      Folders & files are fairly self-explanatory, with the main ones being:
      - **`hugo.toml`** - your site's configuration variables.
      - **`/content`** - where you create folders to store blog posts (`.md`) & site pages
      - **`/assets`** - ideally where media is stored & linked (although you can place them anywhere, theoretically, providing you link back to it correctly)
      - **`/public` (only created when website is built, see step 8.)** - where your **raw website** (raw HTML, CSS, JS) will be built to and live. **you shouldn't need to touch this folder.**
      - **`/themes`** - where all of your sites themes are installed (and specified/switched between in `hugo.toml`)

   8. sometimes, themes require installing **other tools** as part of their custom build process. this *should* be specified in the theme's documentation. 
      for me, that required needing to install the following with `npm`: 
      `npm install postcss-cli autoprefixer postcss-import` 

   9. to build your site locally, ensure you're in your site's base directory and run `hugo server -t [THEME-NAME]` (if using a theme). 
      for me: `hugo server -t hugo-texify3`

      ![](/posts/10/Screenshot%202025-01-08%20at%206.33.44%20pm.png)

   10. now, navigate to the local address to see your site in action! it should live-reload as you make changes in your code editor.

   11. **[EXTRA]** after analysing my specific theme's layout & directory structure, I mirrored elements of it (thus overwriting what was contained in the `/themes` folder) to create the below folder structure, allowing me to:
      - add dedicated website pages in `/pages`
      - use `/posts` to hold my site post, with each in its own **folder** alongside any assets (images, media, etc.). this was done due to my particular workflow (writing in obsidian, see below). 
      - split site configuration into two files for readability in `/config`: one for parameters (enabling/disabling certain features like social links, metadata etc.), and one for overall config & layout.
        ![](/posts/10/heya.png)

### - pushing to remote git repository & deploying via cloudflare pages 
*(aka my weird custom workflow):*
   1. after running `git init`, ensure you are authenticated locally with `gh auth login` (requires use of Github CLI, install with `brew install gh` or similar)

   2. create a new repository on GitHub with:
      `gh repo create juni-blog --public --source=. --remote=origin`
      - `--public`: sets repo as public, as cloudflare will need to monitor it for changes
      - `--source=.`: initialises the remote repository with your current local directory's contents
      - `--remote=origin`: sets up the remote URL for the repository

   3. now, just push the local branch to the remote with `git push -u origin main`.
      > The `-u` (or `--set-upstream`) option in `git push` links your **local branch** (`main`) to the **remote branch** (`origin`) by default, allowing you to run `git push` and `git pull` commands in the future without adding `origin main` at the end (AKA specifying which remote branch to interact with by default).

   4. navigate to your remote repository on github to check whether the changes have been propagated!

   5. setup, login to and open [cloudflare pages](https://dash.cloudflare.com/) and navigate to your **`Workers & Pages`** section, then **`Create`** to deploy a new "site". The click **`Connect to Git`** and follow the prompts to authenticate, and link to the repository that you just pushed to. 

      ![](/posts/10/Screenshot%202025-01-08%20at%206.56.36%20pm.png)

      ![](/posts/10/Screenshot%202025-01-08%20at%206.58.49%20pm.png)

      After that, you can specify which branch Cloudflare should look for changes on, as any frameworks that you're using to build & deploy the site, and where the built assets & HTML files are stored. I selected `Hugo` (for obvious reasons), and it populated the build command with a simple `hugo`.
      
      ***However***, if your site requires other tools as part of the build process like mine (specified in step 8 in the previous section), be sure to **install them on the remote server this site is running off** with the relevant commands. for me, these are:
      `npm install postcss-cli autoprefixer postcss-import && hugo`
      Cloudflare should provide logs from the remote server should your build fail, making troubleshooting fairly simple.

      ![](/posts/10/Screenshot%202025-01-08%20at%207.01.36%20pm.png)

   6. then you're all set! cloudflare will now watch for any changes made to the specified branch of this repo (for me, `main`), and if detected, it will automatically run `npm install postcss-cli autoprefixer postcss-import && hugo` on its remote server(s) to build the updated version of my site, and then serve **only** the created HTML, CSS & JS files in the specified output directory (for me, `public`). 
      you should be able to access your site from the default URL created and provided to you, like https://b3ce9f44.juni-blog.pages.dev/.

   7. **[extra]** to change this URL to a custom domain that you own, go to the **`Custom Domain`** section of the page you just created, click **Set up a custom domain**.

      ![](/posts/10/Screenshot%202025-01-08%20at%207.16.58%20pm.png)

      Then, follow the prompts. In my case, I associated the `myblog` CNAME with this "Page" (`juni-blog.pages.dev`).

      ![](/posts/10/Screenshot%202025-01-08%20at%207.13.22%20pm.png)

      Then, simply navigate to your domain's (`juni-mp4.com`) DNS records (for me, also managed via Cloudflare) and add a record for the CNAME you just specified.

      ![](/posts/10/Screenshot%202025-01-08%20at%207.16.02%20pm.png)

      This means that when someone visits `myblog.juni-mp4.com`, they will functionally be visiting `juni-blog.pages.dev`.

this combination of github & cloudflare pages allows me to easily host & deploy sites from **different github repos**, each built with **all kinds of different build methods** (from hand-coding raw HTML/CSS/JS to using various SSGs like `jekyll`, `hugo` etc.), as subdomains of my primary domain `juni-mp4.com` and all served securely & quickly from Cloudflare's worldwide array of web servers. *(not sponsored haha - the only thing you monetarily pay for is your domain registration fee, and even that is optional.)*
   
***Side note:*** *yes, whilst you could argue that you "pay" in the form of you & your content being technically in the hands of cloudflare, outside of hosting file files yourself on a VPS or a home server - which comes with a slew of additional overhead, maintenance and security configuration concerns - this is a fairly reasonable compensation to make for the uptime, security & responsiveness that Cloudflare's network of servers provides, at least imho and for my use case.*

this very particular workflow & need for flexibility is why i chose to use cloudflare instead of just deploying straight from [Github Pages](https://pages.github.com/). 

---

## - my final note-taking process: an overview
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
         
   2. write the post :3. drag & drop / copy-paste images as needed, after making sure the `Absolute path in vault` option is selected in your vault's **Files and links** settings. This may need to be tweaked depending on your site's layout later, but it worked for me, and is easily changed in bulk in VSCode or a similar editor via **find & replace**.

      ![](/posts/10/Screenshot%202025-01-04%20at%2010.28.55%20pm.png)

   3. once I'm finished writing, I switch to my full website directory tree in VSCode (my Obsidian "blog" vault is just the website's `content` folder, hence the `.obsidian` folder inside it). 

      ![](/posts/10/Screenshot%202025-01-04%20at%2010.33.17%20pm.png)

      i run the build command `hugo server -t [theme-name-here]` in the VScode terminal to start a live server, and visit `http://localhost:1313/` to double check that the changes have been formatted properly.

      ![](/posts/10/Screenshot%202025-01-04%20at%2010.41.47%20pm.png)

   4. then a simple `git commit -m "new blog post: hugo site build" -a && git push origin main` pushes the changes to my site where it's rebuilt & served as new HTML pages! 

---
