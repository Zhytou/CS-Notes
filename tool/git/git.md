# Git

## Basical Concepts

![git workflow](../img/git_workflow.jpg)

## Usage

### Simple example at a glance

This example presents how initialize a local git repo and bind it with several remote repo.

``` bash
# create an empty git repo
git init

# add a remote repo and get the existed files from the remote repo
git remote add public https://github.com/xxx/project.git
git fecth public
git merge public/master

# first commit
git add .
git push -m "first commit"

# add your personal remote repo and set the upstream 
git remote add origin https://github.com/Zhytou/project.git
git push -u origin master

```

### Use .gitignore

change .git/info/exclude to make .gitignore work

``` bash
cat .gitignore > .git/info/exclude
```

### Other useful commands

git status: check the files state(untracked or unstaged)

git checkout: change the branch

git branch -a: show all the branches you created including remote

git diff xxx: show differences between two files