---
layout: post
title: "Ubuntu 15.04上安装配置powerline"
date: 2015-08-22 21:22
comments: true
categories: "Tools"
---

1. 安装pip和git:打开terminal: `Ctrl`+`Alt`+`t`

   `sudo apt-get install python-pip git`

2. 继续在terminal运行

   `pip install --user git+git://github.com/Lokaltog/powerline`

3. 编辑~/.profile,将~/.local/bin添加到$PATH

   `vim ~/.profile`

   将下面内容添加到文件的最后

       if [ -d "$HOME/.local/bin" ]; then
         PATH="$HOME/.local/bin:$PATH"
       fi
4. 安装字体（不安装的话可能出现部分特殊字符乱码）

   1) `git clone https://github.com/powerline/fonts.git`

   2) `cd fonts && ./install.sh`

5. 使用powerline

   1) ***在Vim中使用***将下面内容添加到~/.vimrc

       source $HOME/.local/lib/python2.7/site-packages/powerline/bindings/vim/plugin/powerline.vim
       set laststatus=2

   2) ***在Bash中使用***将下面内容添加到~/.bashrc
   
       if [ -f ~/.local/lib/python2.7/site-packages/powerline/bindings/bash/powerline.sh ]; then
         source ~/.local/lib/python2.7/site-packages/powerline/bindings/bash/powerline.sh
       fi

6. ***附加：将terminal设置成256色***将下面内容添加到~/.bashrc
       
       if [ -n "$DISPLAY" -a "$TERM" == "xterm" ]; then
         export TERM=xterm-256color
       fi
7. 效果图
      ![powerline](/images/powerline.png)
