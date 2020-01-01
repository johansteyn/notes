Git Notes
=========

Experimenting with Git on a single machine...

Creating and Cloning
--------------------

### Create a "bare" remote repo:
`$ git init --bare helloworld.git`

### Clone a local repo for Alice and add new file1 in the master branch:
`$ mkdir alice`
`$ cd alice`
`$ git clone ../helloworld.git`
`$ cd helloworld`
`$ echo "line1" > file1.txt`
`$ git diff`
`$ git add file1.txt`
`$ git diff --staged`
`$ git commit -m "Alice: added file1 to master branch."`
`$ git push`


Some handy aliases
------------------
`alias gs='git status -s'`
`alias gr='git remote show origin'`
`alias gb='git branch -a'`
`alias gl='git log --all --oneline --decorate --graph'`
`alias glr='git log --all --oneline --decorate --graph --remotes'`
`alias gll='git log --all --decorate --graph --abbrev-commit --format=format:'\''%C(yellow)%h%C(reset) - %C(cyan)%ad%C(reset) %C(green)(%ar)%C(reset)%C(red)%d%C(reset)%n'\'''\''          %C(red)%an: %C(bold) white)%s%C(reset)'\'''`

