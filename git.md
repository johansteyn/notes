Git Notes
=========

Experimenting with Git on a single machine...

Creating and Cloning
--------------------

Create a "bare" remote repo:
```
$ git init --bare helloworld.git
```

Clone a local repo for Alice and add new file1 in the master branch:
```
$ mkdir alice
$ cd alice
$ git clone ../helloworld.git
$ cd helloworld
$ echo "line1" > file1.txt
$ git add file1.txt
$ git diff --staged
$ git commit -m "Alice: Added file1 to master branch."
$ git push
```

Clone a local repo for Bob, edit file1, add new file2, check diffs before/after committing - all in the master branch:
```
$ cd ../..
$ mkdir bob
$ cd bob
$ git clone ../helloworld.git
$ cd helloworld
# NOTE: Not strictly necessary to do a fetch here, but it gives us a FETCH_HEAD to use...
$ git fetch
$ echo "line2" >> file1.txt
$ echo "line1" > file2.txt
$ git diff
$ git diff --stat
$ git diff --name-status
$ git add file1.txt file2.txt
$ git diff HEAD
$ git diff HEAD --stat
$ git diff HEAD --name-status
$ git commit -m "Bob: Appended line2 to file1 and added file2 to master branch."
$ git diff origin/master master
  OR
  git diff origin/master..
  OR
  git diff origin/master
  OR
  git diff @{upstream}
  OR
  git diff @{u}
  OR
  git diff FETCH_HEAD
$ git log --oneline
$ git log -p --branches --not --remotes
$ git push
```

Merging
-------
Back to Alice, see what changes were pushed to the remote repo, and then merge them into her local repo:
```
$ cd ../../alice/helloworld
$ git fetch
$ git diff master origin/master
  OR
  git diff ..origin/master
  OR
  git diff ..@git push{upstream}
  OR
  git diff ..@{u}
  OR
  git diff ..FETCH_HEAD
$ git diff ..FETCH_HEAD --stat
$ git diff ..FETCH_HEAD --name-status
$ git log --oneline
$ git log --oneline --remotes
$ git merge origin/master
NOTE: No need to specify origin/master (will leave it out from here on...)
```
Now Alice and Bob are identical

Branching and Rebasing
----------------------
Back in Bob, create a feature1 branch and append to file2:
```
$ cd ../../bob/helloworld
$ git branch feature1
$ git checkout feature1
$ echo "line2" >> file2.txt
$ git add file2.txt
$ git commit -m "Bob: Appended line2 to file2 in feature1 branch."
```
Back in Alice, make further (non-conflicting) change to master branch, and push:
```
$ cd ../../alice/helloworld
$ echo "line3" >> file1.txt
$ git add file1.txt
$ git commit -m "Alice: Appended line3 to file1 in master branch."
$ git push
```
Back in Bob, fetch Alice's changes and merge them into master:
```
$ cd ../../bob/helloworld
$ git fetch
$ git checkout master
$ git diff master origin/master
$ git merge
```
Rebase to incorporate Alice's changes into the feature branch:
```
$ git log --oneline --graph --all
$ git checkout feature1
$ git rebase master
```
Make further changes, then push the feature branch to remote for Alice to review Bob's two commits:
```
$ echo "line4" >> file2.txt
$ git add file2.txt
$ git commit -m "Bob: Appended line4 to file2 in feature1 branch"
$ git push --set-upstream origin feature1
```

Reviewing and Cherry-picking
----------------------------
Back in Alice, fetch the feature branch and cherry-pick individual commits:
```
$ cd ../../alice/helloworld
$ git fetch
$ git log --oneline --remotes --all
```
Checkout the feature1 branch to review and make further changes to file1:
```
$ git branch -t feature1 origin/feature1
$ git checkout feature1
```
Alice notices that Bob made a mistake - he added line4 instead of line3 to file2, so remove it:
```
$ cat file2.txt | fgrep -v "line4" > file2.txt
$ git add file2.txt
$ git commit -m "Alice: Removed incorrect line4 from file2 in feature1 branch."
```
Then go ahead and append the correct line:
```
$ echo "line3" >> file2.txt
$ git add file2.txt
$ git commit -m "Alice: Appended line3 to file2 in feature1 branch."
```
Switch back to master and cherry-pick only the two correct commits, where `<sha1>` and `<sha2>` refer to the IDs of Bob and Alice's respective commits:
```
$ git checkout master
$ git diff master feature1
$ git log --oneline --graph --all
$ git cherry-pick <sha1>
$ git cherry-pick <sha2>
```
Note that Alice doesn't need to add or commit anything - she has already cherry-picked the commits she wants.
So, she can simply go ahead and push her master branch:
```
$ git push
```

Deleting Branches
-----------------
Alice can now delete the feature branch, in both local and remote.

To delete a local branch:
```
$ git branch -D feature1
```
Note that -D is shorthand for -d with --force.
Using -d will only delete a branch if it has already been merged into master.

To delete the remote branch:
```
$ git push origin --delete feature1
```
Back in Bob, fetch changes and merge into master:
```
$ cd ../../bob/helloworld
$ git fetch --prune
$ git checkout master
$ git merge
```
Note that he needs to fetch with the --prune option to delete his remote feature branch.

Now he can delete his local feature1 branch:
```
git branch -D feature1
```
Alice and Bob are again identical.

Squashing
---------
Create a feature2 branch, make four commits, squash them into a single commit, and merge into master:
```
$ cd ../../alice/helloworld
$ git branch feature2
$ git checkout feature2
$ echo "line5" >> file1.txt
$ git add file1.txt
$ git commit -m "Alice: Appended line5 to file1 in feature2 branch."
```
Oops! Alice made a mistake...
She was supposed to append line4, so fix it:
```
$ cat file1.txt | fgrep -v "line5" > file1.txt
$ git add file1.txt
$ git commit -m "Alice: Removed incorrect line5 from file1 in feature2 branch."
$ echo "line4" >> file1.txt
$ git add file1.txt
$ git commit -m "Alice: Appended line4 to file1 in feature2 branch."
$ echo "line4" >> file2.txt
$ git add file2.txt
$ git commit -m "Alice: Appended line4 to file2 in feature2 branch."
$ git log --oneline --remotes --all
```
Clean up the feature branch before merging into master:
```
$ git rebase -i master
# Edited the interactive session file to "drop" the first two commits, "pick" the third and "squash" the fourth.
# Edited the final commit message to replace all the commit messages with a single one:
#  Alice: Appended line4 to both file1 and file2 in feature2 branch.
$ git log --oneline --remotes --all
```
Now that the feature2 branch is clean, merge it into master:
```
$ git checkout master
$ git merge feature2
```
No need for the feature2 branch anymore, so delete it:
```
$ git branch -d feature2
$ git push
```
Back to Bob's repo to merge in the latest changes:
```
$ cd ../../bob/helloworld
$ git fetch
$ git merge
```
Now Alice and Bob are again identical.


Some handy aliases
------------------
```
alias gs='git status -s'
alias gr='git remote show origin'
alias gb='git branch -a'
alias gl='git log --all --oneline --decorate --graph'
alias glr='git log --all --oneline --decorate --graph --remotes'
alias gll='git log --all --decorate --graph --abbrev-commit --format=format:'\''%C(yellow)%h%C(reset) - %C(cyan)%ad%C(reset) %C(green)(%ar)%C(reset)%C(red)%d%C(reset)%n'\'''\''          %C(red)%an: %C(bold white)%s%C(reset)'\'''
```

Tips
----
* To discard changes that have not been added yet, simply checkout the file(s), eg:
  ```
  $ git checkout <path-to-file>
  ```
  For more info on undoing things... 
  https://git-scm.com/book/en/v2/Git-Basics-Undoing-Things
* More tips...
