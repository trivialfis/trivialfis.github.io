---
title: Fix Fedora DNF bash completion
layout: post
category: Linux
description: If your bash completion doesn't work with Fedora DNF, here is why and how to fix it.
---
# Introduction

For somebody who doesn't know what is DNF. DNF is the default packages management system for Fedora which is written in python. For managing packages, it works perfectly. But there are other minor issues in DNF like bash completion. It doesn't work on a freshly installed Fedora.

# Fix

Just install the `sqlite` with following command:

```sh
sudo dnf install sqlite
```

The /usr/share/bash-completion/dnf script require **sqlite3** command which is provided by `sqlite` package. Requiring sqlite as a dependency of DNF is actually proposed in the Bugzilla.

<!--  LocalWords:  sqlite
 -->
