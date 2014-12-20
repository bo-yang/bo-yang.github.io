---
layout: post
title: Building Remote+Local *nix Develop Environment(II)
categories: 
- Unix/Linux
- Notes
tags:
- Unix/Linux
- Notes
status: publish
type: post
published: true
author: Bo Yang
---

_This is the second article(collection) on how to build a *nix development environment by integrating remote servers and local Linux/Mac clients. For the previous article on this topic, please refer to [Building Remote+Local *nix Develop Environment](http://www.bo-yang.net/2014/10/21/remote-local-linux-develop-env/)._

### 1. Vim Tips & Plugins

#### 1.1 Highlight All Instances of Word Under Cursor

Add following line into your `$HOME/.vimrc`

    autocmd CursorMoved * exe printf('match IncSearch /\V\<%s\>/', escape(expand('<cword>'), '/\'))

Or use a more complicated one in the `.vimrc`:

    " Highlight all instances of word under cursor, when idle.
    " Useful when studying strange source code.
    " Type z/ to toggle highlighting on/off.
    
    nnoremap z/ :if AutoHighlightToggle()<Bar>set hls<Bar>endif<CR>
    function! AutoHighlightToggle()
      let @/ = ''
      if exists('#auto_highlight')
        au! auto_highlight
        augroup! auto_highlight
        setl updatetime=4000
        echo 'Highlight current word: off'
        return 0
      else
        augroup auto_highlight
          au!
          au CursorHold * let @/ = '\V\<'.escape(expand('<cword>'), '\').'\>'
        augroup end
        setl updatetime=500
        echo 'Highlight current word: ON'
        return 1
      endif
    endfunction

#### 1.2 Automatically Load Ctags

If you have generated ctags file, then you can automatically load it by:

- export `CTAGS_TAG` in `$HOME/.bashrc` by `export CTAGS_TAG=/path/to/your/tags`.

- add following lines into your `$HOME/.vimrc`

```Shell
    if filereadable($CTAGS_TAG)
	    set tags=$CTAGS_TAG
    endif
```

#### 1.3 Most Recently Used(MRU) Files

If you want to access the most recently used files in Vim, you need plugin [MRU](http://www.vim.org/scripts/script.php?script_id=521). The `:MRU` command will show you all the recently used files, and you can choose a file and press `<Enter>` to open it in current window. In addition

- To open a file from the MRU window in a new tab, press the `t` key. 
- You can open multiple files from the MRU window by specifying a count before pressing `<Enter>` or `v` or `o` or `t`. 
- You can close the MRU window by pressing the `q` key or the `<Esc>` key or using one of the Vim window commands. 
- You can specify a pattern to the `:MRU` command, such as `:MRU <pattern>`. 

#### 1.4 [Pathogen](https://github.com/tpope/vim-pathogen)

Vim `runtimepath` manager, widely used by many plugins. Adding `call pathogen#infect()` to your `.vimrc`, then any plugins you wish to install can be extracted to a subdirectory under `~/.vim/bundle`. And they will be added to the `runtimepath`.

#### 1.5 [NERDTree](https://github.com/scrooloose/nerdtree) & [NERDTree Tabs](https://github.com/jistr/vim-nerdtree-tabs)

The NERD tree allows you to explore your filesystem and to open files and directories, and NERDTree Tabs can make NERDTree available for all Vim tabs(sometimes it is useful). After installing the these plugins, you can add the following lines to `.vimrc`.

    " Nerd Tree
    " let g:NERDTreeDirArrows=0 " Do not use new arrows for directories
    map <C-n> :NERDTreeToggle<CR>
    let g:nerdtree_tabs_open_on_gui_startup=0 " no nerdtree_tabs by default

For some Linux distributions, the NERDTree could not show arrows for directories, then you need to uncomment the line `let g:NERDTreeDirArrows=0` in your `.vimrc`.

#### 1.6 [Supertab](https://github.com/ervandew/supertab)

Supertab is a vim plugin which allows you to use `<Tab>` for all your insert completion needs (:help ins-completion).

#### 1.7 [CtrlP](http://kien.github.io/ctrlp.vim/) & [Command-T](https://github.com/wincent/Command-T)

These two plugins are used for searching/opening files(even not in ctags) in Vim. CtrlP is written in pure Vimscript, so it is very slow. Although Command-T is faster, it relies on Ruby, which makes it difficult to install. Actually, I rarely use them in daily work.

My `vimrc` can be found at [https://github.com/bo-yang/misc/blob/master/vimrc](https://github.com/bo-yang/misc/blob/master/vimrc).

### 2. [Cscope](http://cscope.sourceforge.net/)

Cscope is a tool for browsing source code. You can either run cscope standalone or [use it with Vim](http://cscope.sourceforge.net/cscope_vim_tutorial.html). No matter in which way, you need to generate cscope database first. And the cscope DB depends on the source files you specified. General steps of using cscope are:

    find /my/project/dir -name '*.c' -o -name '*.h' > /foo/cscope.files
    cd /foo
    cscope -b
    CSCOPE_DB=/foo/cscope.out; export CSCOPE_DB

In Vim, you can load cscope DB by command `:cs add <path_to_cscope_db>`. For more cscope operations in Vim, please run command `:cs help`. To automatically load cscope into Vim, you can export `CSCOPE_DB` in `$HOME/.bashrc`, such as

    export CSCOPE_DB=/path/to/cscope.out

Then the `CSCOPE_DB` will be automatically loaded every time you run Vim.

To save the effort of building cscope DB, I wrote a cross-platform(Linux & Mac OS X) wrapper script, which can be found in my [GitHub channel](https://github.com/bo-yang/misc/blob/master/gen_cscope_ctags).

### 3. sshfs Wrapper

Since `sshfs` command requires too much parameters, and things will be worse when the network is not stable. Following script will ease your pain. 

    #!/bin/sh
    
    USER=<your_name>
    SERVER=<your_server>
    remote_dir=/nobackup/$USER
    local_dir=$HOME/Documents/VMs
    
    if [ ! -d ${local_dir} -o ! -s ${local_dir} ]
    then
    	sudo umount -f $local_dir
    fi
    
    cd ${local_dir}
    sshfs $USER@${SERVER}:${remote_dir} ${local_dir}

