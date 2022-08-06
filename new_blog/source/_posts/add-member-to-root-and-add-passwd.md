---
title: Server Add Member and Passwd
date: 2022-08-05 14:37:32
tags: 
- Service
- Handbook
---

1. 登陆机器，进入 root
   
   ```shell
   smc toc 10.129.112.84
   SSE@10-129-112-84:~$ sudo su -
   root@10-129-112-84:~#
   ```

2. 增加 sudoers
   
   ```shell
   vim /etc/sudoers
   ```
   
   显示如下：
   
   ```shell
   # This file MUST be edited with the 'visudo' command as root.
   #
   # Please consider adding local content in /etc/sudoers.d/ instead of
   # directly modifying this file.
   #
   # See the man page for details on how to write a sudoers file.
   #
   Defaults    env_reset
   Defaults    mail_badpass
   Defaults    secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
   
   # Host alias specification
   
   # User alias specification
   
   # Cmnd alias specification
   
   # User privilege specification
   root    ALL=(ALL:ALL) ALL
   
   # Members of the admin group may gain root privileges
   %admin ALL=(ALL) ALL
   
   # Allow members of group sudo to execute any command
   %sudo    ALL=(ALL:ALL) ALL
   
   # See sudoers(5) for more information on "#include" directives:
   
   #includedir /etc/sudoers.d
   ```

3. 在 `User privilege specification`  中增加 sudoers 权限
   
   ```shell
   hang.gao        ALL=(ALL:ALL) ALL
   zhenghai.tan    ALL=(ALL:ALL) ALL
   dongtf  ALL=(ALL:ALL) ALL
   ```

4. `:wq!` 退出，已经加入了我们的 sudoers 了

5. 给 sudoers 加密码
   
   ```shell
   passwd zijian.zeng
   New password: [Add your passwd]
   Retype new password: [Retype your passwd]
   
   passwd: password updated successfully
   ```

6. 可以愉快得加权限了。
