# Git

## Basical Concepts

###

## Usage

### Simple example at a glance

This example presents how initialize a local git repo and bind it with several remote repo.

``` bash
git init
git remote
git remote add origin https://github.com/Zhytou/project.git
git push -u origin master

```

Patterns which are specific to a particular repository but which do not need to be shared with other related repositories (e.g., auxiliary files that live inside the repository but are specific to one user's workflow) should go into the $GIT_DIR/info/exclude file.

## change .git/info/exclude to make .gitignore work

``` bash
cat .gitignore > .git/info/exclude
```

### other useful commands

git status

git checkout