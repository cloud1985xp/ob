---
tags:
  - tools
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# Environment Setup

Created: 2022年4月15日 下午3:03

# New Added by Brew Install

```jsx
brew install tree
brew install fastfetch
# brew install koekeishiya/formulae/skhd
brew install --cask nikitabobko/tap/aerospace

brew install font-hack-nerd-font
brew install font-sf-pro
brew install --cask sf-symbols

brew install lazygit

# file browser
brew install yazi ffmpeg sevenzip jq poppler fd ripgrep fzf zoxide resvg imagemagick font-symbols-only-nerd-font

# replacement of "cd"
brew install zoxide

# https://github.com/sharkdp/bat
brew install bat

# install sketchybar

# need to upgrade mac OS orz
brew tap FelixKratz/formulae
brew install borders

brew install gh

install:
claude-code
```

### MCP

[Setting Up MCP Servers in Claude Code: A Tech Ritual for the Quietly Desperate](https://www.reddit.com/r/ClaudeAI/comments/1jf4hnt/setting_up_mcp_servers_in_claude_code_a_tech/)

```bash
claude mcp add-from-claude-desktop

claude mcp add --scope user --transport sse docfork https://mcp.docfork.com/ssej
claude mcp add --scope user context7 -- npx @upstash/context7-mcp@latest
claude mcp add --scope user sequential-thinking -- docker run -i --rm mcp/sequentialthinking
claude mcp add --scope user puppeteer npx @modelcontextprotocol/server-puppeteer
claude mcp add --scope user brave-search -- env BRAVE_API_KEY=$BRAVE_API_KEY npx -y @modelcontextprotocol/server-brave-search

// by project
claude mcp add serena -- uvx --from git+https://github.com/oraios/serena serena-mcp-server --context ide-assistant --project $(pwd)

```

## iTerm2

- Settings → Profile →
    - Keys →
        - General → Left&Right Option → Esc+
        - Kay Mappings → Presets → [Terminal.app](http://Terminal.app) Compatibility
    - Terminal → Unlimited scrollback

## Homebrew

[Homebrew](https://brew.sh/index_zh-tw)

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## tmux

```bash
brew install tmux
```

### Tmux Plugins

```
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

- tmux-resurrect
- vim-tmux-navigator


TODO: tmuxifier, tmux-xpane

[https://github.com/jimeh/tmuxifier](https://github.com/jimeh/tmuxifier)

[https://github.com/greymd/tmux-xpanes](https://github.com/greymd/tmux-xpanes)

## zsh & oh-my-zsh & powerlevel10k

- New mac default to zsh

Install oh-my-zsh

[oh my zsh](https://ohmyz.sh/#install)

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

install powerlevel10k

[GitHub - romkatv/powerlevel10k: A Zsh theme](https://github.com/romkatv/powerlevel10k#oh-my-zsh)

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k

Set ZSH_THEME="powerlevel10k/powerlevel10k" in ~/.zshrc
```

set as theme

```bash
vim ~/.zshrc
```

Install zsh-autosuggestions

[zsh-autosuggestions/INSTALL.md at master · zsh-users/zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions/blob/master/INSTALL.md#oh-my-zsh)

Set hotkey for autosuggest

in .zshrc

```jsx
bindkey '^k' autosuggest-accept
```

Install zsh-completions

[https://github.com/zsh-users/zsh-completions](https://github.com/zsh-users/zsh-completions)

Install zsh-syntax-highlight

[zsh-syntax-highlighting/INSTALL.md at master · zsh-users/zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting/blob/master/INSTALL.md)

[TBD] Other ZSH plugin

- zsh-gcloud-prompt

## Install Packages

```bash
brew install the_silver_searcher
brew install ripgrep
brew install fzf
brew install jq
brew install mysql-client
brew install redis
brew services start redis
brew install imagemagick
brew install pkg-config
brew install watch
brew install libpq
```

If only install mysql-client (instead of mysql), when ruby bundler installing mysql2, it could be failed and needs to use following command:

```bash
gem install mysql2 -v '0.5.4' -- \
--with-mysql-lib=$(brew --prefix mysql-client)/lib \
--with-mysql-dir=$(brew --prefix mysql-client) \
--with-mysql-config=$(brew --prefix mysql-client)/bin/mysql_config \
--with-mysql-include=$(brew --prefix mysql-client)/include
```

## Neovim

[Installing Neovim](https://github.com/neovim/neovim/wiki/Installing-Neovim#macos--os-x)

```jsx
brew install neovim
```

### 2026/05/10 (Mac Studio)

```
系統套件(brew)

  brew install neovim          # 需要 0.12.0+(新版 nvim-treesitter 強制要求)
  brew install tree-sitter     # parser runtime
  brew install tree-sitter-cli # 新版 nvim-treesitter 編譯 parser 用 ★ 這次新加
  brew install lazygit         # lazygit.nvim 用
  brew install ripgrep fd      # telescope 搜尋更快(建議)

  tar、curl、cc(Xcode CLT)macOS 預設都有,不用額外裝。

  設定檔

  把 ~/.config/nvim/ 整份 clone/複製過去即可,不需要先手動裝 packer:plugins-setup.lua:1-12 會自動 bootstrap packer.nvim 到 ~/.local/share/nvim/site/pack/packer/start/。

  第一次啟動流程

  1. nvim — 第一次開會自動 clone packer。
  2. :PackerSync — 安裝所有 plugin。期間 nvim-treesitter 的 run = ":TSUpdate" 會被觸發,自動下載並編譯所有 parser。
  3. 退出後再開 nvim,treesitter 的 install() 對已安裝的 parser 會 no-op,啟動會很乾淨。

  其他可能要做的事(與這次修改無關但設定 nvim 時會需要)

  - GitHub Copilot::Copilot setup 登入。
  - Claude Code plugin:確保 claude CLI 已裝(你應該已經有)。
  - Elixir LSP(elixir-tools.nvim):需要本地有 Elixir/mix 環境。

  如果之後要加新語言到 treesitter,記得 treesitter.lua 的 parsers 與 FileType 兩個 list 都各加一筆。
```

### 2025/01/24

Followed this to setup nvim using lua config

[https://www.youtube.com/watch?v=vdn_pKJUda8](https://www.youtube.com/watch?v=vdn_pKJUda8&list=PLrVtg3bkJkLgXDqOycRSz6fr5sIDiXY58)

[https://github.com/josean-dev/dev-environment-files/blob/packer-nvim-setup/.config/nvim/lua/josean/plugins-setup.lua](https://github.com/josean-dev/dev-environment-files/blob/packer-nvim-setup/.config/nvim/lua/josean/plugins-setup.lua)

- [https://github.com/glepnir/nvim-lua-guide-zh](https://github.com/glepnir/nvim-lua-guide-zh)
- [https://github.com/wbthomason/packer.nvim](https://github.com/wbthomason/packer.nvim)
- [https://github.com/nvim-tree/nvim-tree.lua](https://github.com/nvim-tree/nvim-tree.lua)
- [https://docs.rockylinux.org/books/nvchad/nvchad_ui/nvimtree/](https://docs.rockylinux.org/books/nvchad/nvchad_ui/nvimtree/)
- [https://github.com/nvim-treesitter/nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter)
- [https://github.com/hrsh7th/nvim-cmp](https://github.com/hrsh7th/nvim-cmp)
- Elixir
    - [https://github.com/elixir-editors/vim-elixir](https://github.com/elixir-editors/vim-elixir)
    - [https://github.com/elixir-tools/elixir-tools.nvim](https://github.com/elixir-tools/elixir-tools.nvim)
    - [https://medium.com/@siever/setup-vim-for-elixir-development-280a01150152](https://medium.com/@siever/setup-vim-for-elixir-development-280a01150152)
    - [https://www.mitchellhanberg.com/how-to-set-up-neovim-for-elixir-development/](https://www.mitchellhanberg.com/how-to-set-up-neovim-for-elixir-development/)
    - [https://elixirforum.com/t/neovim-elixir-setup-configuration-from-scratch-guide/46310](https://elixirforum.com/t/neovim-elixir-setup-configuration-from-scratch-guide/46310)
- [https://github.com/elixir-tools/credo-language-server](https://github.com/elixir-tools/credo-language-server)
- Utils
    - [https://github.com/vim-scripts/ReplaceWithRegister](https://github.com/vim-scripts/ReplaceWithRegister)
    - [https://github.com/tpope/vim-surround](https://github.com/tpope/vim-surround)
- Elixir Language Server
    - [https://github.com/elixir-lsp/elixir-ls](https://github.com/elixir-lsp/elixir-ls)
    - [https://stackoverflow.com/questions/71284810/how-do-i-setup-elixir-ls-using-nvim-lspconfig-with-autocompletion-in-neovim](https://stackoverflow.com/questions/71284810/how-do-i-setup-elixir-ls-using-nvim-lspconfig-with-autocompletion-in-neovim)
    - [https://github.com/williamboman/nvim-lsp-installer#setup](https://github.com/williamboman/nvim-lsp-installer#setup)

===== 2025/01/24 Deprecated, Switch to use lua config ======

Migrate Settings of Vim:

in .config/nvim/init.vim

```jsx
set runtimepath^=~/.vim runtimepath+=~/.vim/after
let &packpath = &runtimepath
source ~/.vimrc
```

[Migrating from Vim to NeoVim](https://blog.duyet.net/2021/06/neovim.html)

[GitHub - VundleVim/Vundle.vim: Vundle, the plug-in manager for Vim](https://github.com/VundleVim/Vundle.vim#quick-start)

```jsx
git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

Launch `vim` and run `:PluginInstall`

To install from command line: `vim +PluginInstall +qall`

===== 2025/01/24 ======================================

Setup env dotfile, config

- vimrc
- zshrc
- .ssh/config
- .gitconfig
- .tmux.conf

## asdf

```bash
brew install asdf
asdf plugin add ruby
asdf plugin add erlang
asdf plugin add elixir
asdf plugin add nodejs
asdf install nodejs $latest_version
npm install —global yarn
```

## puma-dev

[https://github.com/puma/puma-dev](https://github.com/puma/puma-dev)

```jsx
brew install puma/puma/puma-dev

# Configure some DNS settings that have to be done as root
sudo puma-dev -setup
# Configure puma-dev to run in the background on ports 80 and 443 with the domain `.test`.
puma-dev -install
```

## Python3, pip3

New mac has installed python3, or consider to install by asdf

### Ansible

pip3 install ansible

## AWS cli, OneLogin-AWS

Install AWS cli

```bash
# v1, maybe need to install v2
pip3 install awscli —upgrade —user
```

Install onelogin-aws-cli

```bash
pip3 install onelogin-aws-cli
```

Setup onelogin-aws config

AWS ECS CLI

```jsx
brew install amazon-ecs-cli
```

SessionManagerPlugin

[Install the Session Manager plugin for the AWS CLI - AWS Systems Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html)

## Google Cloud CLI

Download sdk

[Quickstart: Install the Google Cloud CLI  |  Google Cloud CLI Documentation](https://cloud.google.com/sdk/docs/install-sdk)

```bash
google-cloud-sdk/install.sh
gcloud init

# gcloud with gke
gke-gcloud-auth-plugin --version

# if not installed yet
gcloud components install gke-gcloud-auth-plugin
```

## Terraform

```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
brew install kreuzwerker/taps/m1-terraform-provider-helper
m1-terraform-provider-helper activate

m1-terraform-provider-helper install hashicorp/template -v v2.2.0
```

## Kubernetes

### kubectl

[Install and Set Up kubectl on macOS](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/)

```bash
# arm64
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/arm64/kubectl"
```

OR

```bash
gcloud components install kubectl
```

Get credentials of cluster

```bash
gcloud container clusters get-credentials CLUSTER_NAME --region=REGION
```

### kubesec

```bash
# arm64

GO111MODULE="on" go install github.com/willyguggenheim/kubesec@latest
goto GOPATH or ~/go
chmod a+x bin/kubesec
sudo mv bin/kubesec /usr/local/bin
```

Ref:

[What does go install do?](https://stackoverflow.com/questions/24069664/what-does-go-install-do)

[Where does go get install packages?](https://stackoverflow.com/questions/50633092/where-does-go-get-install-packages)

### kube_ps1

[https://github.com/jonmosco/kube-ps1](https://github.com/jonmosco/kube-ps1)

add kube_ps1 to zsh plugins

```bash
plugins=(
  kube-ps1
)

PROMPT='$(kube_ps1)'$PROMPT
```

### kustomize

```bash
brew install kustomize
```

### kubectx (optional)

[https://github.com/ahmetb/kubectx](https://github.com/ahmetb/kubectx)

### More

[4 Kubernetes CLI Gems](https://manrai-tarun.medium.com/4-kubernetes-cli-gems-d4ae02fd2dd8)

## Rust

Cargo

[Installation - The Cargo Book](https://doc.rust-lang.org/cargo/getting-started/installation.html)

```jsx
curl https://sh.rustup.rs -sSf | sh
```

## System

### btop

[https://github.com/aristocratos/btop](https://github.com/aristocratos/btop)

```jsx
brew install btop
```

# Check List

- [x]  xbar setting
- [x]  Aily CLI
- [x]  SSH Config
    - [x]  dokku / ishin-bots
    - [x]  protectwise sensor
    - [x]  activedirectory
    - [x]  aktsk-tw-corp (hr-blog)
    - [x]  mikoto-jenkins
- [x]  SSH Keys
    - [x]  .pem
        - [x]  ishin-global-prod-2019
        - [x]  ishin-global-dev-2019
        - [x]  ishin-staging (2016) using by protectwise-sensor
        - [x]  legacy: placed to 1password
            - [x]  ishin-staging 2018
            - [x]  ishin-prod-tw-201812
            - [x]  ishin-prod-tw 2016
        - [x]  aktsk-corp
        - [x]  others
- [x]  AWS Credentials
    - [x]  Personal
    - [x]  promo: placed into 1password
    - [x]  bne
        - [x]  bank
        - [x]  audit: placed into 1password
- [x]  Security
- [x]  check connect to aktsk-corp (arising blog), db connection & backup (edited)
- [x]  gcloud cli
- [ ]  kubectl
    - [ ]  mikoto log cluster / bq sender
    - [ ]  starfish cluster
- [x]  bq CLI
- [ ]  terraform
    - [ ]  Mikoto
    - [ ]  ISHIN
    - [ ]  Starfish
- [ ]  setup postman, or alternative
- [x]  sequel pro config
- [x]  mikoto jenkins relay
- [x]  Documents
- [x]  Backup Shell History
- [ ]  Other dot config files
- [x]  dump db test data if necessary
    - [x]  ishin develop user/master, kpi_ishin_develop

### Softwares

- sequel pro
- Obsdian
- Fork
- Sequal Ace
- TablePlus
- DB Browser for SQLite

[Downloads - DB Browser for SQLite](https://sqlitebrowser.org/dl/)

### Docker Desktop

[Install Docker Desktop on Mac](https://docs.docker.com/desktop/install/mac-install/)

### GitX

[https://github.com/gitx/gitx/](https://github.com/gitx/gitx/)

### Raycast

- Replace spotlight, (disable spolight)
- Common Functions:
    - Run Application
    - sleep →
    - empty trash
    - mute
    - file >
        - filename
    - command k → other actions
- Alias settings → extensions → apps → alias
- Extensions add quick links
- quick link with query, ex CS tool
- left half / right half / max
- clipboard
- snippet
- Go to store to install extension

Ref:

[【分享】Oh My Zsh + powerlevel10k 快速打造好看好用的 command line 環境](https://holychung.medium.com/分享-oh-my-zsh-powerlevel10k-快速打造好看好用的-command-line-環境-f66846117921)

## Zsh

**AutoJump**

[https://github.com/wting/autojump](https://github.com/wting/autojump)

**AutoSuggestions**

[https://github.com/zsh-users/zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)

# Other

my local run docker services

```
docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=xxxxx -v /Users/aaron.kuo/docker/mysql8:/var/lib/mysql -d mysql:8.4 --mysql-native-password=ON
```

```jsx
docker run --name mysql57 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=xxxx -v /Users/aaron.kuo/docker/mysql:/var/lib/mysql -d mysql:57
```

Mac arm has to use unofficial image

[MySQL 5.7 Does Not Have an Official Docker Image on ARM/M1 Mac](https://betterprogramming.pub/mysql-5-7-does-not-have-an-official-docker-image-on-arm-m1-mac-e55cbe093d4c)

```jsx
docker run --name mysql57 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=xxxx -v /Users/aaron.kuo/docker/mysql:/var/lib/mysql -d biarms/mysql:5.7
```

```jsx
docker run —-name postgres -e POSTGRES_PASSWORD=postgres -p 5432:5432 -d postgres
```

### TODO

- Manipulate starfish clusters / delete stack
- Setup EC2 backup for hr-blog

## Ruby with Ubuntu

```jsx
apt install libyaml-dev
```

[打造高效的工作环境 – Shell 篇 | 酷 壳 - CoolShell](https://coolshell.cn/articles/19219.html?fbclid=IwAR3wbFGgmHloR4pEW_5ES90FwKyAIE2-t77PwZMmBE3n5W0rLu6lWtIERC8)

## Other Productivity  Software

Postgres DB GUI

[The SQL Editor and Database Manager Of Your Dreams](https://www.beekeeperstudio.io/)

[HazeOver on Setapp | Distraction Dimmer for your Mac screen](https://setapp.com/apps/hazeover)

[Automate Window Positioning With macOS and Apps - TidBITS](https://tidbits.com/2020/03/10/automate-window-positioning-with-macos-and-apps/)

[MenubarX - A powerful Mac menu bar browser](https://menubarx.app/)

[https://github.com/gao-sun/eul](https://github.com/gao-sun/eul)

[https://github.com/dwarvesf/hidden](https://github.com/dwarvesf/hidden)

[iStat Menus](https://bjango.com/mac/istatmenus/)

[Bartender 5 - Take control of your Menu bar](https://www.macbartender.com/)

## Stream Deck

[](https://docs.elgato.com/en/documentation/stream-deck/sdk/create-your-own-plugin)

[Stream Deck Icons](https://marketplace.elgato.com/stream-deck/icons)