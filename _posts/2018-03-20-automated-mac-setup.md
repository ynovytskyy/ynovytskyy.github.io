---
layout: post
title:  "Automated Apple MacBook setup"
date:   2018-03-20
---

My ideal setup for my MacBook Pro - single script to run. And the script should be maintained as in a repo (getting there). Shouldn't be any different for any other computer running recent versions of MacOS X.

# My current script
Will require to enter your password during the process.
```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew tap caskroom/cask
brew tap caskroom/version
brew tap jcgay/jcgay
brew tap cloudfoundry/tap
brew tap pivotal/tap

brew install wget jq openssl git maven-deluxe bosh-cli terraform cf-cli

brew cask install java8
brew cask install iterm2
brew cask install shiftit
brew cask install slack
brew cask install atom
brew cask install trolcommander
brew cask install google-chrome
brew cask install viber
```

## Install latest Docker CE for Mac
Semi-manual - didn't find cask for this edition:
```bash
#get and install latest Docker CE for Mac
wget https://download.docker.com/mac/stable/Docker.dmg
open Docker.dmg
#install manually, then test in terminal
docker run hello-world
```

## And/or use Pivotal goodies
```bash
mkdir -p ~/work/tools
cd ~/work/tools
git clone https://github.com/pivotal/workstation-setup.git
cd workstation-setup
./setup.sh java8 golang
```

# List of installed packages
```bash
○ → brew tap
caskroom/cask
caskroom/versions
cloudfoundry/tap
git-duet/tap
homebrew/core
homebrew/services
jcgay/jcgay
pivotal/tap
seattle-beach/tap

○ → brew list
autoconf		gradle			python
bbl			grc			rbenv
bosh-cli		jq			readline
cf-cli			libidn2			ruby-build
coreutils		libssh2			springboot
direnv			libunistring		sqlite
gdbm			maven-deluxe		terraform
gettext			oniguruma		the_silver_searcher
git			openssl			watch
git-duet		pcre			wget
git-pair		pkg-config		xz
git-together		pstree

○ → brew cask list
atom           github         macvim         skype          textmate
dash           intellij-idea  postman        slack          viber
firefox        java8          rowanj-gitx    sourcetree     virtualbox
flycut         macdown        shiftit        sublime-text   zoomus
```

# Links
- Homebrew - The missing package manager for macOS - <https://brew.sh/>
  - <https://brew.sh/analytics/install-on-request/>
- Homebrew Cask - “To install, drag this icon…” no more! - <https://caskroom.github.io/>
- Bash-it - <https://github.com/Bash-it/bash-it>
- Another customization of terminal with `zsh` - <https://medium.com/@elviocavalcante/5-steps-to-improve-your-terminal-appearance-on-mac-osx-f58b20058c84>

## Some other usefull apps to mention:
- <https://bjango.com/mac/istatmenus/>
- <https://seense.com/menubarstats/> - ? 5 USD / 6.66 CAD
- <https://member.ipmu.jp/yuji.tachikawa/MenuMetersElCapitan/>

# Additional notes
Also found the following script to install an app from a `*.dmg` in a a scripted way. Haven't used it yet though.
```bash
temp=$TMPDIR$(uuidgen)
mkdir -p $temp/mount
curl https://dl.google.com/chrome/mac/stable/GGRO/googlechrome.dmg > $temp/1.dmg
yes | hdiutil attach -noverify -nobrowse -mountpoint $temp/mount $temp/1.dmg
cp -r $temp/mount/*.app /Applications
hdiutil detach $temp/mount
rm -r $temp
```
