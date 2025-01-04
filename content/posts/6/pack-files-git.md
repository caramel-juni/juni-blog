---
title: "PACK files in .git - a rabbit hole"
date: "2024-12-23"
description: "lite 'n easy: for PACK files"
toc: true
math: false
draft: false
categories:
  - git
tags: 
  - PACK file
  - git-filter-repo


---

## - so, how did we get here? :see_no_evil:


---



git stores all historical changes to a repo in a PACK file inside the hidden .git folder. This allows restoration of previous repo states in the future. 

However, if you upload files like binaries, photos or videos, this file gets VERY large, even if you delete them in a future commit. 

### - enter: `git-filter-repo`

Luckily a tool exists called git-filter-repo that you can download and use (python script) to analyse your PACK file, and filter out any unwanted bits (e.g. file extensions, paths, etc.). This can dramatically reduce the size of the PACK file.

It works in a single command (with the option to point the command to a file defining what to keep/exclude, if preferred). Just download the python script, move it to your working directory (MUST have .git folder, as it will analyze this), and run:

`python git-filter-repo.py --analyze`

(the use of "python" or similar, and giving the script a .py extension, is necessary sometimes on windows, depending on what shell you're running the above command in, your PATH configuration etc. - but just think of it as running a script file and passing the "analyze" argument to it)

It then produces a folder with text files showing files/repo paths (historical) and their relative sizes. From here, you can search through and figure out how to filter what you'd like to remove.

When you've decided what you're going to remove and how (path/extension/date etc.), I recommend doing a --dry-run, which will produce two files (the original version and the modified version) and comparing what elements were removed with your filter. For me, using the following command, I went from 6473 lines of committed files to 1428.

`python git-filter-repo.py --path 'old-site/audio/' --path 'old-site/photos/' --invert-paths --dry-run --force`

*(I used --force as I had one untracked change - being moving the git-filter-repo script file itself into the directory - that I didn't want to push to git)*

And to make *"the changes"* ***[PERMANENTLY!! CAUTION!!!]*** remove the `--dry-run` component of the above command, resulting the following:

`python git-filter-repo.py --path 'old-site/audio/' --path 'old-site/photos/' --invert-paths`

Run that, and then there you go - it should make the changes to the .git folder in your repo, stripping out the components of the file you filtered and producing a new, (hopefully) smaller `PACK` file.

Now, it just needs to be pushed to the remote repository with:

`git push --all [remote-repo-URL]`

### - and then... 

well, if all things went well, you should have shaved a few KBs/MBs/GBs off your `PACK` file - well done! grab yourself a cookie, you've well and truly earnt it :3

![saya-congrats](/posts/6/saya-banner.jpeg)


---

### - other helpful links:
- https://www.youtube.com/watch?v=eoF2p3ZDiAc
- https://htmlpreview.github.io/?https://github.com/newren/git-filter-repo/blob/docs/html/git-filter-repo.html#EXAMPLES