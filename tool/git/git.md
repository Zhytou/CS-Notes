# Git

- [Git](#git)
  - [Basical Concepts](#basical-concepts)
  - [Usage](#usage)
    - [Simple example at a glance](#simple-example-at-a-glance)
    - [Use .gitignore](#use-gitignore)
    - [Other useful commands](#other-useful-commands)
  - [Other](#other)

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
git commit -m "first commit"

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

## Other

**git branch vs git tag**：

Both tags and branches point to a commit, they are thus aliases for a specific hash and will save you time by not requiring to type in a hash.

The difference between tags and branches are that a branch always points to the top of a development line and will change when a new commit is pushed whereas a tag will not change. Thus tags are more useful to "tag" a specific version and the tag will then always stay on that version and usually not be changed.

**set user name and email**:

``` bash
# make sure the email same as your github registered email, it's important to count your contribution
git config --global user.name "Zhytou"
git config --global user.email "zhoytou@gmail.com"
```

**use ssh instead of http**:

``` bash
# generate key
ssh-keygen -t rsa -C "zhoytou@gmail.com"

# show the public key and add it to your personal account
cat id_rsa.pub

# show remote url and change it into ssh
git remote -v
git remote set-url origin git@github.com:Zhytou/project.git

# start ssh-agent
eval `ssh-agent -s`

# add the private key
ssh-add ~/.ssh/id_rsa

# then you're ready to go
```
