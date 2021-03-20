# git - distributed revision control system

## git help

- https://github.com/github/training-kit/blob/master/downloads/de/github-git-cheat-sheet.md
- book: https://git-scm.com/book/en/v2

## git with bash

~~~bash
cd ; cd source
git clone https://github.com/<gituser>/<project>  [./source/new-folder]
cd <project>
~~~

or with `ssh`

~~~bash
# remoteuser must exist on host example.com
# remoteuser can have a public ssh key
# public key has be 
# - uploaded to host via gui into the <project>
#   - Settings: Deploy Keys: Add Deploy Key
# - or uploaded to host via gui in the global user settings (for every project)
#   - Settings: SSH Keys: Add Key
git clone remoteuser@example.com:<gituser>/<project>  [./source/new-folder]

# ssh on a different port
git clone ssh://gituser@localhost:3400/<gituser>/<project>

~~~

or with wget

~~~bash
wget https://github.com/harbaum/TouchUI/archive/master.zip
7z x master.zip
mv <project>-master <project>
rm master.zip
~~~

## create a git repo

~~~
# goto new directory eg ./project
git init
git add README.md
git config user.email email@example.com
git config user.name "E. Mail"
git commit -m "first commit"
git remote add origin gituser@git.example.com:user/project.git
git push
### XXX git push -u origin master?
~~~


## interesting git repos

* https://github.com/rwgresham/maltrail
* https://github.com/tuantm8/\_docs
* https://github.com/MarkBaggett (python), see also: https://gist.github.com/MarkBaggett
* https://github.com/jwasham/coding-interview-university


## schema

~~~bash

local-dir                    local-dir/.git   |
                                              |
working        staging       local            |     remote
dir            area          repository       |     repository
                                              |
<----------- git clone -----------|<------- git clone----|              initial
                                              |                   clone of repo
                                              |
<----------- git pull ------------|<------- git pull ----|         'update', no 
                                              |               parameters needed
                                              |
|--- git add --->|
                 |---git commit-->|
                                  |------ git push ----->|
~~~

## update local git code (pull from remote repository)

~~~bash
cd source/repo
git pull
~~~

## change code, add to repository, commit and push

~~~bash
git status                                        # shows status of all files
git diff --name-only                              # shows all changed files
#git add .                                         # add all files in all dirs
git commit -m 'new function xy added' ./src/bla.c
git push

# add a new file in an existing directory
vi new-file-in-existing-dir.md
git add new-file-in-existing-dir.md
git commit -a -m "new file in existing dir"
git push

  # Bemerkung: mache ich kein git add vorher, dann kommt beim git push
  # die Fehlermeldung:
  #
  # Untracked files:
  #   new-file-in-existing-dir.md


# add a new directory to the git
mkdir new-dir
vi new-dir/new-file.md
git add new-dir
git commit -a -m "neues Verzeichnis mit neuer Datei"
git push

# commit messages

fix: a bug fix, eg issue 1
feat: new feature name
refactor: a code change that neither fixes a bug nor adds a feature
docs: documentation only changes
test: add missing tests or correcting existing tests
perf: a code change that improves performance


# move files: do not use local _mv_ command
git mv image.gif ./images/newimage.gif

# add/commit/push with a function
function git-update() {
  git add . 
  git commit -m "$( echo "$@" )"
  echo "- pushing repo"
  git push
  # if we have tags, push the tags
  if ( git tag | grep -q . ) ; then
    echo "- pushing tags"
    git push --tags
  fi
}
git-update function xy added

# message when you do your _first commit_ to a repo
*** Please tell me who you are.
Run

  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"

to set your accounts default identity.
Omit --global to set the identity only in this repository.
~~~

## git configuration for local repo or global

- with option `--global`, values will be stored globally: `~/.gitconfig`
- without `--global`, values will be stored in            `.../repo/.git/config`

~~~
# configure email and full name. 
git config [--global] user.email "your.name@example.com"
git config [--global] user.name  "Your Name"

# get the remote url
git config --get remote.origin.url


# proxy

# configure proxy globally and delete proxy for a repo
git config --global http.proxy http://user:pass@proxy.server.com:port
cd ~/git/repo; git config http.proxy ""  
# proxy for a specific domain
git config --global http.https://domain.com.proxy http://proxyUsername:proxyPassword@proxy.server.com:port
git config --global http.https://domain.com.sslVerify false


# first push
git config --global push.default current
# see git help config and search for 'push.default'
~~~

## show git config

~~~
echo "GLOBAL" ; git config --global -l ; echo LOCAL ; git config -l
~~~

## git with ssh

- create an ssh key
- add the public key to the repo (or to the user)
- start ssh agent and add key to the agent
- test, if it is successful (with ssh -T user@githost

~~~
$ ssh -T gituser@githost.example.com 2>&1 | grep successfully
Hi there, You've successfully authenticated, but Gogs does not provide shell access.
~~~

## git tags

Git supports two types of tags: `lightweight` and `annotated`.

A `lightweight` tag is like a branch that doesn't change. It is just a pointer to a specific commit.

A `annotated` tag is stored as full objects in the git database, contain the tagger name, email and date and have a tagging message.

Create tags with

~~~bash
# create a lightweight tag / add tag
git tag v1.0
# the tag is not seen in the repo, until you do a
git push --tags             # push _all_ tags
git push origin v1.0

# create an annotated tag
git tag -a v1.1 -m "My Ver 1.1"

# show tags
# if it is an annotated tag, it will show Tagger: Name...
git show v1.0

# list all defined tags
git tag
git tag -l "v1.*"        # show all tags with v1...

# now you can cange files/make tests, commit and push
# to revert to tag v1.0, use
git checkout v1.0

# to go to the actual version, use
git checkout master

# delete a tag
git tag -d v2.4                  # delete tag in local repo
git push origin --delete v2.4    # delete tag on remote server
~~~

## make branches for development

~~~
# list all branches
git branch -a
git checkout -b dev              # create a new branch and checkout
git branch -a
* dev
vi dev.txt
git add dev.txt
git commit -m "new file dev.txt in dev branch"
git checkout master
ls -al dev.txt         # <<< error: file is not here any more!
git checkout dev
ls -al dev.txt         # <<< file is here

# merge: when merging, we need to be on the branch
# that we want to _merge to_
git checkout master
git branch
#  $ git branch
#   dev
# * master              # <<< we are on 'master' branch

# get current branch
git rev-parse --abbrev-ref HEAD

# merge all commits from the dev branch into master
git merge dev --no-ff   # --no-ff tells git to retain all of
                        # the commit messages in the dev
                        # branch we did earlier

git push  # updating master

# cleanup
git branch -d dev       # delete dev branch
                        # git will throw an error if branch
                        # is not yet merged
~~~

## misc git commands: log blame diff 

~~~bash
git log            # shows author, date and the comments
git log --oneline --decorate --graph
git log --oneline --decorate --graph -n 10
git log --graph --pretty=format:'%C(bold)%h%Creset%C(magenta)%d%Creset %s %C(yellow)<%an> %C(cyan)(%cr)%Creset' --abbrev-commit

# git diff arguments
<file>                       # one file
<tag>                        # differences between tag1 and now
<tag1> <tag2>                # differences between two tags
<tag1> <tag2> <file> [...]   # differences between two tags, one or more files

# git diff options
git diff --name-only         # just show the names
git diff --name-status       # show name and status
                   # stati

                   # A Added
                   # C Copied
                   # D Deleted
                   # M Modified
                   # R Renamed
                   # T Type changed (T), regular file, symlink, ...
                   # U Unmerged
                   # X Unknown
                   # B Broken pairing

git blame file     # shows every line, with version of author of each line
                   # shows also changes which are not yet commited

git remote show    # shows eg 'origin', see get-url

git remote get-url origin  # get the git path
~~~


## edit git config

~~~bash
git config --edit             # local config file <project>/.git/config
git config --global --edit    # global config file ~/.gitconfig
~~~

## git via proxy

~~~bash
# write in config file (ommit --global for local configuration)
git config --global http.proxy http://proxy.example.com:8080
git config --global https.proxy http://proxy.example.com:8080

# specify proxy on the commandline
git clone -c http.sslVerify=false \
  -c http.proxy='' https://localhost:3000/user/repo
~~~


## archive - make an archive/backup

~~~
lasttag="$(git tag | tail -1)"
branch="$( git branch | grep "^\*" | awk '{print $2}' )"
branch="$( git symbolic-ref --short HEAD )"
repo="$( basename $( git rev-parse --show-toplevel ) )"
aname="${repo}-${branch}-${lasttag}.zip"
echo "create $aname"
git archive --format zip --output $aname $branch; ls -al $aname
~~~

## `.gitignore` file

~~~git
# Text files and backups
.*.swp
.bak
*~

# Byte-compiled / optimized / DLL files
__pycache__/
*.py[cod]
*$py.class

# C extensions
*.so
*.o

# ignore everything in the root except the "wp-content" directory.
!wp-content/
# ignore everything in the "wp-content" directory, except:
# "mu-plugins", "plugins", "themes" directory
wp-content/*
!wp-content/plugins/

# ignore logfiles, databases
*.log
*.sql
*.sqlite
~~~



## git on the gitserver

see also A Little Of Git's Internals (https://cbx33.github.io/gitt/afterhours2-1.html) (download pdf from: https://cbx33.github.io/gitt/download.html).

~~~
object files in ./repo-name.git/objects
./03/a7c67839f70a1782a8195ca33eb45c81e71ec7
./40/f74aa481af79cf6acebc4a243f8108f7c7df03

./repo-name.git/objects$ git cat-file -p 03a7c67839f70a1782a8195ca33eb45c81e71ec7
100644 blob 425ef4fef90e8ed30b280a7dd9c5904e7882b2a7    README.md
040000 tree fdc19cd8f4f9c03ed57b8c8683b9992ab885c2c7    src
./repo-name.git/objects$ git cat-file -p 40f74aa
tree b63f3a784862ba7c983c650da291a75523d75641
parent f03f66c756315a0317585be575cb098924fc47be
author user <user.name@example.com> 1554274835 +0200
committer user <user.name@example.com> 1554274835 +0200

initial upload

file types with find?


$ find . -type f | while read f ; do f=${f//[\/\.]} ; echo -n "$f "; git cat-file -p $f | file - ; done
ec7fef9f534fd1c31069ff27ca7b074c0249591d /dev/stdin: ASCII text
cc4262e5b3e156ab85bdc8030c78a00110287335 /dev/stdin: ASCII text
f03f66c756315a0317585be575cb098924fc47be /dev/stdin: ASCII text
0c96b57cedac9cc2dfa1a4046cdeca1dd6d46606 /dev/stdin: ISO-8859 text, with CRLF line terminators
319cba7f3d2c0dc652d57bf8712f276b4264e03d /dev/stdin: ASCII text
61f00c4443e86c10940e3c53e47ff8996b31ea5d /dev/stdin: PE32 executable (GUI) Intel 80386, for MS Windows
425ef4fef90e8ed30b280a7dd9c5904e7882b2a7 /dev/stdin: UTF-8 Unicode text

$ gc -p 03a7c67839f70a1782a8195ca33eb45c81e71ec7
100644 blob 425ef4fef90e8ed30b280a7dd9c5904e7882b2a7    README.md
040000 tree fdc19cd8f4f9c03ed57b8c8683b9992ab885c2c7    src
~~~


## testing

~~~
git for-each-ref --format="%(authorname) %09 %(if)%(HEAD)%(then)*%(else)%(refname:short)%(end) %09 %(creatordate)" refs/remotes/ --sort=authorname DESC

https://github.com/Bash-it/bash-it/blob/master/aliases/available/git.aliases.bash
~~~
