---
title: "How to Safely Reset Git to a Previous Commit"
toc: true
toc_label: "Content"
tags:
  - git
  - branch
  - commit
  - software development
date: May 23, 2023
header:
  teaser: /assets/images/thumbnails/generic-thumb-400.jpg
excerpt: "Sometimes, when working on a project with Git, you may want to revert back to a previous commit."
---

# How to Safely Reset Git to a Previous Commit

Sometimes, when working on a project with Git, you may want to revert back to a previous commit. This can be done using the `git reset` command. However, it is important to do this safely to avoid losing any work. In this guide, we will go over how to reset Git to a previous commit safely.

## Step 1: Find the Previous Commit

The first step is to find the previous commit you want to reset to. This can be done using the `git log` command. 

```
$ git log --oneline
```

This command will show you a list of all the commits in the repository, with the most recent one at the top. Find the commit hash of the previous commit you want to reset to. 

```
commit 1234567 - Updated homepage
commit abcdefg - Added new feature
commit 9876543 - Initial commit
```


## Step 2: Reset to the Previous Commit

Once you have the commit hash, you can use the `git reset` command to reset to that commit. There are three types of `git reset` commands: `--soft`, `--mixed`, and `--hard`. 

- `--soft` will reset the commit, but leave the changes in the working directory. 
- `--mixed` will reset the commit and unstage any changes, but leave them in the working directory. 
- `--hard` will reset the commit and discard any changes in the working directory. 

For example, to reset to the previous commit using the `--mixed` option, run the following command:

```
$ git reset --mixed abcdefg
```


## Step 3: Commit the Changes

After resetting to the previous commit, you will need to commit the changes. This can be done using the `git commit` command. 

 ```
 git commit -m "Revert to previous commit"
 ```


## Step 4: Push the Changes

Finally, push the changes to the remote repository using the `git push` command. 

 ```
 git push origin master 
 ```

And that's it! You have safely reset Git to a previous commit. 

## Conclusion

Resetting Git to a previous commit can be a useful tool when working on a project, but it is important to do it safely to avoid losing any work. By following the steps in this guide, you can reset Git to a previous commit without any issues.
