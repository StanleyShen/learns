# Version Control
Version control is a system that records changes to a file or set of files over time so that you can recall specific versions later.
There are kinds of version control system:
*	Local version control system
	*	just copy files into another directory(perhaps a time-stamped directory)
*	Centralized version control system
	*	CVS
	*	Subversion
	*	Perforce
*	Distributed version control systems - they fully mirror the repository
	*	Git
	*	Mercurial

#Git basic concepts
*	snapshots, not differences
	*	The major difference between git and other VCs is the way Git thinks about its data. Conceptually, most other systems store informatin as a list of file-based changes, they think of the information they keep as a set of files and the changes made to each file over time. ![Other VCs](https://github.com/StanleyShen/learns/raw/master/git/images/VCs-changes.png)
	*	Git thinks of its data more like a set of snapshots of a miniature filesystem. Every time you commit/save the state of your project in Git, it basically takes a pricture of what all your files look like at that moment and stores a reference to that snapshot. To be efficient, if files are not changed, Git doesn't store it again, just a link to the previous identical file it has already stored. ![Git](https://github.com/StanleyShen/learns/raw/master/git/images/Git-changes.png)

*	Nearly every operations is local
	* Most operations in Git only need local files and resources to operate - generally no information is needed from another computer on your network.

*	Git has integrity (SHA-1 hash)
	*	Everything in Git is check-summed before it is stored and is then referred to by that checksum, which means it's impossible to change the contents of any file/directory without Git knowing about it.  This is a 40-character string composed of hexadecimal characters (0-9 and a-f) and calculated based on the content of a file/directory structure in Git. In fact, Git stores everyting in its database not by file name but by the hash values of its contents.

*	Git generally only adds data
	*	When you do actions in Git, nearly all of them only add `data` to the Git database. It's hard to get the system to do anything that is not undoable or to make it erase data in any way.

*	The three states
	*	Git has three main states that your files can reside in: committed, modified and staged.
		*	Committed: the data is safely stored in your local database.
		*	Modified: you have changed the file but have not committed it to your database yet.
		*	Staged: you have marked a modified file in its current version to go into your next commit snapshot.
		
		This leads us to three main sections of a Git project: The Git directory, the working directory and the staging area. ![](https://github.com/StanleyShen/learns/raw/master/git/images/three-states.png)
		*	Git directory - where Git stores the metadata and object database for your project.
		*	working directory - a single checkut of one version of the project, these files are pulled out of the compressed database in the Git directory and placed on disk for you to use/modify.
		*	staging area - a file, generally contained in your Git directory, that stores information about what will go into your next commit. It's sometimes referred to as the `index`.

		The basic Git workkflow goes something like this:
		*	Modify files in your working directory
		*	Stage the files, adding snapshots of them to your staging area
		*	Do a commit, which takes the files as they are in the staging area and stores that snapshot permanently to your Git directory.

#Git basic operations
*	starting to track an existing project in Git
```
git init xx
```

*	cloning an existing repository
```
git clone https://github.com/libgit2/libgit2 new-name
```

*	checking the status of your files
![](https://github.com/StanleyShen/learns/raw/master/git/images/file-lifecycle.png)
```
git status
```

*	tracking new files/staging modified files
```
git add new-file
git add modified-file
```

*	ingoring files
```
cat .gitignore
```

*	checking changes
```
git diff   # compares what is in your working directory with what is in your staging area
git diff --staged   # compares your staged changes to your last commit, --cached is same
```

*	committing your changes
```
git commit
git commit -a -m 'stage every file that is already tracked before doing the commit, letting you skip the git add part '
```

*	removing files
```
git rm xxx
```

*	moving files
```
git mv from to
```

*	viewing the commit history
```
git log -p -2   # -p shows the differences and -2 limits to last 2 entries
git log --stat   $ --stat shows some abbreviated stats for each commit
git log --pretty= #change output format other than the default
					oneline
					format:"%h - %an, %ar : %s"
useful options are:
%H   	commit hash
%h 	 	abbreviated commit hash
%T 		Tree hash
%t 		abbreviated tree hash
%P 		Parent hash
%p 		abbreviated parent hash
%an 	author name
%ae 	author email
%ad 	author date(format respects the --date=option)
%ar 	author date, related
%s 		subject

It also has cn/ce/cd/cr, which is related to committer.
The author is the person who originally wrote the work, whereas the committer is the person who last appled the work.

--graph shows the branch and merge history too.

git log --since=2.weeks  # the last 2 weeks
git log --until=2.weeks  $ 2 weeks ago
```

*	Undoing things
```
git commit --amend  #amend last commit instead of adding a new commit
```

*	Unstaging a staged file
```
git reset HEAD file  #unstage a staged file
```

*	Unmodifying a modified file
```
git checkout file
```

# Working with remotes
*	Listing remotes
```
git remote -v   # -v shows you the URLs that Git has storedfor the shortname.
```

*	Adding remotes
```
git remote add pb https://github.com/meteorping
```

*	Fetching/Pulling from remotes
```
git fetch [remote-name] #pulls down all the data from that remote project that you don't have yet.
git pull remote branch # fetch data and merge into code you're working on
```

*	Pushing to remote
```
git push origin master
```

*	Removing/Renaming remotes
```
git remote rename pb paul
git remote rm paul
```

# Tagging
*	Listing tags
```
git tag
git tag -l "v1.8*"
```

*	Creating tags
Git uses two main types of tags: lightweight and annotated.
	*	lightweight tag is very much like a branch that doesn't change - it's just a pointer to a specific commit
	*	annotated tag, stores as full objects in Git databases. They're checksummed, contains the tagger name, email and date and message.
```
git tag -a v1.4 -m 'version 1.4'		#annotated tag
git tag v1.4-lw    						#lightweight tag without a/s/m options
git tag -a v1.2 9fceb02					#checksum 9fceb02 is(part of)  
```

*	Sharing tags
By default, the git push doesn't transfer tags to remote servers. 
You will have to explicitly push tags to a shared server after you have created them.
```
git push origin v1.5
git push origin --tags  # push all tags which is not existed in remote
```

*	Checking out tags
```
git checkout -b version2 v2.0.0  # checkout tag v2.0.0 to branch version2
```

# Git Aliases
```
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
```

# Git branch
