---
title: Oh-my-zsh
date: 2022-10-29 23:25:49
categories:
- Linux
tags:
- zsh
---

### [Oh-my-zsh](https://ohmyz.sh/)

Before installing Oh-my-zsh, make sure zsh has been install. If not, tpye the following command.

```bash
sudo apt install zsh
# change your shell from bash into zsh
sudo chsh -s /usr/bin/zsh
# locate shell
cat /etc/shells
```

Then, download and run install script.

```bash
wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh
./install.sh
```
