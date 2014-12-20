---
layout: post
title: Building Remote+Local *nix Develop Environment
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

_This is the first article(collection) on how to build a *nix development environment by integrating remote servers and local Linux/Mac clients. For the following-up article on this topic, please refer to [Building Remote+Local *nix Develop Environment(II)](http://www.bo-yang.net/2014/12/19/remote-local-linux-develop-env-2/)._

### 1. `.bashrc` vs `.bash_profile`

The Bash is configured by `.bashrc` or `.bash_profile`. According to the [bash man page](http://linux.die.net/man/1/bash), `.bash_profile` is executed for login shells, while `.bashrc` is executed for interactive non-login shells.

When you login (type username and password) via console(either sitting at the machine, or remotely via ssh), `.bash_profile` is executed to configure your shell before the initial command prompt.

HOwever, if you’ve already logged into your machine and open a new terminal window(like _xterm_) inside Gnome or KDE, then `.bashrc` is executed before the window command prompt. `.bashrc` is also run when you start a new bash instance by typing `/bin/bash` in a terminal.

An exception to the terminal window guidelines is Mac OS X’s _Terminal.app_, which runs a login shell by default for each new terminal window, calling `.bash_profile` instead of `.bashrc`. 

There are two options to make one copy of bash config file work for all scenarios:

* force `.bash_profile` to be a link of `.bash_rc`, or vice versa: `ln -s ~/.bashrc ~/.bash_profile`.
* or source `.bashrc` from your `.bash_profile` file, then putting `PATH` and common settings in `.bashrc`. To do this, add the following lines to `.bash_profile`:

	if [ -f ~/.bashrc ]; then
		source ~/.bashrc
	fi

### 2. sshfs

SSHFS (SSH Filesystem) is a filesystem client to mount and interact with directories and files located on a remote server or workstation. SSHFS is based on [FUSE](http://fuse.sourceforge.net/) (the best userspace filesystem framework for *nix ). 

For Linux distributions, you can install sshfs using `sudo apt-get install sshfs` or `sudo yum install sshfs`. However, for Mac OS X, you will need to download FUSE and SSHFS from the [**osxfuse site**](http://osxfuse.github.io/).

To mount remote directories, you can use command 

	sshfs username@hostname_or_ip:/remote/dir /local/dir [ssh_options]

After mounting the remote directory, you can manipulate remote files just like local files. And you can use any local tools that’s convenient for you to read/write remote files.

To remove the mounted remote directory, please use command

	fusermount -u <LOCAL_MOUNT_POINT>

for Linux, and use command

	umount [-f] <LOCAL_MOUNT_POINT>

for Mac OS X.

Since sshfs command requires too much parameters, and things will be worse when the network is not stable. [This wrapper script](https://github.com/bo-yang/misc/blob/master/sshfs_wrapper) will ease your pain.

### 3. vim/gvim/mvim

File `$HOME/.vimrc` is used for Vim configuration, and Vim plugins are can be installed into directory `$HOME/.vim`.

#### 3.1 Word, variable, function, and line completion

_Word completion_: if words or functions are in your dictionary, or in the current file, you can save a lot of time with `<Ctrl-P>` and `<Ctrl-N>`. 

![vim completion]({{ site.url }}/assets/images/2014-10-21-develop-env/vim_completion1.jpg)

_Line completion_: `<Ctrl-X>`, `<Ctrl-L>` will load the matching lines (white space matters!) into the menu, and from there you can move forward and backward with arrows or `<Ctrl-P>` and `<Ctrl-N>` (for _Previous_ and _Next_).

#### 3.2 Indentation

To turn indentation on, use command `:filetype indent plugin on`. You also can add this line into `.vimrc`.

For those times when you've pasted some text in and the indentation is wrong, (your indent plugin must be loaded), you can use the = command. It's probably easiest to type `10=` to re-indent the next ten lines, or to use visual mode and press `<=>`.

If you want, you can indent lines with `>>` and unindent them with `<<`.

If you are in insert mode, use `<Ctrl-D>` and `<Ctrl-T>` to change the indentation of the line (`<Ctrl-D>` decreases indentation by one level and `<Ctrl-T>` increases it by one level).

#### 3.3 Open Multiple Files

If you want to start vim with several files in a splitted window, just type

	vim -o a b c

for the horizontal split, and

	vim -O a b c

for the vertical split.

To change between the windows opened, use `<crtl+ww>`.


Here are some commands to turn one vim session (inside one xterm) into multiple windows.

	:e filename      - edit another file
	:split filename  - split window and load another file
	ctrl-w up arrow  - move cursor up a window
	ctrl-w ctrl-w    - move cursor to another window (cycle)
	ctrl-w_          - maximize current window
	ctrl-w=          - make all equal size
	10 ctrl-w+       - increase window size by 10 lines
	:vsplit file     - vertical split
	:sview file      - same as split, but readonly
	:hide            - close current window
	:only            - keep only this window open
	:ls              - show current buffers
	:b 2             - open buffer #2 in this window
 
#### 3.4 Execute shell command

To execute a shell command in vim, use `:! <command>` .

### 4. ctags+vim+Tagbar

`ctags` generates an index (or tags) file of language objects found in source files that allows these items to be quickly and easily located by a text editor or other utility. [Exuberant-ctags](http://ctags.sourceforge.net/) is the most popular tags tool in *nix world. However, the built-in Mac OS X is not exuberant-ctags and is missing most of the useful features. Command `brew install ctags` can be used to install exuberant-ctags in OS X.

To use ctags:

	ctags -R .  # create file "tags" for current directory
	ctags -R -f ./.git/tags .   # create "tags"  file under a specified directory

In Vim:

- To show the tags you’ve traversed since you opened vim, run `:tags`.
- Typing `:tag text` will cause Vim to search through tags for a tag name, 
- and `:tag /pattern` will search for pattern instead.
- `:ts` or `:tselect` shows the list.
- `:tn` or `:tnext` goes to the next tag in that list. 
- `:tp` or `:tprev` goes to the previous tag in that list.
- `:tf` or `:tfirst` goes to the first tag of the list.
- `:tl` or `:tlast` goes to the last tag of the list.

After moving the cursor to a word,

- `<Ctrl+]>` - go to definition.
- `<Ctrl+T>` - Jump back from the definition.
- `<Ctrl+W Ctrl+]>` - Open the definition in a horizontal split.
- `<Ctrl+Left MouseClick>` - Go to definition.
- `<Ctrl+Right MouseClick>` - Jump back from definition

To open the definition of a tag in a **new tab**, add the following lines in `.vimrc`:

	map <C-\> :tab split<CR>:exec("tag ".expand("<cword>"))<CR>
	map <A-]> :vsp <CR>:exec("tag ".expand("<cword>"))<CR>

Then,

- `<Ctrl+\>` - Open the definition in a new tab.
- `<Alt+]>` - Open the definition in a vertical split.


To search for a specific tag and open Vim to its definition, run command `vim -t <tag>` in your shell.

[Tagbar](http://majutsushi.github.io/tagbar/) is another useful vim plugin to browse the tags of the current file and get an overview of its structure. Before using Tagbar, map shortcut in `.vimrc`, like

	nmap tb :TagbarToggle<CR>

, which bind key `<t+b>` to show/hide Tagbar.

To save the effort of generating ctags, I wrote a cross-platform(Linux & Mac OS X) [wrapper script](https://github.com/bo-yang/misc/blob/master/gen_cscope_ctags), which can be found in [my GitHub channel](https://github.com/bo-yang/misc/blob/master/gen_cscope_ctags).

### 5. vnc

Vncserver and vncviewer are convenient for accessing remote GUI. Actually I am trying my best to avoid the use of remote GUI and VNC. Because I don't like start heavy GUI like Gnome or KDE in remote server, I will try to use [MWM](http://xwinman.org/mwm.php) or [OLVWM](http://xwinman.org/olvwm.php) if I have to. To make them fancy, you need to modify or download the corresponding `.*rc` files, `.Xdefaults` or `.Xresources` files.

### References
1. [.bash_profile vs .bashrc](http://www.joshstaiger.org/archives/2005/07/bash_profile_vs.html)
2. [What's the difference between .bashrc, .bash_profile, and .environment?](http://stackoverflow.com/questions/415403/whats-the-difference-between-bashrc-bash-profile-and-environment)
3. [How To Use SSHFS to Mount Remote File Systems Over SSH](https://www.digitalocean.com/community/tutorials/how-to-use-sshfs-to-mount-remote-file-systems-over-ssh)
4. [Vim and Ctags](http://andrew.stwrt.ca/posts/vim-ctags)
5. [Vim 101: Tags](http://usevim.com/2013/01/18/tags/)
6. [Vim and Ctags tips and tricks](http://stackoverflow.com/questions/563616/vim-and-ctags-tips-and-tricks)
7. [http://xwinman.org/](http://xwinman.org/)

