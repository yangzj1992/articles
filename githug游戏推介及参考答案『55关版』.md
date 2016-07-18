title: githug游戏推介及参考答案『55关版』
date: 2016-01-04 00:21:21
categories: Git
tags: [Git]
---

# 游戏简介
这几天看到一个 git 的游戏—— [githug](https://github.com/Gazler/githug)，试了下发现可以帮助大家熟悉 git 的一些常用操作,试着做了一下感觉还不错，通关过程基本没有什么太大的难点，这里做一个推荐并把自己通关的过程记录一下方便后人查看。

当然这些游戏练习只是帮助熟悉，难度基本属于日常操作，如果是git新手最好的学习方法还是先看一遍 [git 文档](https://git-scm.com/book/zh/v2)

githug 题目 list（更新日期为2016-01-03）

```
#1: init
#2: config
#3: add
#4: commit
#5: clone
#6: clone_to_folder
#7: ignore
#8: include
#9: status
#10: number_of_files_committed
#11: rm
#12: rm_cached
#13: stash
#14: rename
#15: restructure
#16: log
#17: tag
#18: push_tags
#19: commit_amend
#20: commit_in_future
#21: reset
#22: reset_soft
#23: checkout_file
#24: remote
#25: remote_url
#26: pull
#27: remote_add
#28: push
#29: diff
#30: blame
#31: branch
#32: checkout
#33: checkout_tag
#34: checkout_tag_over_branch
#35: branch_at
#36: delete_branch
#37: push_branch
#38: merge
#39: fetch
#40: rebase
#41: repack
#42: cherry-pick
#43: grep
#44: rename_commit
#45: squash
#46: merge_squash
#47: reorder
#48: bisect
#49: stage_lines
#50: find_old_branch
#51: revert
#52: restore
#53: conflict
#54: submodule
#55: contribute
```

个人git alias如下
```
alias.co=checkout
alias.ci=commit
alias.st=status
alias.br=branch
```
整个过程中如果暂时想不起来也可以查看提示
```
githug hint
```

# 攻略

---
Name: init
Level: 1
Difficulty: *

```
A new directory, `git_hug`, has been created; initialize an empty repository in it.
```

Answer:

```
$ git init

```
---
Name: config
Level: 2
Difficulty: *

```
Set up your git name and email, this is important so that your commits can be identified.
```

Answer:

```
$ git config --global user.name "yangzj1992" 

$ git config --global user.email yangzj1992@qq.com
```
---
Name: add
Level: 3
Difficulty: *

```
There is a file in your folder called `README`, you should add it to your staging area

Note: You start each level with a new repo. Don't look for files from the previous one.
﻿```


Answer:

```
$ git add README
```
---
Name: commit
Level: 4
Difficulty: *

```
The `README` file has been added to your staging area, now commit it.
```

Answer:

```
$ git ci -m "add"
```
---
Name: clone
Level: 5
Difficulty: *

```
Clone the repository at https://github.com/Gazler/cloneme.
```
Answer:
```
$ git clone https://github.com/Gazler/cloneme
```
---
Name: clone_to_folder
Level: 6`
Difficulty: *

```
Clone the repository at https://github.com/Gazler/cloneme to `my_cloned_repo`.
```

Answer:
```
$ git clone https://github.com/Gazler/cloneme my_cloned_repo
```
---
Name: ignore
Level: 7
Difficulty: **

```
The text editor 'vim' creates files ending in `.swp` (swap files) for all files that are currently open.  We don't want them creeping into the repository.  Make this repository ignore `.swp` files.
```

Answer:
在.gitignore文件里添加

```
*.swp
```
---
Name: include
Level: 8
Difficulty: **

Notice a few files with the '.a' extension.  We want git to ignore all but the 'lib.a' file.

Answer:
在.gitignore文件里添加

```
*.a
!lib.a
```
---
Name: status
Level: 9
Difficulty: *

There are some files in this repository, one of the files is untracked, which file is it?

Answer:

```
$ git st
输入Untracked files 的名字(database.yml)
```

---
Name: number_of_files_committed
Level: 10
Difficulty: *

There are some files in this repository, how many of the files will be committed?

Answer:

```
$ git st
输入Changes to be committed的文件数量（2）
```

---
Name: rm
Level: 11
Difficulty: **

A file has been removed from the working tree, however the file was not removed from the repository.  Find out what this file was and remove it.

Answer:

```
$ git st
$ git rm deleteme.rb
```
---
Name: rm_cached
Level: 12
Difficulty: **

A file has accidentally been added to your staging area, find out which file and remove it from the staging area.  *NOTE* Do not remove the file from the file system, only from git.

Answer:

```
$ git st
$ git rm --cached deleteme.rb
```
---
Name: stash
Level: 13
Difficulty: **

You've made some changes and want to work on them later. You should save them, but don't commit them.

Answer:

```
$ git stash
```
---
Name: rename
Level: 14
Difficulty: ***

We have a file called `oldfile.txt`. We want to rename it to `newfile.txt` and stage this change.

Answer:

```
$ git mv oldfile.txt newfile.txt
```
---
Name: restructure
Level: 15
Difficulty: ***

You added some files to your repository, but now realize that your project needs to be restructured.  Make a new folder named `src` and using Git move all of the .html files into this folder.

Answer:

```
$ mkdir src
$ git mv *.html src
```
---
Name: log
Level: 16
Difficulty: **

You will be asked for the hash of most recent commit.  You will need to investigate the logs of the repository for this.

Answer:

```
$ git log
输入recent commit 的hash(afc31d4ce6322353cc6bd32e9e661dd8d974e419)
```

---
Name: tag
Level: 17
Difficulty: **

We have a git repo and we want to tag the current commit with `new_tag`.

Answer:

```
$ git tag new_tag
```
---
Name: push_tags
Level: 18
Difficulty: **

There are tags in the repository that aren't pushed into remote repository. Push them now.

Answer:

```
$ git push --tags
```
---
Name: commit_amend
Level: 19
Difficulty: **

The `README` file has been committed, but it looks like the file `forgotten_file.rb` was missing from the commit.  Add the file and amend your previous commit to include it.

Answer:

```
$ git st
$ git ci --amend
```
---
Name: commit_in_future
Level: 20
Difficulty: **

Commit your changes with the future date (e.g. tomorrow).

Answer:

```
$ git ci --date=2016-01-04T00:00:00
```
---
Name: reset
Level: 21
Difficulty: **

There are two files to be committed.  The goal was to add each file as a separate commit, however both were added by accident.  Unstage the file `to_commit_second.rb` using the reset command (don't commit anything).

Answer:

```
$ git reset head to_commit_second.rb
```
---
Name: reset_soft
Level: 22
Difficulty: **

You committed too soon. Now you want to undo the last commit, while keeping the index.

Answer:

```
$ git reset --soft HEAD^
```
---
Name: checkout_file
Level: 23
Difficulty: ***

A file has been modified, but you don't want to keep the modification.  Checkout the `config.rb` file from the last commit.

Answer:

```
$ git co -- config.rb
```
---
Name: remote
Level: 24
Difficulty: **

This project has a remote repository.  Identify it.

Answer:

```
$ git remote -v
输入remote_repo name(my_remote_repo)
```

---
Name: remote_url
Level: 25
Difficulty: **

The remote repositories have a url associated to them.  Please enter the url of remote_location.

Answer:

```
$ git remote -v
输入remote_repo url(https://github.com/githug/not_a_repo)
```

---
Name: pull
Level: 26
Difficulty: **

You need to pull changes from your origin repository.

Answer:

```
$ git pull origin master
```
---
Name: remote_add
Level: 27
Difficulty: **

Add a remote repository called `origin` with the url https://github.com/githug/githug

Answer:

```
$ git remote add origin https://github.com/githug/githug
```
---
Name: push
Level: 28
Difficulty: ***

Your local master branch has diverged from the remote origin/master branch. Rebase your commit onto origin/master and push it to remote.

Answer:

```
$ git rebase origin/master
$ git push origin master
```
---
Name: diff
Level: 29
Difficulty: **

There have been modifications to the `app.rb` file since your last commit.  Find out which line has changed.

Answer:

```
$ git diff
输入changed line number(26)
```

---
Name: blame
Level: 30
Difficulty: **

Someone has put a password inside the file `config.rb` find out who it was.

Answer:

```
$ git blame config.rb
输入password行的最近提交人(Spider Man)
```

---
Name: branch
Level: 31
Difficulty: *

You want to work on a piece of code that has the potential to break things, create the branch test_code.

Answer:

```
$ git co -b test_code
```
---
Name: checkout
Level: 32
Difficulty: **

Create and switch to a new branch called my_branch.  You will need to create a branch like you did in the previous level.

Answer:

```
$ git co -b my_branch
```
---
Name: checkout_tag
Level: 33
Difficulty: **

You need to fix a bug in the version 1.2 of your app. Checkout the tag `v1.2`.

Answer:

```
$ git co v1.2
```
---
Name: checkout_tag_over_branch
Level: 34
Difficulty: **

You need to fix a bug in the version 1.2 of your app. Checkout the tag `v1.2` (Note: There is also a branch named `v1.2`).

Answer:

```
$ git co tags/v1.2
```
---
Name: branch_at
Level: 35
Difficulty: ***

You forgot to branch at the previous commit and made a commit on top of it. Create branch test_branch at the commit before the last.

Answer:

```
$ git br test_branch -v a2ba76984a08d706d40a4d206462140e2ec4c53b
```

---
Name: delete_branch
Level: 36
Difficulty: **

You have created too many branches for your project. There is an old branch in your repo called 'delete_me', you should delete it.

Answer:

```
$ git br -d delete_me
```
---
Name: push_branch
Level: 37
Difficulty: **

You've made some changes to a local branch and want to share it, but aren't yet ready to merge it with the 'master' branch.  Push only 'test_branch' to the remote repository

Answer:

```
$ git push origin test_branch
```
---
Name: merge
Level: 38
Difficulty: **

We have a file in the branch 'feature'; Let's merge it to the master branch.

Answer:

```
$ git merge feature
```
---
Name: fetch
Level: 39
Difficulty: **

Looks like a new branch was pushed into our remote repository. Get the changes without merging them with the local repository

Answer:

```
$ git fetch origin
```
---
Name: rebase
Level: 40
Difficulty: **

We are using a git rebase workflow and the feature branch is ready to go into master. Let's rebase the feature branch onto our master branch.

Answer:

```
$ git co feature
$ git rebase master
```
---
Name: repack
Level: 41
Difficulty: **

Optimise how your repository is packaged ensuring that redundant packs are removed.

Answer:

```
$ git repack -d
```
---
Name: cherry-pick
Level: 42
Difficulty: ***

Your new feature isn't worth the time and you're going to delete it. But it has one commit that fills in `README` file, and you want this commit to be on the master as well.

Answer:

```
$ git co new-master
$ git blame README.md
$ git cherry-pick ca32a6da
```
---
Name: grep
Level: 43
Difficulty: **

Your project's deadline approaches, you should evaluate how many TODOs are left in your code

Answer:

```
$ git grep TODO
```
---
Name: rename_commit
Level: 44
Difficulty: ***

Correct the typo in the message of your first (non-root) commit.

Answer:

```
$ git rebase -i HEAD~2
```
将错误的commit的pick修改为edit,然后
```
$ git ci --amend  修改commit信息
$ git rebase --continue
```
---
Name: squash
Level: 45
Difficulty: ****

You have committed several times but would like all those changes to be one commit.

Answer:

```
$ git rebase 331299b796c6e873dbffd08f2ac111454fa75b8a -i(最先的commit hash)
修改pick 为squash
```
---
Name: merge_squash
Level: 46
Difficulty: ***

Merge all commits from the long-feature-branch as a single commit.

Answer:

```
$ git merge --squash long-feature-branch
$ git ci -a -m "merge branch"
```
---
Name: reorder
Level: 47
Difficulty: ****

You have committed several times but in the wrong order. Please reorder your commits.

Answer:

```
$ git rebase -i 89b082ce1e8c445b5ad336d51fe4491756949053
修改pick的顺序
```
---
Name: bisect
Level: 48
Difficulty: ***

A bug was introduced somewhere along the way.  You know that running `ruby prog.rb 5` should output 15.  You can also run `make test`.  What are the first 7 chars of the hash of the commit that introduced the bug.

Answer:
二分查找,可能不少人看不懂这关。。其实就是让你用git来找BUG

```
$ git log --reverse 找到第一次提交hash
$ git bisect start
$ git bisect good f608824888b83bbedc1f658be7496ffea467a8fb
$ git bisect bad
$ git bisect run make test
日志里的18ed2ac1522a014412d4303ce7c8db39becab076 is the first bad commit回答此hash
```

---
Name: stage_lines
Level: 49
Difficulty: ****

You've made changes within a single file that belong to two different features, but neither of the changes are yet staged. Stage only the changes belonging to the first feature.

Answer:

```
$ git add -p 
stage this hunk 选择e编辑，删除second feature stage
```
---
Name: find_old_branch
Level: 50
Difficulty: ****

You have been working on a branch but got distracted by a major issue and forgot the name of it. Switch back to that branch.

Answer:

```
$ git reflog
可见操作历史中倒数第二条：6876e5b HEAD@{1}: checkout: moving from solve_world_hunger to kill_the_batman
说明原来是在 solve_world_hunger分支
$ git co solve_world_hunger
```
---
Name: revert
Level: 51
Difficulty: ****

You have committed several times but want to undo the middle commit.
All commits have been pushed, so you can't change existing history.

Answer:

```
$ git revert 8be480b0e0332dedf8dd9a2b4ee3c4d061a2c79d
```
---
Name: restore
Level: 52
Difficulty: ****

You decided to delete your latest commit by running `git reset --hard HEAD^`.  (Not a smart thing to do.)  You then change your mind, and want that commit back.  Restore the deleted commit.

Answer:

```
$ git reflog
$ git co 5fefc92
```
---
Name: conflict
Level: 53
Difficulty: ****

You need to merge mybranch into the current branch (master). But there may be some incorrect changes in mybranch which may cause conflicts. Solve any merge-conflicts you come across and finish the merge.

Answer:

```
$ git merge mybranch
$ vim poem.txt
$ git add .
$ git ci -m "merge"
```
---
Name: submodule
Level: 54
Difficulty: **

You want to include the files from the following repo: `https://github.com/jackmaney/githug-include-me` into a the folder `./githug-include-me`. Do this without cloning the repo or copying the files from the repo into this repo.

Answer:

```
$ git submodule add https://github.com/jackmaney/githug-include-me githug-include-me
```
---
Name: contribute
Level: 55
Difficulty: ***

This is the final level, the goal is to contribute to this repository by making a pull request on Github.  Please note that this level is designed to encourage you to add a valid contribution to Githug, not testing your ability to create a pull request.  Contributions that are likely to be accepted are levels, bug fixes and improved documentation.




