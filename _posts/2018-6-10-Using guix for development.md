---
layout: post
title: Using Guix for development
category: Linux
description: Using Guix for creating development environment.
---
# Table of Contents #
- [Introduction](#introduction)
- [First example `--ad-hoc`](#first-example)
  - [Where are those packages?](#where)
  - [More details](#details)
- [Second example](#second-example)
- [Third example `--manifest`](#third-example)
- [Fourth example `--container`](#fourth-example)
- [Garbage collection](#garbage-collection)
- [With GUI support](#gui)
- [Another caveat](#caveat)
- [Sample union packages](#union)
- [Further reading](#further)
- [Conclusion](#conclude)
- [About GNU Guix](#about)

<a id="#introduction"></a>
# Introduction

If you haven't heard about GNU Guix, quoting from official site, it's GNU's advanced transactional package manager. You can obtain it from the [official site](https://www.gnu.org/software/guix/). There are many merits in Guix that can empower it's user. One of my favorite is using it to construct an isolated development environment. Underlying, Guix is a packages management system, just like `dnf/yum` from Fedora/CentOS, or `apt` from Debian. But Guix comes with support for transactional upgrades and roll-backs. And further, it's a functional packages management system, which means given a same set of packages as dependencies(input), it will have the same package as output. If you heard about functional programming, you should have a pretty good idea about what I'm talking. But today, I wanna demonstrate how to use Guix for constructing a development environment. Unlike virtualenv from python, Guix doesn't limit the usage of languages, nor does it need to make copy for needed packages. You told Guix what do you need, and Guix give it to you. If the packages are previously downloaded or built in your device, Guix can reuse it, no need for duplication.

I will use the phase "Here is how I do it with guix" through the rest of this tutorial. The phase simply means I'm not the official reference, my method could be wrong, due to my own mistake of changes in guix.

<a id="#first-example"></a>
# First example `--ad-hoc`

Well, this article is about demonstration, not reference. So you won't get any detailed reference. Instead, you get examples.

Imagine that someday, you need to work on a python based project called **foo**, which uses `numpy` for matrix calculation, you would first make an empty directory, possibly use `virtualenv` to copy your python distribution into this directory, then install `numpy`.

So here is how I do it in Guix. For our **foo** example, first, `chdir` to your project directory, enter:

	`guix environment --ad-hoc python python-numpy`

With the above command, you will see messages about Guix downloading or building `python` and `numpy`, then you will enter a new shell with these two particular packages available. Try `import numpy` with python3(not python, it's python3) in that shell, as you will see, it's available for you now. The `--ad-hoc` argument means that you want `python` and `python-numpy` as immediate packages. Without this argument, Guix will give you the dependencies of `python` and `numpy`, which are those packages needed for compiling `python` and `numpy`. You don't want to compile them, rather, you want to use them for your project, hence the `--ad-hoc` argument.

You can also add `--pure` argument to the above command:

	`guix environment --ad-hoc python python-numpy --pure`

In this case, Guix will remove all existing environment variables predefined in your shell. Which means the new environment is **pure**. You won't be able to access any existing commands in your system except for `python3`. That sounds weird at first, why would anyone do that? Well, actually it's a good thing, and sometimes it's a must. Without `--pure`, the `python` offered by Guix could accidentally access `site-packages` from your original system path, which could cause problems due to mismatched versions. And it won't be an isolated environment anymore. In short, `--pure` ensures the needed packages work as intended and the reproducibility of your project.

When you are done. You can exit this shell like any other terminal shell, enter `Ctrl-D` or `exit`. The rest of your system won't be changed in anyway.

<a id="#where"></a>
## Where are those packages?

The downloaded packages are stored in /gnu/store/. The file path for each package contains hash code for it. You won't be able to access them unless you specify the full path. So it's completely isolated from the rest of your system.

<a id="#details"></a>
## More details

The created environment is a profile. After invoking the command, Guix creates a directory within /gnu/store with symbol links for those needed packages gathered. You can check the actual path of that directory with

	`echo $GUIX_ENVIORNMENT`

The path is hashed, remember when I said that Guix is a functional packages management system? When you specify the same packages next time you invoke `guix`, you get the same environment, same path. Unless you have ungraded Guix and the corresponding packages' dependencies tree changed.

<a id="#second-example"></a>
# Second example

Having `numpy` and `python`  doesn't mean you would start writing python code immediately, you need helper tools. Like `flake8` for syntax check, `yapf` for code structuring etc. So you would install them as well. For one python project, that's probably ok, what if you have several? Or even worse, what if you just want to read others source code to learn some implementations? Do you setup such an environment for each project? I would rather not. Ok, now, what if someday you want to see what's under the hook of `autograd` (an implementation for auto gradient written in python). Then you need all its dependencies, helper tools, since you don't want to read other's code without code jumping tools right? Here is how I do it with Guix:

	`guix environment python-autograd --ad-hoc python python-flake8 python-autopep8 python-yapf emacs --pure`

The command line is long, but don't worry, we will have more convenient method later. I put `python-autograd` before `--ad-hoc`, so that all its dependencies will be in position (but not `autograd` itself) as explained in first example. This way, when you jump around the code, you can jump to the packages that `autograd` uses as well. My Emacs has proper configuration for python which makes uses of `flake8`, `autopep8` and `yapf`. So I put them after `--ad-hoc`, as also explained in the first example. In the newly spawned shell, open Emacs (or any other text editor packaged in Guix and specified in `guix enviornment --ad-hoc`), then you can start reading the source code of `autograd` without polluting your system environment.

<a id="#third-example"></a>
# Third example `--manifest`

We can't finish our project in one day. Also, we don't want to specify these packages every time we need this environment. How do we do it? The first thing we do, is invoking `guix environment` with `--manifest` argument. You can write down your needed package in a file possibly named "foo-mainfest.scm" like this one:

``` scheme
	(specifications->manifest
		'("emacs" "python-numpy" "python" "python-flake8"))
```

Then invoke guix:

	`guix environment --manifest ./foo-manifest.scm`

But there's a caveat about manifest, you need to specify every package you need, which means all packages specified in manifest file are `--ad-hoc` packages.

<a id="#garbage-collection"></a>
# Garbage collection

After a while, you might have a lots of unused packages inside /gnu/store, then you would want to delete them. You can do so with:

	`guix gc`

Guix will check those packages that weren't used anymore and delete them accordingly. But you might wanna preserve some environment you created while doing garbage collection. Here is how I do it with guix when creating that particular environment:

	`guix enviroment <packages> --profile ./foo-profile

With this command, at the same time of creating an environment, guix also adds a symbol link "foo-profile" to the profile path of this environment. When you invoke `guix gc`, guix will spare those packages needed by this environment.

<a id="#forth-example"></a>
# Fourth example `--container`

What if we want to install the project we are working on, but still don't want to pollute the environment? That's the real world, right? We tried to modify some software while at the same time we are using them. That the job for `--container`. Continuing our **foo** example, here is how I do it with Guix:

	`guix environment --ad-hoc python python-numpy --container`

In this way, guix will create a environment with changed root. Which means you won't be able to see any files other than those within your working directory and its sub-directories. Within this shell, you have root privilege. You can install the project as you would without polluting your system thanks to the changed root. But after exiting the environment, you need to call

	`guix gc -d <path to that profile>`

to clean up your installation, so that those installed files won't waste your storage space. An caveat is that you need to re-install your project every time you create the environment. To me it's not a big deal, since that's the point of creating a contained environment, used only during development and the project is re-installed frequently for testing purpose.

<a id="#gui"></a>
# With GUI support

Currently, I'm running Guix on top of Fedora, my default desktop environment is Gnome. You might have noticed that the shell spawned by `guix environment` is just a text shell. It's powerful but still, we might miss the convenience brought by a GUI shell, like opening random files with simple clicks. Spawning a GUI shell is much more complex, which Guix doesn't support. But lucky for us, there's a simple work around. You can just make the file manager as an ad-hoc package. In my case, it's nautilus. If you are using other desktop environment, choose the corresponding one. Currently Guix supports Xfce and Gnome. After spawning the new environment with

	`guix environment --ad-hoc nautilus <packages...>`

Then simply enter nautilus (or the name of your favorite file manager) in the newly created shell, the file manager window should pop out. In such a case, this window is contained within the guix shell. If you open a file with emacs in this window, Emacs can access any packages you have specified, and with corresponding environment variable set. It's worth noting that you can't use this work around with `--container` option, nautilus won't be able to connect to your desktop environment with changed root.

<a id="#caveat"></a>
# Another caveat

After specifying an environment, you can't change it without leaving that environment. The environment is hashed based on it's packages. If you add or remove a package, the environment is changed, which means another environment is created. You can't have a same hash code with two different environments.

<a id="#union"></a>
# Sample union packages

I promised there is a more convenient way to create environment with frequently used packages. But the convenience requires some knowledge about packaging for Guix. You can create your own packages composed of those packages you use frequently. If you want to do so, continue reading. Or better, try the official [packaging guide](https://www.gnu.org/software/guix/manual/guix.html#Defining-Packages).

I have merged some basic tools into union [packages](https://github.com/trivialfis/guixpkgs/blob/master/programming-trivialfis.scm), you can check it out and use the code within as an example for creating you own packages. The link contains 3 packages you might find useful. All of those are union packages, which means they are packages that composed by other packages. Inside the file, you would find `basic-programming`, which combines `coretuils`(command tools like ls), `nautilus`, and some other tools including a customized version of `Emacs`. Find these words, and replace them as needed. One thing to notice is that you need to declare the module you use in file header. You can search the packages you need with the following command:

	`guix package -s python-autograd`

In the resulting message, you will find a line looks similar to this one:

	`location: machine-learning.scm:45:4`

It means `python-autograd` is defined in file **machine-learning.scm**. Then `machine-learning` is the module you need to add at the top of the file that you declare your union package. As for how to declare, I believe a smart person like you is perfectly capable of finding out by the example provided in above link with the help of [official packaging guide](https://www.gnu.org/software/guix/manual/guix.html#Defining-Packages). After creating your own package, set the environment variable `GUIX_PACKAGE_PATH` to point to the directory containing your package.

<a id="#further"></a>
# Further reading

Anyway, Guix is a powerful package management system. Here I'm only scratching the surface. Try to read more at the [official documentation](https://www.gnu.org/software/guix/manual/guix.html). If you like some more tutorial, there's an [official blog](https://www.gnu.org/software/guix/blog/) containing many interesting use cases. Enjoy.

<a id="#conclude"></a>
# Conclusion

I hope this blog post provides some intuition about using guix to ease your development. Guix provides `guix environment` command you can utilize to make different environments to suite your own needs.

<a id="#about"></a>
# About GNU Guix

GNU Guix is a transactional package manager for the GNU system. The Guix System Distribution or GuixSD is an advanced distribution of the GNU system that relies on GNU Guix and respects the user's freedom.

In addition to standard package management features, Guix supports transactional upgrades and roll-backs, unprivileged package management, per-user profiles, and garbage collection. Guix uses low-level mechanisms from the Nix package manager, except that packages are defined as native Guile modules, using extensions to the Scheme language. GuixSD offers a declarative approach to operating system configuration management, and is highly customizable and hackable.

GuixSD can be used on an i686, x86_64 and armv7 machines. It is also possible to use Guix on top of an already installed GNU/Linux system, including on mips64el and aarch64.

[1]: https://www.gnu.org/software/guix/

<!--  LocalWords:  coretuils CentOS reproducibility
 -->
