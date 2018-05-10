---
layout: post
title:  "Unpublished template"
published: false
---

alias.gst=git
alias.st=status
alias.di=diff
alias.co=checkout
alias.ci=commit
alias.br=branch
alias.sta=stash
alias.llog=log --date=local
alias.flog=log --pretty=fuller --decorate
alias.lg=log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative
alias.lol=log --graph --decorate --oneline
alias.lola=log --graph --decorate --oneline --all
alias.blog=log origin/master... --left-right
alias.ds=diff
alias.fixup=commit
alias.squash=commit
alias.unstage=reset
alias.rum=rebase master@{u}
alias.it=!git init && git commit -m "root" --allow-empty
alias.l=log --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %C(bold blue)<%an>%Creset'
alias.ll=!git l --decorate --numstat
alias.lga=!git l --graph --all
alias.lgb=!git l --graph --branches
alias.li=!git l HEAD..@{upstream}
alias.lo=!git l @{upstream}..HEAD
alias.refs=for-each-ref --sort=-committerdate --format='%(color:red bold)%(refname:short)%(color:reset) %(color:yellow)%(committerdate:relative)%(color:reset) %(color:magenta bold)%(authorname)%(color:reset) %(color:green)%(objectname:short)%(color:reset) %(contents:subject)'
alias.lbs=!git refs refs/heads
alias.rbs=!git refs refs/remotes
alias.lob=!git branch -vv | grep ': gone' | cut -d ' ' -f 3
alias.dob=!git lob | xargs git branch -D
alias.fp=!f() { git fetch upstream pull/$1/head && git checkout FETCH_HEAD; }; f


git config --global alias.st=status
git config --global alias.ci=commit
git config --global
