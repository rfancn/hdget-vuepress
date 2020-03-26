---
date: '2017-04-13 10:21:06'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- tips
title: Git useful commands
---

##### Q. How to rename current master branch as old version and start to working on a new version?

1. rename current master to backup name or old version name
    
        git branch -m master <backup name> 

2. push to remote

        git push origin <backup name>

3. checkout to remote master

        git checkout master
        
##### Q: How to check changelogs between speicified version and latest version?

1. fetch the latest commits from git repository

        git fetch origin master
     
2. list the tags

        git tag
     
3. check the changelogs difference
     
        git log --pretty=oneline [OLD VERSION]...[NEW VERSION]
        e,g: git log --pretty=oneline v3.8.13-44...v3.8.13-55.1.5
     
##### Q: How to recover the forced deleted files in git

1. find the lost-found files

        git fsck --lost-found
    
    or

        git fsck --cache --no-reflogs --lost-found --unreachable  HEAD
    
2. Then examine files in '.git/lost-found/other', e,g:

        cat .git/lost-found/other/390b179065eec600bb7be2310b9d331bc5c7633e

3. Rename and move the missed files to the correct location

        mv .git/lost-found/other/390b179065eec600bb7be2310b9d331bc5c7633e [/path/to/FILENAME]
     
##### Q: How to delete remote branch

    git push origin --delete [REMOTE BRANCH NAME]

##### Q: How to delete local branch

    git branch -d [LOCAL BRANCH NAME]

##### Q: How to push local to remote branch

    git push origin [REMOTE BRANCH]

##### How to switch to different branch

    git checkout [NEW BRANCH NAME]