---
title: "Setting up Git LFS Linode server with the help of open-source"
summary: "To avoid splitting commits between GitHub and GitLab, I created my own LFS server on GitHub to surpass the 1GB limit. With the help of Git LFS S3 Proxy, I transferred the projects to the new server."
author: Laurence Sadler
date: 2024-01-24T00:00:00+00:00
ShowReadingTime: true
---



For those looking to set up your Linode Server with git-lfs, I used [git-lfs-s3-proxy](https://github.com/milkey-mouse/git-lfs-s3-proxy); this is my experience with setting it up.

My motivation to do this stems from trying to keep all my source code on one platform. Before, I was splitting my commits between GitHub and GitLab. For game development, Gitlab is a better choice as they give you 10GB of LFS storage across all your projects, while GitHub gives you….1GB - one unreal project that had only code, and the starter assets took up more than half of my storage alongside with my other unity projects. But GitLab doesn’t have much of an open-source community like GitHub has. Plus, I want my git contribution graph to be popin.

So, I decided to go with GitHub and set up my own LFS server to get past the 1GB limit. Why not just buy more storage? Because each project would still add to my total LFS storage, I would have to pay more later once I reach that upper limit, even if I archive it. The only way (on GitHub) to reclaim that space is to delete the repo, and you’ll lose those commits from your contribution graph. Which no…. I want those green squares.

![git graph image](https://blog.karenying.com/static/3791821603a2ba731cabf17cc891864b/9a301/github-commit-bot.webp)

Okay, that’s the motivation. Now, I had to pick a host. I decided to go with Linode, seeing that it was the most cost-effective one compared to AWS (in some cases), and I heard a lot about it from [The Linux Experiment](https://www.youtube.com/@TheLinuxEXP). Using ChatGPT & Bard, I thought it was going to be easy as - In the repo, initialize git lfs and use   `git config --global lfs.url https://your-object-storage-bucket.your-region.linodeobjects.com`.  Done, so I push a test 100mb file…. Nothing. I retrace my steps and use git lfs env, and the address is right, but it still doesn’t work. Now, it's time to get some help.

First, I went to Reddit to see how other people managed to get it working. Many of them were using AWS, most likely using this setup by [Alan Edwardes](https://alanedwardes.com/blog/posts/serverless-git-lfs-for-game-dev/). Well all I knew I had a S3 bucket, and I need to send data to it. What I noticed is that they all used some proxy, and then I found [Git LFS S3 Proxy](https://github.com/milkey-mouse/git-lfs-s3-proxy) by Milkey Mouse. After going through his setup steps and reinitializing the repo. It started, but I was getting an Authorization error. Using `GIT_TRACE=1`, I showed that it was not reaching the HTTP address, and it kept trying until it timed out. Feeling lost cause I’ve been trying to make this work for two days, I filed an issue and hoped the dev would respond. Luckily, within 2 days, 2 DAYS, he came in with a fix, and it worked! Now I have my own LFS server up & running, and I managed to migrate my projects to it.
