---
tags:
  - devops
  - tools
created: 2024-01-01
updated: 2025-01-23
status: active
---

Rename Title to Local Environment Setup

# Start
https://codecharms.me/posts/mac-setup-%E8%AE%93%E4%BD%A0%E7%9A%84%E9%96%8B%E7%99%BC%E9%80%9F%E5%BA%A6%E5%A4%A7%E8%BA%8D%E9%80%B2
https://codecharms.me/posts/%E6%AF%8F%E5%80%8B%E9%96%8B%E7%99%BC%E8%80%85%E9%83%BD%E6%87%89%E8%A9%B2%E8%A6%81%E6%9C%83%E7%94%A8%E7%9A%84%E7%B7%A8%E8%BC%AF%E5%99%A8-vim

## Tmux / Zsh / Powerlevel10K
https://ithelp.ithome.com.tw/articles/10215362
https://github.com/romkatv/powerlevel10k#installation
https://holychung.medium.com/%E5%88%86%E4%BA%AB-oh-my-zsh-powerlevel10k-%E5%BF%AB%E9%80%9F%E6%89%93%E9%80%A0%E5%A5%BD%E7%9C%8B%E5%A5%BD%E7%94%A8%E7%9A%84-command-line-%E7%92%B0%E5%A2%83-f66846117921


## iTerm2 / Font / Theme / Zsh

https://iterm2colorschemes.com/
https://medium.com/nitas-learning-journey/mac%E7%B5%82%E7%AB%AF%E6%A9%9F-terminal-%E8%A8%AD%E5%AE%9A-iterm2-ba63efd0df6a

## For Kubernetes

https://github.com/jonmosco/kube-ps1

https://golangexample.com/kubecolor-colorize-your-kubectl-output/
https://github.com/hidetatz/kubecolor

## Cheatsheet

nvim .config/nvim/init.vim
```
set runtimepath^=~/.vim runtimepath+=~/.vim/after
let &packpath = &runtimepath
source ~/.vimrc
```

### Install on Amazon Linux

https://gist.github.com/kawaz/393c7f62fe6e857cc3d9

```
#!/usr/bin/env bash
sudo yum groups install -y Development\ tools
sudo yum install -y cmake
sudo yum install -y python34-{devel,pip}
sudo pip-3.4 install neovim --upgrade
(
cd "$(mktemp -d)"
git clone https://github.com/neovim/neovim.git
cd neovim
make CMAKE_BUILD_TYPE=Release
sudo make install
)
```

copy content of .vimrc
neovim +PluginInstall +qall


### Install Silver Search

https://www.linode.com/docs/guides/silver-searcher-on-linux/

# Vundle

https://github.com/VundleVim/Vundle.vim

# NeoVim
Installation
https://github.com/neovim/neovim/wiki/Installing-Neovim
https://nvchad.netlify.app/getting-started/setup

Compare
https://www.linuxfordevices.com/tutorials/linux/vim-vs-neovim

Migrated from vim
https://blog.duyet.net/2021/06/neovim.html
https://otavio.dev/2018/09/30/migrating-from-vim-to-neovim/

# Operation
https://itsfoss.com/how-to-exit-vim/

# Plugin
https://github.com/folke/which-key.nvim
https://github.com/tpope/vim-fugitive
https://github.com/nvim-treesitter/nvim-treesitter
https://github.com/nvim-telescope/telescope.nvim
https://github.com/prabirshrestha/vim-lsp

https://github.com/amix/vimrc
https://github.com/vgod/vimrc/blob/master/vimrc

fzf
https://github.com/junegunn/fzf.vim
https://sourabhbajaj.com/mac-setup/iTerm/fzf.html

LSP + Treesitter + Fuzzy Finder
https://blog.inkdrop.app/how-to-set-up-neovim-0-5-modern-plugins-lsp-treesitter-etc-542c3d9c9887
https://blog.backtick.consulting/neovims-built-in-lsp-with-ruby-and-rails/

# Split
https://technotales.wordpress.com/2010/04/29/vim-splits-a-guide-to-doing-exactly-what-you-want/
https://www.tecmint.com/split-vim-screen/

# Session
:mksession

https://blog.wildsky.cc/posts/vim-save-session
https://github.com/rmagatti/auto-session
https://stackoverflow.com/questions/5142099/how-to-auto-save-vim-session-on-quit-and-auto-reload-on-start-including-split-wi
https://alldrops.info/posts/vim-drops/2020-11-15_vim-sessions/
https://dockyard.com/blog/2018/06/01/simple-vim-session-management-part-1

# Buffer , Window
https://stackoverflow.com/questions/22087724/vim-close-window-without-closing-buffer

# Shell
https://stackoverflow.com/questions/1879219/how-to-temporarily-exit-vim-and-go-back


# Vimdiff
Use vimdiff as merge tool in git
https://blog.rex-tsou.com/2018/11/%E7%94%A8-vimdiff-%E8%A7%A3-git-%E8%A1%9D%E7%AA%81/


# Setup Github CLI?
https://cli.github.com/
