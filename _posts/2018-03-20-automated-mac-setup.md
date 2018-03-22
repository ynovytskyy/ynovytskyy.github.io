---
layout: post
title:  "Automated Apple MacBook setup"
date:   2018-03-20
---

My ideal setup for my MacBook Pro - single script to run. And the script should be maintained as in a repo (getting there). Shouldn't be any different for any other computer running recent versions of MacOS X.

# Use a script maintained in GitHub
*TODO*

# My current script
Will require to enter your password during the process.
```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew tap caskroom/cask
brew install wget
brew cask install iterm2
brew cask install viber
brew cask install shiftit
brew cask install slack
brew cask install atom
brew cask install google-chrome

brew tap caskroom/version
brew tap install java8
```

To install latest Docker CE for Mac (semi-manual):
```bash
#get and install latest Docker CE for Mac
wget https://download.docker.com/mac/stable/Docker.dmg
open Docker.dmg
#install manually, then test in terminal
docker run hello-world
```

And/or use Pivotal goodies:
```bash
mkdir -p ~/work/tools
cd ~/work/tools
git clone https://github.com/pivotal/workstation-setup.git
cd workstation-setup
./setup.sh java8 golang
```

# Links
- Homebrew - The missing package manager for macOS - <https://brew.sh/>
  - <https://brew.sh/analytics/install-on-request/>
- Homebrew Cask - “To install, drag this icon…” no more! - <https://caskroom.github.io/>
-

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
