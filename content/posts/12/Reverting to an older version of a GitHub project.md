---
title: Branching out with a previous commit in a GitHub project
date: 2025-01-12
description: a very simple thing to do that i cannot for the life of me ever seem to do right
toc: true
math: true
draft: false
categories:
  - git
tags:
---
*just a lil guide for my future self when i inevitably forget this again (and it's probably still wrong oops-)*

## - Steps:

### - Find the commit you want to revert to & copy its hash:

   ![](/posts/12/Screenshot%202025-01-12%20at%2011.23.40%20am.png)

### - Return to your open project:
   for me, i was working with a locally-cloned copy in VScode, connected to the remote repo's `main` branch, and was up to date with all of the changes made.

### - Create new remote branch: 
   open the terminal and run `git checkout -b <new-remote-branch> <old-commit-hash>`. This will create a new remote branch **populated with the project at the time of the commit hash you specified**, and switch you to it.
   
   *E.g., `git checkout -b names-update 4853ecf5765b7174465e604e8fd8bdd5430ea84f`.*

### - Push this new remote branch:
   then, simply push this new remote branch with `git push origin <new-remote-branch>`, and check that it appears on github!

   ![](/posts/12/Screenshot%202025-01-12%20at%2011.28.11%20am.png)
   
   Now you can operate off this new branch, containing the project in a previous commit's state.

---

***yes... i know this is a very simple thing to do that i only just kinda grasped >.<***