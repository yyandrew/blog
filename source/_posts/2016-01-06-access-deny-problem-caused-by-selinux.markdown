---
layout: post
title: "access denied problem caused by SELinux"
date: 2016-01-06 14:22
comments: true
categories: "System"
---

前不久将dedicate server的centos重装了一下，部署网站的时候始终会出现报502错误。通过设置网站主目录的拥有者和组以及访问权限都不能解决问题。最后深挖之后发现是Centos自带的[SELinux](https://wiki.centos.org/HowTos/SELinux)造成的。

如果你地发现`/var/log/audit/audit.log`中出现了`type=AVC msg=audit(1452058471.094:70): avc:  denied  { open } for  pid=1819 comm="php-fpm" name="index.php" dev=dm-0 ino=12716944 scontext=system_u:system_r:httpd_t:s0 tcontext=unconfined_u:object_r:user_home_t:s0 tclass=file`(关键字type=AVC)之类的错误信息，那么你需要用到下面的解决方法。

以php-fpm为例, 解决方法如下
```bash
grep php-fpm /var/log/audit/audit.log | audit2allow -M php-fpm
semodule -i php-fpm.pp
semodule -R
```
更加详细的解决方法请参照[SELinux policy for nginx and GitLab unix socket in Fedora 19](http://axilleas.me/en/blog/2013/selinux-policy-for-nginx-and-gitlab-unix-socket-in-fedora-19/)
