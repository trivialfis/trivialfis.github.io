---
layout: post
title: Guix use manually downloaded tarball.
category: Linux
description: Download source manually for Guix.
---
By default, Guix downloads substitutes from hydra (its build farm). If the pre-build binary is not available, Guix tries to download the source from its caches or upstream. But things won't always be that easy. There are some cases that guix couldn't carry out the download by itself. For instance, you might need a http proxy to download some source packages from upstream website, or you just happen to have the tarball and don't want Guix to download it from the internet again.

One way to do it, is to ask Guix to download the tarball locally. Yes, Guix can do that. For example, when you need texlive-texmf, but the official cache is too slow while you know there's a faster mirror. In short of waiting for Guix, you can first download the tarball from that mirror to your local machine, then ask guix to use it:

	guix download /path/to/download/directory/texlive-texmf-<version>.tar.gz
	
Executing the above command, guix will "download" the tarball into its store from the local path. After that, if you execute `guix build texlive-texmf`, guix will recognize and use the tarball in store. Be careful though, the version needs to be matched, because guix will check the hash code against the one recorded inside the corresponding package in your profile.
