---
layout: post
title: Fix Emacs not recognizing environment variables
category: Emacs
description: How to fix Emacs not recognizing environment variables like PATH.
---

# Problem
If you start Emacs in GUI form on Linux, then your Emacs might not be able to read the environment variables that set in **.bashrc** file. This is due to the fact that the system login shell is not the same as the interactive shell. The login shell does not read the **.bashrc** file by default, instead, it reads the **.bash_profile** file which normally locates in your **$HOME** directory. And desktops like Gnome use the login shell to boot.

# Solution
So, here is a simple solution, move your environment variables configuration to **.bash_profile**, and add the following line in **.bashrc**:
```shell
source ~/.bash_profile
```
Reboot the system. When you come back, everything should works fine.

# Test
Following command in Emacs should display the **PATH** variable.
```
M-x getenv PATH <ret>
```
