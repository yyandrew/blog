---
layout: post
title: "在bash中判断一个package是否已经安装"
date: 2015-11-29 10:47
comments: true
categories: "Bash"
---
在bash中判断nginx是不是已经安装

``` bash
#!/bin/bash
function package_exists() {
  return dpkg -l "$1" &> /dev/null
}
if ! package_exists nginx ; then
  echo "nginx will be install..."
  apt-get install nginx
fi
```
