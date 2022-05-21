---
layout: post
title: 맥 개발환경 설정
subtitle: 새로 받은 맥북을 이렇게 셋팅했습니다.
date: 2022-05-22 00:07:00 +0900
---

## 기본설정
* [한/영키 설정](https://gist.github.com/onkwon/3d101c15e09542e2b49d13c97ee2885b)
* [HomeBrew](https://brew.sh/)
* 폰트
    * [MesloLGMDZ Nerd Font Mono M-DZ Regular](https://github.com/ryanoasis/nerd-fonts/blob/master/patched-fonts/Meslo/M-DZ/Regular/complete/Meslo%20LG%20M%20DZ%20Regular%20Nerd%20Font%20Complete%20Mono.ttf)
* Siri 프라이버시 설정
* spotlight 검색 범위 설정

## Xcode

## git

```shell=
$ brew install gpg
$ gpg --list-secret-keys k@mononn.com
$ gpg --export-secret-keys k@mononn.com > k-gpg.key
$ gpg --import k-gpg.key
$ ssh-keygen -t ed25519 -C "k@mononn.com"
```

* gpg 사이닝 실패할 경우
    * `brew install pinentry-mac`

## nvim

```shell=
$ brew install nvim
$ sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs \
       https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'
```

* jedi python 버전 에러 발생할 경우
    * `pip install pynvim`

## Iterm2
* [https://iterm2.com/](https://iterm2.com/)

## zsh

```shell=
$ brew install zsh
```

### oh-my-zsh plugin
* [https://ohmyz.sh](https://ohmyz.sh)

## dotfiles

```shell=
$ cat $(HOME)/.zshrc
[...]
alias dotfiles='/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'
[...]

$ echo ".dotfiles" >> .gitignore
$ git clone --bare git@github.com:onkwon/dotfiles.git $HOME/.dotfiles
$ dotfiles checkout
$ dotfiles config --local status.showUntrackedFiles no
```

## tmux

```shell=
$ brew install tmux
```

## 터미널 꾸미기

```shell=
$ git clone --depth=1 https://github.com/romkatv/powerlevel10k.git .oh-my-zsh/themes/powerlevel10k
```

### plugins

```shell=
$ git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
$ git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
$ git clone https://github.com/zpm-zsh/tmux ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/tmux
$ git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
$ git clone https://github.com/zdharma/fast-syntax-highlighting.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/fast-syntax-highlighting
$ git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:=~/.oh-my-zsh/custom}/plugins/zsh-completions

$ brew install fzf
$ $(brew --prefix)/opt/fzf/install
```

* tmux plugin 을 설치하기 위해 `<prefix> + I`

## Python

```shell=
$ brew install pyenv
$ pyenv install 3.9.6
$ pyenv global 3.9.6
$ $(pyenv which python3) -m pip install virtualenvwrapper 
```

## JAVA
* [https://www.azul.com/downloads/?package=jdk#download-openjdk](https://www.azul.com/downloads/?package=jdk#download-openjdk)

## J-Link
* [https://www.segger.com/downloads/jlink/](https://www.segger.com/downloads/jlink/)

## 기타

```shell=
$ brew install coreutils

$ brew install ccls
$ brew install ctags cscope
$ brew install ripgrep
$ brew install tree
$ brew install gsed

$ brew install graphviz
$ brew install plantuml

$ brew install automake
$ brew install lcov
$ brew install sphinx-doc
$ brew install doxygen

$ brew install openocd
```

### display
* [https://github.com/usr-sse2/RDM](https://github.com/usr-sse2/RDM)
* [https://codeclou.github.io/Display-Override-PropertyList-File-Parser-and-Generator-with-HiDPI-Support-For-Scaled-Resolutions/](https://codeclou.github.io/Display-Override-PropertyList-File-Parser-and-Generator-with-HiDPI-Support-For-Scaled-Resolutions/)

## 참고자료
* 폰트
    * [https://johngrib.github.io/wiki/coding-font/](https://johngrib.github.io/wiki/coding-font/)
* dotfiles
    * [https://www.atlassian.com/git/tutorials/dotfiles](https://www.atlassian.com/git/tutorials/dotfiles)
