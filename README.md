# Keeping commit history clean and readable.

#[todo] add TOC

In this repo we model a realistic (though quite simplified) work-flow listed in the Day 1,2,3 **Sequence of events** section below.

The common scenario shown, is where you have a branch used to combine contributions, we'll call ours: `development`. 
Then in order to implement a feature in isolation, you will create a separate branch, in this example, called `feature-x`

Given this context we will compare and contrast different strategies of managing the feature branch which includes applying
changes from the development branch during `feature-x`'s development and the final merge into `development` once the work on
`feature-x` is complete.

## Sequence of events

The real world activity for the `development`, and `feature-x` branches is shown below.  This same sequence of events
is used for all strategies demonstrated below.  The demonstrations only differ in how they merge work between the branches. 

**Day 1**

* Jane creates the `feature-x` branch from the `development` branch
* Bob commits file0.txt to `development`
* Jane commits file1.txt to `feature-x`

**Day 2**

* Jane starts the morning by applying new changes from `development` onto `feature-x`
* Jane commits changes to file1.txt on `feature-x`
* Bob commits file2.txt to `development`
* Bob commits file3.txt to `development`
* Jane end the day by applying new changes from `development` onto `feature-x`

**Day 3**

* Bob commits an edit to file2.txt on `development`
* Jane applies new changes from `development` onto `feature-x`
* Jane committed file4.txt to `feature-x`
* Bob committed and edit to file3.txt on `development`
* Jane is finished with `feature-x`
  * She does one final application of new changes from `development` onto `feature-x`
  * And then merges `feature-x` into `development`


## Strategies to enact the work-flow above

Below, a few strategies will be shown for managing this work-flow.  This will primarily center around the use of two git
tools: `merge` and `rebase`.

Depending on how these tools are utilized can have a dramatic effect on the resulting commit history of the `development` 
branch.  Via these demonstrations, the goal here is that you will be able to choose the technique that works best for your
circumstance.

### Strategy 1: Use Only Simple Merge

In this case, in the above sequence of events, any where it states: *applying new changes from development onto feature-x*,
the literal git command used is `merge`.

    [feature-x]$ git merge development
    
The resulting feature branch is [here](https://github.com/samkeen/git-demos/commits/feature-x-always-merged-development)

And the state of the `development` branch after the merge is shown [here](https://github.com/samkeen/git-demos/commits/development-received-feature-x-merged)

Our primary concern here is the commit history of the `development` branch since that is the audit trail of work done for this 
project. The `feature-x` branch will be deleted when it is completed, so at that point it is no longer a concern.

If you feel these commit histories seem difficult to interpret, or even misleading, you are not alone.

#### Issues with this strategy

* **Merge commit spam**
* **feature-x contributions to the development history are non-sequential**

First off, **Merge commit spam**, the `development` branch history has 12 entries for the work-flow, but if you examine that work-flow it only 
lists 8 commits?  Maybe 9 if you would expect a commit for the final merge of `feature-x` to `development`.

All these *Merge branch 'development' into feature-x-merge* commits are valid information.  They are telling us that the
developer of `feature-x` was routinely pulling in new commits from `development` into `feature-x` during the implementation of
`feature-x`.  Great, but in the context of the `development` branch commit history, we don't care.  This information has no
effect on the actual mutations/additions of content on the `development` branch, so it's addition is simply making the commit
history more difficult to read.

So on to the the **non-sequential nature of the contributions from feature-x** to `development`.

If we remove the merge spam, the sequence of commits on `development` look like this

```
#
# commit history of development branch (minus the merge commits)
#
3e97637 development : added middle and end to file 3
6b0f943 feature-x : added file 4
d21db1c development : added middle to file 2
e4e6aed feature-x : added the end to file 1
dca2baa development : added file 3
e042f4c development : added file 2
000bc1c feature-x : added file 1
47ae98d development : added file 0
85afe13 Initial commit
```

The 3 `feature-x` commits are interspersed between `development` branch commits.  
In actuality, this order is the true order of the commits on `feature-x` with respect to `development` as `feature-x` and `development` were worked on in parallel.  This is valid information, but with respect the the commit history of the `development` branch, we don't care.  
All we care about is the moment in time when the contributions from `feature-x` where combined with the history of `development`, and would prefer those contributions to be atomic; a single commit or an uninterrupted sequence of commits.
If this were the case, the history on `development` would be greatly simplified and allow tools such as `bisect` to function as expected.  

This is more apparent if we consider the fact that I've helped us out here bay adding the branch name to commit messages (something you would never see in the real world), if you remove those, all obvious signal as to the origins of contributions is gone:

```
#
# branch hints in commit messages removed (source of commits less than obvious)
# 
3e97637 added middle and end to file 3
6b0f943 added file 4
d21db1c added middle to file 2
e4e6aed added the end to file 1
dca2baa added file 3
e042f4c added file 2
000bc1c added file 1
47ae98d added file 0
85afe13 Initial commit
```
 
### Strategy 2: Use Rebase to keep in sync with development (rather than simply merge)


In this case, in the above sequence of events, any where it states: *applying new changes from development onto feature-x*,
the literal git command used is `rebase`.

    [feature-x]$ git rebase development
    
The resulting feature branch is [here](https://github.com/samkeen/git-demos/commits/feature-x-always-rebased-development)

And the state of the `development` branch after the merge is shown [here](https://github.com/samkeen/git-demos/commits/development-received-feature-x-rebased-with-ff)

In looking at the resultant `development` branch history there should be something immediately obvious: **no merge spam**.  
As noted above, this demonstration work-flow contains 8 actual commits and there are 8 entries in this commit history.  
So we are already better off, no clutter in the commit history, making it easier to read.

Looking at the commit history, there is another apparent difference with the simple merge only strategy:  our 
`feature-x` contributions to the `development` branch have been recorded as an atomic block of commits.

```
[development-received-feature-x-rebased-with-ff]$ git log --oneline

6b6fa9f feature-x : added file 4
5521ab7 feature-x : added the end to file 1
2249bf8 feature-x : added file 1
3e97637 development : added middle and end to file 3
d21db1c development : added middle to file 2
dca2baa development : added file 3
e042f4c development : added file 2
47ae98d development : added file 0
85afe13 Initial commit
```

With the commit history of `development` recorded thusly, those viewing it have the most important information presented
simply and clearly:  ***At what point where the the `feature-x` contributions merged to `development`, and which commits
were those contributions comprised of?***

With this history, it is completely obvious how to take to repo back to a point just prior to the addition of the `feature-x` 
contributions (i.e. git reset --hard 3e97637).  This sequential history also makes tools such as `bisect` much more useful
and overall trouble shooting of a repo much more straight forward.


#### Issues with this strategy

You can only rebase if this is your *private* branch, in the sense that no one else is contributing to `feature-x` with you (even 
yourself on multiple machines).  In the real world this is very often the case where you have single developers solely
responsible for small features. 

But even at that you will need to -f your pushes to origin/feature-x since you are overwriting the history there, and it just feels dirty.


### Strategy 3: Do what ever you want on the `feature-x` branch and let rebase sort it out in the end

So here we are under the premise the `feature-x` used simple merge to pull changes from `development` from time to time just
as we did in the first example.  Maybe we did this since we were collaborating with another developer on `feature-x`, regardless
now we are finished with `feature-x` and it is time to merge it into the `development` branch.

If we simply merge `feature-x` into `development` as it is, it will create the issues demonstrated in the first example here.
Luckily for us, git has what is called *interactive rebasing*, allowing us to change the history of `feature-x` prior to 
merging it into `development`.  Since this is the 'end of life' for `feature-x`, rebasing it does not have the history conflict
concerns we would have if we were still collaborating with others on `feature-x`. 

If we look at `feature-x`'s history (ignoring merge commits) we see this:

```
[feature-x]$ git log --oneline --no-merges
3e97637 development : added middle and end to file 3
6b0f943 feature-x : added file 4
d21db1c development : added middle to file 2
e4e6aed feature-x : added the end to file 1
dca2baa development : added file 3
e042f4c development : added file 2
000bc1c feature-x : added file 1
47ae98d development : added file 0
85afe13 Initial commit
```
    
We would like to reorder these commits so that the `feature-x` commits are all at the tip of this history.  We do this by 
telling rebase to take us back to commit 47ae98d.  This can be done in one of these two ways:

```
# rebase back 7 commits
[feature-x]$ git rebase -i head~7

# rebase back to (but not including) 85afe13
[feature-x]$ git rebase -i 85afe13
```
    
You will then see the rebase console: (note: the commits are in the reverse order you are used to seeing in tools 
such as git `log`)

```
pick 000bc1c feature-x : added file 1
pick 47ae98d development : added file 0
pick e4e6aed feature-x : added the end to file 1
pick e042f4c development : added file 2
pick dca2baa development : added file 3
pick d21db1c development : added middle to file 2
pick 6b0f943 feature-x : added file 4
pick 3e97637 development : added middle and end to file 3

# Rebase 85afe13..dad96f0 onto 85afe13
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

You can then use your editor to bring all the `feature-x` commits to the tip of this history (leaving the order of the
`developments` commits intact)

```
pick 47ae98d development : added file 0
pick e042f4c development : added file 2
pick dca2baa development : added file 3
pick d21db1c development : added middle to file 2
pick 3e97637 development : added middle and end to file 3
pick 000bc1c feature-x : added file 1
pick e4e6aed feature-x : added the end to file 1
pick 6b0f943 feature-x : added file 4

# Rebase 85afe13..dad96f0 onto 85afe13
```
    
You then save/close your editor session.  git will apply the history to your current branch as you indicated.

```
Successfully rebased and updated refs/heads/feature-x-always-merged-development-interactive-rebased.

[feature-x]$ git log --oneline
e5af985 feature-x : added file 4
e8cdbe3 feature-x : added the end to file 1
5c24b26 feature-x : added file 1
3e97637 development : added middle and end to file 3
d21db1c development : added middle to file 2
dca2baa development : added file 3
e042f4c development : added file 2
47ae98d development : added file 0
85afe13 Initial commit
```
    
The history of `feature-x` is now identical to that of Strategy 2, so when we merge it with `development`, we will have the clean coherent history demonstrated there:

```
#
# Clean commit history on development since we cleaned up feature-x with rebase -i
#
[development-received-feature-x-merged-but-rebased]$ git log --oneline

e5af985 feature-x : added file 4
e8cdbe3 feature-x : added the end to file 1
5c24b26 feature-x : added file 1
3e97637 development : added middle and end to file 3
d21db1c development : added middle to file 2
dca2baa development : added file 3
e042f4c development : added file 2
47ae98d development : added file 0
85afe13 Initial commit
```
    
### A few words about squashing

Given we are about to merge `feature-x` into `development` and we have a `feature-x` history as show here

```
[feature-x]$ git log --oneline
e5af985 feature-x : added file 4
e8cdbe3 feature-x : added the end to file 1
5c24b26 feature-x : added file 1
3e97637 development : added middle and end to file 3
d21db1c development : added middle to file 2
dca2baa development : added file 3
e042f4c development : added file 2
47ae98d development : added file 0
85afe13 Initial commit
```
    
Many projects (especially those who receive a great deal of contributions), mandate that any contribution to be applied to the mainline branch branch (i.e. development, master) be a **single commit**.  

Also if you are on a team following agile practices, branches should be at the 'story' size and thus capable of being contained in one commit.  This ***single commit per contribution***, make management of the mainline branch that much easier.

Here our `feature-x` contribution is comprised of 3 commits. We can easily change that with *interactive rebasing*.  (note: there is also a git `squash` command to accomplish this, but here we will demonstrate the more versatile `rebase -i`)

```
# 
# feature-x ready to be squashed
#
[feature-x]$ git log --oneline
e5af985 feature-x : added file 4
e8cdbe3 feature-x : added the end to file 1
5c24b26 feature-x : added file 1
3e97637 development : added middle and end to file 3
d21db1c development : added middle to file 2
dca2baa development : added file 3
e042f4c development : added file 2
47ae98d development : added file 0
85afe13 Initial commit
```

Initiate the rebase

    git rebase -i head~3
    
The rebase session is shown

```
pick 2249bf8 feature-x : added file 1
pick 5521ab7 feature-x : added the end to file 1
pick 6b6fa9f feature-x : added file 4

# Rebase 3e97637..6b6fa9f onto 3e97637
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```
    
We simply pick the first commit and set the next two to squash (s)

```
pick 2249bf8 feature-x : added file 1
s 5521ab7 feature-x : added the end to file 1
s 6b6fa9f feature-x : added file 4

# Rebase 3e97637..6b6fa9f onto 3e97637
#
```
    
Then save/exit this editor session and you are presented with an editor session that enables you to edit the concatenated
commit messages. *Note, the commented lines (#) will not be part of the commit message.*

```
# This is a combination of 3 commits.
# The first commit's message is:
feature-x : added file 1

# This is the 2nd commit message:

feature-x : added the end to file 1

# This is the 3rd commit message:

feature-x : added file 4

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# Not currently on any branch.
# You are currently editing a commit during a rebase.
#
# Changes to be committed:
#   (use "git reset HEAD^1 <file>..." to unstage)
#
#       new file:   file1.txt
#       new file:   file4.txt
#
```

We do a little cleanup on the message, 

```
feature-x
  - added file 1 with ending
  - added file 4
# Please enter the commit message for your changes. Lines starting
```

then save/exit this editor session.

```
[feature-x]$ git log --oneline
9dafc86 feature-x   - added file 1 with ending   - added file 4
3e97637 development : added middle and end to file 3
d21db1c development : added middle to file 2
dca2baa development : added file 3
e042f4c development : added file 2
47ae98d development : added file 0
85afe13 Initial commit
```
    
We then merge `feature-x` onto `development`

    [development-received-feature-x-squashed]$ git merge feature-x-squashed
    
And see that it's contribution is comprised of a single commit

```bash
[development-received-feature-x-squashed]$ git log --oneline
9dafc86 feature-x   - added file 1 with ending   - added file 4
3e97637 development : added middle and end to file 3
d21db1c development : added middle to file 2
dca2baa development : added file 3
e042f4c development : added file 2
47ae98d development : added file 0
85afe13 Initial commit
```

## Summary

I believe strategy 3 represents what most developers evolve to once they've used git for a while.

Simply stated:

> In order to ensure clean, consise mainline branch (development and/or master) commit history become skilled at *cleaning up* feature branch history prior to mering back to the the main branch.

When talking about merging contributions to a mainline branch, I'd be remiss if I did not mention [Github Pull Requests](https://help.github.com/articles/using-pull-requests).  This is an excellent work-flow that allows contributers to 'queue' commits for a mainline branch.  They can be utilized for gating write access to a repository and/or as a vehicle for 'peer reviews' of contribution prior to merging.



