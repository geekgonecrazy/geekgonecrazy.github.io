---
layout: post
title:  "Fedora Setup"
date:   2017-12-05
comments: true
publish: false
tags: [osx]
---

Install nvidia drivers (oss drivers on this card fail miserably)

Install gnome tweak tool
`sudo dnf install gnome-tweak-tool`

Enable global dark theme

Enable minamize and maximize buttons

Enable workspaces span displays

Set screenshot area to clipboard to ctrl+shift+4

Install tilix

`sudo dnf install tilix`

`sudo dnf install tmux`

```
sudo dnf groupinstall "Development Tools"
sudo dnf groupinstall "C Development Tools and Libraries"
```

sudo dnf install zsh

Install oh my zsh
`sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"`

restore dotfiles

Install Docker
`sudo dnf -y install dnf-plugins-core`
```
sudo dnf config-manager \
    --add-repo \
    https://download.docker.com/linux/fedora/docker-ce.repo
```

`sudo dnf install docker-ce`

At time of writing have to actually hop in: `/etc/yum.repos.d/docker-ci.repo` and set version to 26 since docker doesn't have fedora 27
```
baseurl=https://download.docker.com/linux/fedora/26/$basearch/stable
```

add user to group
`sudo usermod -G docker aaron`

too lazy to restart to make sure things are working... `newgrp docker`

Install golang
`sudo dnf install golang-bin`

`sudo dnf install nodejs`

Drop in .gitconfig

Add gpg key for signing
https://www.debuntu.org/how-to-importexport-gpg-key-pair/
Install Rocket.Chat

Install Google Chrome

Install Google Play Music Desktop Player
`flatpak install --from https://flathub.org/repo/appstream/com.googleplaymusicdesktopplayer.GPMDP.flatpakref`

Install Visual Studio Code
`flatpak install --from https://flathub.org/repo/appstream/com.visualstudio.code.flatpakref`

Install Enpass


Install Corebird
`flatpak install --from https://flathub.org/repo/appstream/org.baedert.corebird.flatpakref`

GNOME Extensions
`sudo dnf install chrome-gnome-shell`

Dash to panel - https://extensions.gnome.org/extension/1160/dash-to-panel/
Top Icons Plus - https://extensions.gnome.org/extension/1031/topicons/


Download minio hook up to s3 buckets (digitalocean)
https://minio.io/downloads.html#download-client


yubikey u2f setup
https://www.yubico.com/support/knowledge-base/categories/articles/can-set-linux-system-use-u2f/


Install keybase
`sudo yum install https://prerelease.keybase.io/keybase_amd64.rpm`

Install kubectl
https://kubernetes.io/docs/tasks/tools/install-kubectl/