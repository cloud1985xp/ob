---
tags:
  - vim
  - tools
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# Vim / Neovim

Created: 2023年4月29日 上午12:31

## Vundle

[https://github.com/VundleVim/Vundle.vim](https://github.com/VundleVim/Vundle.vim)

```jsx
git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

Install Plugin

```ruby
:PluginInstall
```

or

```ruby
vim +PluginInstall +qall
```

### Leader Key

[Vim 101: What is the Leader Key?](https://medium.com/usevim/vim-101-what-is-the-leader-key-f2f5c1fa610f)

default: `\`

### Direction

HJKL

←↓↑→

### Indent Lines

[Quickly indent multiple lines in vi/vim](https://blog.cadena-it.com/linux-tips-how-to/quickly-indent-multiple-lines-in-vivim/)

v

JK

>

Multiple Lines Copy / Cust

v

jk

y / x

p

### Copy Lines

[How do I copy multiple lines in VIM?](https://www.quora.com/How-do-I-copy-multiple-lines-in-VIM)

## Insert Mode

Delete Whole Line:

Ctrl-w

### Switch Window

ctrl+w h → 左 window 

ctrl+w l → 右 window

### Switch Tab

gt → next tab

gT → previous tab

[In vim, how can I quickly switch between tabs?](https://superuser.com/questions/410982/in-vim-how-can-i-quickly-switch-between-tabs)

### Insert New line, Editing

[https://vim.fandom.com/wiki/Insert_newline_without_entering_insert_mode](https://vim.fandom.com/wiki/Insert_newline_without_entering_insert_mode)

- a: Append text following the current cursor position
- A: Append text to the end of the current line
- i: Insert text at the current cursor position
- I: Insert text at the beginning of the current line
- o: Begin a new line below the cursor to insert text
- O: Begin a new line above the cursor to add text

### Deleting Backward

1. d<leftArrow> will delete current and left character
2. d$ will delete from current position to end of line
3. d^ will delete from current backward to first non-white-space character
4. d0 will delete from current backward to beginning of line
5. dw deletes current to end of current word (including trailing space)
6. db deletes current to beginning of current word

[vim deleting backward tricks](https://stackoverflow.com/questions/1373841/vim-deleting-backward-tricks)

### Resize Split

[Resize splits more quickly](https://vim.fandom.com/wiki/Resize_splits_more_quickly)

```ruby
:resize 30 
:vertical resize 80
:vertical resize +5
:vertical resize -5
C-w +/-
C-w >/<
C-w =
C-w _
C-w |
```

### Open New File in Split

:sp newfile

:vsp newfile

C-w r: Move Split to Right Position

C-w x: Move Split to Left

C-w v: New vertical split

C-w s: New (horizontal) split

C-w c: close window but keep buffer

C-w o: Close other window

C-w =: make all splits equal size

:vertical sball: vertical split all buffers

## 在 Vim 把選擇文字複製到剪貼簿

用 selection mode (v) 選擇文字

輸入 

```jsx
"*y
```

就可以到其他地方貼上了

## 在 Vim 用 fzf

先 brew install 安裝 fzf

或其他安裝方式

[fzf/README-VIM.md at master · junegunn/fzf](https://github.com/junegunn/fzf/blob/master/README-VIM.md)

```jsx
在 .vimrc 加入

set rtp+=/usr/local/opt/fzf

Plugin 'junegunn/fzf'
Plugin 'junegunn/fzf.vim'

:source ~/.vimrc
:PluginUpdate
```

就可以用各種指令了

[https://github.com/junegunn/fzf.vim](https://github.com/junegunn/fzf.vim)

Ex

:Ag [PATTERN]

可以加設定

```jsx
let g:fzf_layout = { 'window': { 'width': 0.9, 'height': 0.6 } }
let g:fzf_colors = { 'fg': ['fg', 'Normal'], 'bg': ['bg', 'Normal'], 'hl': ['fg', 'Comment'], 'fg+': ['fg', 'CursorLine', 'CursorColumn', 'Normal'], 'bg+': ['bg', 'CursorLine', 'CursorColumn'], 'hl+': ['fg', 'Statement'], 'info': ['fg', 'PreProc'], 'border': ['fg', 'Ignore'], 'prompt': ['fg', 'Conditional'] }
let g:fzf_preview_window = ['right:50%', 'ctrl-/']
```

## 更換大小1寫

Ctrl+v 選擇多行

u or gu 全變小寫

U or gU 全變大寫

## **telescope.nvim**

leader (\) + ff

### CtrlP

- Ctrl+x → Open in horizontal split
- Ctrl+v → Open in vertical split
- Ctrl+t → Open in new tab

## Plugin To Use

ctriP

[https://github.com/ctrlpvim/ctrlp.vim](https://github.com/ctrlpvim/ctrlp.vim)

Ack

[https://github.com/mileszs/ack.vim](https://github.com/mileszs/ack.vim)

[https://github.com/prabirshrestha/vim-lsp](https://github.com/prabirshrestha/vim-lsp)

NERDTree

[https://github.com/preservim/nerdtree](https://github.com/preservim/nerdtree)

vim-elixir

[https://github.com/elixir-editors/vim-elixir](https://github.com/elixir-editors/vim-elixir)

[Setup vim for elixir development](https://medium.com/@siever/setup-vim-for-elixir-development-280a01150152)