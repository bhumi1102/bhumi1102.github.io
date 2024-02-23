---
layout: post
title:  "How To Squash Commits in Git"
---

This one is about Git and squashing commits. Squashing commits can *feel* tricky since we're doing things like rewriting history, force pushing, and using the `rebase` command.

Note: This post is beginner friendly. If you're new to programming or to using Git, it's for you. If you squash commits in your sleep, you can skip this one. We'll be back with more posts in the Hotwire series later. A little break to document something that has come up a couple times recently (that my future self can reference), including as a question from a student. 

If you're using Github for version control, you may have used the "squash and merge" button in the UI of Pull Requests (PR) to automatically squash commits in a branch before merging the PR.

At my last full-time job, I used *Gitlab* for 7 years. We used the "squash" button to squash our commit before merging. Since we always used that option, I rarely manually squashed commits.

But in some cases, the repository is not configured to use the 'squash and merge' button. Either way, it's good to know how to manually squash and to understand what happens to the commit history behind the scenes. 

Let's see step-by-step how to squash commits. Before we do that, you may be wondering what *is* squashing and why do we need to do that.

## Why Do We Squash Commits

In Git, "squashing commits" refers to combining multiple commits into a single commit. This is typically done to commits on a feature branch before merging them into the main branch. It involves rewriting the commit history.

How often do you commit when you're coding? Some people do it every 10 minutes, some do it once a day. I'm somewhere closer to the 10 minute (it's like a compulsion, similar to doing cmd+s to save when writing prose).

Committing often is good but when we do that, we can clutter the git history. If we merge them to `main` branch without squashing, they not only clutter the git history, they also do not add value to a future reader of the commit log, and they can be a little embarrassing. So we squash those 50 commit messages that say 'typo' and 'it works now' into one pristine one, before merging, to help maintain a clean and readable commit history.

## How to Squash

For this demonstration, we'll first create a new branch `squash_test` off of `main`. Next, we'll make four commits on that branch and then squash those commits into one. So we only see a single commit when we merge `squash_test` into `main`.

You're likely starting a Step 3, and already have a feature branch and bunch of commits in that branch.

### Step 1

Create the `squash_test` branch. `git log` shows the starting commit on that branch:

```bash
$ git checkout -b squash_test
Switched to a new branch 'squash_test'

$ git log
commit b36ff9a23a769cafe686f7686f4e0010dfaf5ff8 (HEAD -> squash_test, origin/main, main)
Author: Bhumi Shah <notmyrealemail@gmail.com>
Date:   Wed Jan 24 12:26:00 2024 -0600

    break out of turbo frame for show action
```

### Step 2

Then make 4 commits to this branch. This is what the commit log looks like before we try the squashing:

```bash
commit 7feb63850ecc15044a8aec00576c66462a76a365 (HEAD -> squash_test, origin/squash_test)
Author: Bhumi Shah <notmyrealemail@gmail.com>
Date:   Mon Feb 12 14:15:43 2024 -0600

    commit four

commit e08d3e5b43b174613948616002260b7f2d0dbe05
Author: Bhumi Shah <notmyrealemail@gmail.com>
Date:   Mon Feb 12 14:14:52 2024 -0600

    commit three

commit 8258bc8dfb17a2a72aa345242373fea026e06a4c
Author: Bhumi Shah <notmyrealemail@gmail.com>
Date:   Mon Feb 12 14:14:14 2024 -0600

    commit two

commit e2144e56b96143abb2a677825119cc012ebd69bd
Author: Bhumi Shah <notmyrealemail@gmail.com>
Date:   Mon Feb 12 14:12:58 2024 -0600

    commit one

commit b36ff9a23a769cafe686f7686f4e0010dfaf5ff8 (origin/main, main)
Author: Bhumi Shah <notmyrealemail@gmail.com>
Date:   Wed Jan 24 12:26:00 2024 -0600

    break out of turbo frame for show action
```

### Step 3

In order to squash commits, we use the `rebase` command. Since we want to squash 4 commits in this case, the specific command is: `git rebase -i HEAD~4`.

An editor will open with a list of commits. Each commit will start with the word "pick". We can keep the first "pick" as is OR change it to "reword" if we want to change the commit message of the resulting squashed commit. We typically do want to change the commit message to something more meaningful. 

To squash the remaining commits, we change "pick" to "squash" (or just "s" for short) for the commits we want to squash into the previous commit.

This is what it looks like after I made the changes:

```bash
reword e2144e5 commit one
squash 8258bc8 commit two
squash e08d3e5 commit three
squash 7feb638 commit four

# Rebase b36ff9a..7feb638 onto 7feb638 (4 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified). Use -c <commit> to reword the commit message.
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

Once we save the above changes, another editor window will open to allow us to edit the commit message of the final squashed commit, if we used "reword". I reworded the message to "this is the squashed commit on branch squash_test". That completes the interactive `rebase`.

```bash
$ git rebase -i HEAD~4
[detached HEAD 7a61d28] this is the squashed commit on branch squash_test
 Date: Mon Feb 12 14:12:58 2024 -0600
 2 files changed, 5 insertions(+)
Successfully rebased and updated refs/heads/squash_test.
```

The git history after the `rebase` command is complete and the squashing is successful look like this:

```bash
commit ca22c3593bb1ed06fa66fb4804dad38c5ad595a7 (HEAD -> squash_test)
Author: Bhumi Shah <notmyrealemail@gmail.com>
Date:   Mon Feb 12 14:12:58 2024 -0600

    this is the squashed commit on branch squash_test

commit b36ff9a23a769cafe686f7686f4e0010dfaf5ff8 (origin/main, main)
Author: Bhumi Shah <notmyrealemail@gmail.com>
Date:   Wed Jan 24 12:26:00 2024 -0600

    break out of turbo frame for show action
```

Aside: I kept the comments in the first editor window above, as they're actually helpful and good to read through if you haven't before. Notice the line `# If you remove a line here THAT COMMIT WILL BE LOST.` This is true and this is why squashing can feel a little intimidating. Most things with Git are reversible though. If you make a mistake, you can `git rebase --abort`. There is also `git reflog`. The "reflog" records all the actions that modify a repository's history, including rebases. We can view the history and find the commit you want in the "reflog". That's is a bit of an advance Git topic, we won't worry too much about it here.

So now that we have a single clean commit in our `squash_test` branch, we need to push that to the remote branch so that it shows up in the PR. How do we that?

### Step 4

First, if this is a solo project and you have control our merging to `main`, you can simply checkout the `main` branch and merge in the `squash_test` branch:

```bash
$ git checkout main

$ git merge squash_test
```

Typically though, you need to push the squashed commit from your local branch to a matching remote branch and you have a PR that will eventually be merged into `main`. If we do `git push` we get an error:

```bash
 ! [rejected]        squash_test -> squash_test (non-fast-forward)
error: failed to push some refs to 'git@github.com:bhumi1102/hotwire_demos.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

This is because the commits we're squashing had already been pushed to the remote `squash_test` branch before. This is the case in a typical workflow. We'll need to force-push as we are rewriting history here:

```bash
git push --force
```

Note: Force pushing and rewriting history has to be used with caution, especially if other team members are working on the same feature branch.

With that, your remote feature branch has one single squashed commit. When your PR is approved and merged, `main` will have a clean and readable history. All your 47 commit messages that say things like 'typo' are gone.

{% include subscribe.html %}
