---
layout: post
title: How To Install Jekyll in Mac OSX Mavericks
categories: 
- notes
- Jekyll
- Github
tags:
- Notes
- Jekyll
status: publish
type: post
published: true
author: Bo Yang
---

1. <a href="http://xthinking.com/github/2014/03/17/github_install_jekyll_mavericks.html">Procedure of installing Jekyll in Mac OS X</a>

2. Install Xcode Command line tools: <a href="http://stackoverflow.com/questions/9329243/xcode-4-4-and-later-install-command-line-tools">http://stackoverflow.com/questions/9329243/xcode-4-4-and-later-install-command-line-tools</a>

3. Clang error during install Jekyll

~~~
clang: error: unknown argument: '-multiply_definedsuppress' [-Wunused-command-line-argument-hard-error-in-future]
clang: note: this will be a hard error (cannot be downgraded to a warning) in the future
make: *** [redcarpet.bundle] Error 1

make failed, exit code 2
~~~

Solution:

~~~
export ARCHFLAGS="-Wno-error=unused-command-line-argument-hard-error-in-future"
export CFLAGS=-Wunused-command-line-argument-hard-error-in-future
export CPPFLAGS=-Qunused-arguments
sudo -E gem install jekyll
~~~

[Note]: Above is only a temporary workaround for Xcode 5.1.

##### References:

* <a href="http://xthinking.com/github/2014/03/17/github_install_jekyll_mavericks.html">http://xthinking.com/github/2014/03/17/github_install_jekyll_mavericks.html</a>
* <a href="http://stackoverflow.com/questions/22313407/clang-error-unknown-argument-mno-fused-madd-python-package-installation-fa">http://stackoverflow.com/questions/22313407/clang-error-unknown-argument-mno-fused-madd-python-package-installation-fa</a>
* <a href="http://forums.getpebble.com/discussion/11862/installation-error-perhaps-due-to-xcode-5-1">http://forums.getpebble.com/discussion/11862/installation-error-perhaps-due-to-xcode-5-1</a>
* <a href="https://langui.sh/2014/03/10/wunused-command-line-argument-hard-error-in-future-is-a-harsh-mistress/">https://langui.sh/2014/03/10/wunused-command-line-argument-hard-error-in-future-is-a-harsh-mistress/</a>
* <a href="http://stackoverflow.com/questions/9329243/xcode-4-4-and-later-install-command-line-tools">http://stackoverflow.com/questions/9329243/xcode-4-4-and-later-install-command-line-tools</a>
