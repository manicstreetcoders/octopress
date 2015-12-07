---
layout: post
title: "Shell Initialization"
date: 2013-09-03 23:28
comments: true
categories: 
---

[Unix shell initialization](https://github.com/sstephenson/rbenv/wiki/Unix-shell-initialization)

### Shell modes

- login shell - e.g. when you login from another host, or login at the text console of a local unix machine
- interactive shells - ones connected to a terminal (or pseudo-terminal in the case of, say, a terminal emulator running under a windowing system).

- `-l`, `--login`
- `-i`

.bashrc in only read by a shell that's both interactive and non-login.

### OS X

- Opening a new Terminal window/tab: .bash_profile
- Logging into a system via SSH: .bash_profile
- Executing a command remotely with ssh or Capistrano: .bashrc
