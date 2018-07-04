---
layout: post
title:  "Automated Apple MacBook setup"
date:   2018-03-20
---

My ideal setup for my MacBook Pro - single script to run. And the script should be maintained as in a repo (getting there). Shouldn't be any different for any other computer running recent versions of MacOS X.

# My current script
Will require to enter your password during the process.
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew tap caskroom/cask
brew tap caskroom/version

xcode-select --install
brew install wget openssl
brew install maven gradle httpie jq node git
brew install mysql

brew cask install java8
brew cask install sourcetree
brew cask install iterm2
brew cask install shiftit
brew cask install slack
brew cask install atom #and/or Sublime Text, Textmate
brew cask install trolcommander
brew cask install google-chrome
brew cask install viber
brew cask install docker
open /Applications/Docker.app #need to start it to set it up

#Kubernetes, Google Cloud Platform (GCP), Pivotal Cloud Foundry (PCF), BOSH, Terraform
brew tap cloudfoundry/tap
brew tap pivotal/tap
brew install kubectl cf-cli bosh-cli terraform
brew cask install google-cloud-sdk
```
I also manually install MenuMeters adaptation for modern MacOS X from <https://member.ipmu.jp/yuji.tachikawa/MenuMetersElCapitan/>

## Other useful tools

### Fonts & terminal enhancements. Highly recommended
```
brew tap caskroom/fonts
brew cask install font-hack-nerd-font
#set `Hack Nerd Font` in your terminal

#Bash-it!
git clone --depth=1 https://github.com/Bash-it/bash-it.git ~/.bash_it
~/.bash_it/install.sh
#edit ~/.bash_profile to change theme to `export BASH_IT_THEME='powerline'`
#https://github.com/Bash-it/bash-it/wiki/Themes#powerline-naked
nano ~/.bash_profile
reload
#or cd into a git repo folder and run the following to try all themes
BASH_PREVIEW=true reload

#Fish shell - just try it out!!
brew install fish
curl -L github.com/oh-my-fish/oh-my-fish/raw/master/bin/install | fish
omf help
omf install bobthefish
omf reload
```

### VLC and file associations
```
brew cask install vlc
brew install duti
duti -s org.videolan.vlc .mkv all
duti -s org.videolan.vlc .avi
duti -s org.videolan.vlc .mp4
```

### Transmission and Transmission remote GUI
```
brew tap caskroom/cask
brew cask install transmission-remote-gui
```



https://www.android.com/filetransfer/




### EncFS cross-platform and cloud-friendly folder encryption
```
brew cask install osxfuse
brew install encfs
```

### SSH FS to be able to mount filesystem over SSH
```
brew cask install osxfuse
brew install sshfs
#sshfs root@10.0.1.1:/var/www ~/Documents/mount
```

### Google Cloud SDK and command line
```
brew cask install google-cloud-sdk
brew install kubectl
```

## And/or use Pivotal goodies
```
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
