---
title: "IdeaVim 自用配置分享"
date: 2022-01-01T10:22:10+08:00
draft: false
tags: ["IdeaVim", "VIM", "配置"]
---

<!--more-->

```
let mapleader = " "

set clipboard=unnamedplus,unnamed
set hlsearch
set incsearch
set ignorecase
set smartcase
set showmode
set number
set scrolloff=3
set history=100000
set multiple-cursors
set commentary
set easymotion
set exchange
set surround
set ideajoin
set clipboard+=ideaput
set ideamarks
set keep-english-in-normal-and-restore-in-insert
set relativenumber
set highlightedyank
set NERDTree
set showmode
set so=5
set incsearch
set nu
set idearefactormode=keep
set ideavimsupport

let g:EasyMotion_override_acejump = 0
let g:argtextobj_pairs="[:],(:),<:>"

map <leader>a :action $SelectAll<CR>
map <leader>x :action $Cut<CR>
map <leader>c :action $Copy<CR>
map <leader>z :action $Undo<CR>
map <leader>v :action $Paste<CR>

nnoremap <esc><esc> :noh<return>
nnoremap gd :action GotoDeclaration
nnoremap ,e :action SearchEverywhere<CR>
nnoremap ,g :action FindInPath<CR>
nnoremap ,s :action FileStructurePopup<CR>

nnoremap gd :action GotoDeclaration<CR>
nnoremap gs :action GotoSuperMethod<CR>
nnoremap gi :action GotoImplementation<CR>
nnoremap gb :action JumpToLastChange<CR>

nnoremap U :action FindUsages<CR>
nnoremap R :action RenameElement<CR>

nnoremap == :action ReformatCode<CR>
vnoremap == :action ReformatCode<CR>

nnoremap [] :action OptimizeImports<CR>
vnoremap [] :action OptimizeImports<CR>

nnoremap cc :action CommentByLineComment<CR>
vnoremap cc :action CommentByLineComment<CR>

nnoremap <C-CR> :action ShowIntentionActions<CR>

nnoremap ,a :action GotoAction<CR>
vnoremap ,a :action GotoAction<CR>

xnoremap p "0p
xnoremap p "_dP

" 搜索文件相关
nnoremap <leader>zc :action GotoClass<CR>
vnoremap <leader>zc :action GotoClass<CR>
nnoremap <leader>za :action GotoAction<CR>
nnoremap <leader>zh :action RecentChangedFiles<CR>
nnoremap <leader>zf :action GotoFile<CR>
nnoremap <leader>zd :action ActivateDebugToolWindow<CR>
nnoremap <leader>zr :action ActivateRunToolWindow<CR>
nnoremap <leader>zs :action ShelvedChangesToolbar<CR>
nnoremap <leader>zt :action ActivateTODOToolWindow<CR>
nnoremap <leader>zv :action ActivateVersionControlToolWindow<CR>
nnoremap <leader>zb :action ShowBookmarks<CR>
nnoremap <leader>zp :action ActivateProjectToolWindow<CR>

nnoremap <Leader>o :<C-u>action RecentProjectListGroup<CR>
noremap <Space>p :action SelectInProjectView<CR>
```
