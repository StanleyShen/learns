#Branch
Branching means you diverge from the main line of development and contine to do work without messing with main line.
When you make a commit, Git stores a `commit object` which contains a pointer to the snapshot of the content you staged. This object also contains the author's name and email, the message you typed and pointers to the commit(s) that directly came before this commit(its parent or parents): zero parents for the initial commit, one parent for a normal commit, and multiple parents for a commit that results from a merge of two or more branches.

Let's assume that you have a directory containing three files and you stage them all and commit. Staging the files checksums each one, stores that version of the file in Git repository(blobs), and adds that checksm to staging area.

```bash
$ git add README test.rb LICENSE
$ git commit -m 'The initial commit of my project'
```
When you create the commit by running `git commit`, Git checksums each subdirectory and stores those thress objects in Git repository, and creates a `commit object` has the metadata and a pointer to the root project tree so it can re-create that snapshot if needed.

You Git repository contains 5 objects: one blob for the contents of each of your three files, one tree that lists the contents of the directory and specifies which file names are stored as which blobs, and one commit object with the pointer to that root tree and all the commit metadata.

![](https://github.com/StanleyShen/learns/raw/master/git/images/commit-object.png)

If you make some changes and commit again, the next commit stores a pointer to the commit that came immediately before it.

![](https://github.com/StanleyShen/learns/raw/master/git/images/git-tree.png)

A branch in Git simply a lightweight movable pointer to one of these commits. The default branch is `master`. As you start making commits, you're given a master branch that points to the last commit you made.

![](https://github.com/StanleyShen/learns/raw/master/git/images/git-tree.png)

#branch operations
###creating branch

```bash
git branch testing   #create a branch but will not switch to it

git log --oneline --decorate  # decorate option helps to see where branch pointers are pointing.
```

![](https://github.com/StanleyShen/learns/raw/master/git/images/create-branch.png)

How does Git know what branch you're currently on? It keeps a special pointer call `HEAD`.

###switching branch
```bash
	git checkout testing
```
![](https://github.com/StanleyShen/learns/raw/master/git/images/switch-branch.png)
```bash
	echo "hello world" >> test.rb
	git commit -a -m 'change'
```
![](https://github.com/StanleyShen/learns/raw/master/git/images/switched-branch-moving.png)
```bash
	git checkout master
```
![](https://github.com/StanleyShen/learns/raw/master/git/images/switch-branch-backing.png)

When we turn back to `master` branch, it did two things. It moved HEAD pointer back to point to master branch and reverted the files in your working directory back to the snapshot that master points to.
And let change things on master again.

```bash
	cat "test" >> test.rb
	git commit -a -m 'change on master'
```

![](https://github.com/StanleyShen/learns/raw/master/git/images/branch-diverge.png)

Because a branch in Git is just a simple file contains the 40 characters SHA-1 checksum of the commit it points to, branches are cheap to create and destroy. Creating a branch is as quick/simple as writing 41 bytes to a file.

###Basic Branching and Merging

![](https://github.com/StanleyShen/learns/raw/master/git/images/git-merge.png)

In order to apply hotfix to master, we need to merge its change in master.
```bash
$ git checkout master
$ git merge hotfix
Updating f42c576..3a0874c 
Fast-forward
```
`Fast-Forward' it becauses the commit pointed to by the branch you merged in (master) was directly upstream of the commit you're on, Git simply moves the pointer forward. 

![](https://github.com/StanleyShen/learns/raw/master/git/images/fast-forward.png)

Continue to work on `iss53` branch, and merge it back to master.
```bash
$ git checkout iss53
Switched to branch "iss53"
$ vim index.html
$ git commit -a -m 'finished the new footer [issue 53]'
```
![](https://github.com/StanleyShen/learns/raw/master/git/images/back-iss53.png)

```bash
$ git checkout master
Switched to branch 'master'
$ git merge iss53
Merge made by the 'recursive' strategy.
```
In this way, Git found the common ancestor and do the 3-way merging and then create a new snapshot. This is referred to as `merge commit`, and is special in that it has more than one parent.

![](https://github.com/StanleyShen/learns/raw/master/git/images/git-recursive.png)
![](https://github.com/StanleyShen/learns/raw/master/git/images/recursive-merge.png)

In some case, this process doesn't go smoothly. If you changed the same file in two branches.
Then Git will not create a snapshot automatically, you need to fix the merge conflict and create a snapshot.

###Branch management
```bash
git branch  # list all branches
git branch --merged  #you have merged into the branch you are working on
git branch -d testing # delete branch
```

### Remote brahes

### Rebasing
In Git, there are two main ways to integrate changes from one branch into another: merge and rebase.
As we mentioned before, `merge` action will find the common ancestor of both branch, and do a 3-ways merge for both branch, and generate a new snapshot.

![](https://github.com/StanleyShen/learns/raw/master/git/images/git-rebase-01.png)
![](https://github.com/StanleyShen/learns/raw/master/git/images/git-rebase-02.png)

There is another way, we can reabase changes in experiment branch to master branch
```bash
$ git checkout experiment
$ git rebase master
First, rewinding head to replay your work on top of it... 
Applying: added staged command
```
It works by going to the common ancestor of the two branches (the one you’re on and the one you’re rebasing onto), getting the diff introduced by each commit of the branch you’re on, saving those diffs to temporary files, resetting the current branch to the same commit as the branch you are rebasing onto, and finally applying each change in turn.

![](https://github.com/StanleyShen/learns/raw/master/git/images/git-rebase-03.png)

At this point, you can go back to the master branch and do a fast-forward merge.
```bash
$ git checkout master 
$ git merge experiment
```

![](https://github.com/StanleyShen/learns/raw/master/git/images/git-rebase-04.png)

Rebasing makes for a cleaner history. If you examine the log of a rebased branch, it looks like a linear history: it appears that all the work happened in series, even when it originally happened in parallel.
Note that the snapshot pointed to by the final commit you end up with, whether it’s the last of the rebased commits for a rebase or the final merge commit after a merge, is the same snapshot

#####More Interesting Rebases
![](https://github.com/StanleyShen/learns/raw/master/git/images/git-rebase-05.png)
At this time, you want to merge client code(C8, C9) but not server side changes.

```bash
$ git rebase --onto master server client
```
It means “check out the client branch, figure out the patches from the common ancestor of the client and server branches, and then replay them onto master.”

![](https://github.com/StanleyShen/learns/raw/master/git/images/git-rebase-06.png)

And then you can fast-forward your master branch
```bash
$ git checkout master 
$ git merge clie
```

Then add server change to master
```bash
$ git rebase master server
```
This replays your server work on top of your master work.
![](https://github.com/StanleyShen/learns/raw/master/git/images/git-rebase-07.png)

```
$ git checkout master 
$ git merge server
```
![](https://github.com/StanleyShen/learns/raw/master/git/images/git-rebase-08.png)