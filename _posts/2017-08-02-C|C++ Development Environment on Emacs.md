---
layout: post
title: C|C++ Development Environment for Emacs
category: Emacs
description: A tutorial to help you set up Emacs for Modern C++ Integrated Development Environment (IDE).
---

# Table of Contents #

- [Introduction](#org2f59065)
- [Packages list](#orga12f599)
- [External requirement](#orgc8c763c)
  - [Optional](#org409d4ec)
- [Some emacs lisp basic](#org9hdz8p)
- [Extra code to make life easier](#orgbbfb9db)
- [Generating compilation database](#orgbd54f0d)
  - [What is compilation database](#orgdd8d1c2)
  - [Generate the database](#orgdbe0275)
- [Using irony](#org5720fa8)
  - [What is irony](#org01604bc)
  - [Auto completion](#orgbbb15e6)
  - [Syntax check](#orga9d8d5b)
  - [Brief summary for symbols](#org37d8b53)
  - [As a whole](#orgb32fc19)
- [Using rtags for navigation and refactoring](#org97a10f6)
  - [Why rtags](#org7b9a494)
  - [Set it up then](#org8d14e06)
- [Use semantic as an alternative](#orgd256286)
  - [Introduction](#orgfd7e4fa)
  - [Basic settings](#orgf4b56be)
  - [Srefactor](#orgd6477df)
  - [As a whole](#org2247c70)
- [Code browsing](#orgfd61392)
- [Format the code](#org6d3156f)
- [Disassemble](#orgf8ee6e5)
- [Debugging](#orgd738ec9)
  - [Emacs build in support:](#orgb1b94c5)
  - [RealGUD](#org44c60c5)
- [Project](#org4d0da4c)
- [Summary](#org2954831)



<a id="org2f59065"></a>

# Introduction

Emacs is a extremely powerful and flexible text editor which can bend to almost everything a programmer might need. There's a joke enjoyed by both Emacs users and haters calling Emacs as the *Emacs operating system* for it's fully featured extension language `emacs lisp`. With the many packages others have developed and some proper glue, we can build a IDE like environment for writing C/C++ code. And most importantly, it can be tailored to your own need.

There are some pre-configured Emacs out there like `spacemace`, `prelude` which will set up everything you need, why would you want a introduction for setting things up yourself? The answer is customization. The very important aspect for using Emacs is you can have your OWN editor. By own, I mean you can configure Emacs to meet your OWN need, create your own environment and develop your OWN habit. You can hack into it with a great joy. Besides, from my point of view, the tremendous amount of code they plugged into Emacs is just another layer preventing you to know about Emacs and a rigid framework stopping you from customizing it.

At some point, I decided to write something for others to get started using Emacs. And hence this tutorial. I run Emacs 25 on Fedora 26, some of the packages I use require Emacs 25+ to work. For other platforms that don't have Emacs 25 in official repositories yet, you can compile Emacs from source or checkout this [ppa](https://launchpad.net/~ubuntu-elisp/+archive/ubuntu/ppa) or Emacs [official site](https://www.gnu.org/software/emacs/).


<a id="orga12f599"></a>

# Packages list

In this tutorial, I will introduce some packages related to dealing with a C/C++ project. In practice, there are many choices for accomplishing the same goad. But the following packages are especially recommended. You can always try whatever else you want in your spare time, at the time of this writing Emacs has over 4000 packages available, make use of them. Here is a list of the packages I will use in this tutorial, I added the link to their respective official site for reference:

-   [helm](https://github.com/emacs-helm/helm): An Emacs incremental and narrowing framework.
-   [rtags](https://github.com/Andersbakken/rtags): A c/c++ client/server indexer for c/c++/objc[++] with integration for Emacs based on clang.
-   [helm-rtags](https://github.com/Andersbakken/rtags): A front-end for rtags.
-   [company-rtags](https://github.com/Andersbakken/rtags): RTags back-end for company.
-   [irony](https://github.com/Sarcasm/irony-mode): C/C++ minor mode powered by libclang.
-   [irony-eldoc](https://github.com/ikirill/irony-eldoc): irony-mode support for eldoc-mode.
-   [company](https://github.com/company-mode/company-mode): Modular text completion framework.
-   [company-irony](https://github.com/Sarcasm/company-irony): Company-mode completion back-end for irony-mode.
-   [company-irony-c-headers](https://github.com/hotpxl/company-irony-c-headers): Company mode backend for C/C++ header files with Irony.
-   [flycheck](http://www.flycheck.org/en/latest/): On-the-fly syntax checking.
-   [flycheck-irony](https://github.com/Sarcasm/flycheck-irony/): Flycheck: C/C++ support via Irony.
-   [flycheck-rtags](https://github.com/Andersbakken/rtags): RTags Flycheck integration.
-   [cmake-ide](http://github.com/atilaneves/cmake-ide): Calls CMake to find out include paths and other compiler flags.
-   [cmake-mode](https://github.com/Kitware/CMake): major-mode for editing CMake sources.
-   [projectile](https://github.com/bbatsov/projectile): Manage and navigate projects in Emacs easily.
-   [disaster](https://github.com/jart/disaster): Disassemble C/C++ code under cursor in Emacs
-   [ecb](https://github.com/alexott/ecb): A code browser for Emacs
-   clang-format: Format code using clang-format
-   [use-package](https://github.com/jwiegley/use-package): A use-package declaration for simplifying your .emacs.

-   [semantic](http://cedet.sourceforge.net/semantic.shtml): API for providing the semantic content of a buffer.
-   [srefactor](https://github.com/tuhdo/semantic-refactor): A refactoring tool based on Semantic parser framework.

All the packages listed above can be either installed from `melpa` or bundled with Emacs as a build in package. For how to install package from `melpa`, please refer to the [official getting started](https://melpa.org/#/getting-started). In the rest of this tutorial, I will assume these packages are installed via Emacs packages manager. Of course, you don't have to install them all in a batch right now, just read along, see what's worth.


<a id="orgc8c763c"></a>

# External requirement

Although Emacs can accomplish almost everything with a Turing complete extension language, but it's still a text editor. It has a build in package named `semantic` which is a part of `CEDET` (Collection of Emacs Development Tools). `Semantic` can generate an C++ AST (Abstract syntax tree) all by itself. But heavy stuffs like parsing your code would be much better offloaded to a real compiler. After all, `emacs lisp` is slow and there's really no reason to reinvent the wheel, especially such an expensive one. Some of the above listed packages rely on the clang compiler to work.

-   clang: A compiler can be used to help coding on Emacs.
-   cmake: For building stuffs from source.
-   [bear](https://github.com/rizsotto/Bear): For generating compilation database.


<a id="org409d4ec"></a>

## Optional

-   global: A GNU code indexer.

Note that `irony` and `rtags` needs `clang-devel` to work on Fedora since you need to compile it from source. The name of both Emacs packages and system package (external requirement) will have the format like `helm`, `clang`. Since there are only 4 system packages in this tutorial, it should be easy enough to distinguish.

<a id="org9hdz8p"></a>

# Some emacs lisp basic
Please remember that you don't have to know anything in this section to go through the tutorial, but knowing some basic concept will smooth the hurdle greatly in your future with Emacs.

Comments:
```emacs-lisp
; A comment starts with ;
```

A function named foo without parameter:
```emacs-lisp
(defun foo ()
    ; do stuffs here
)
```

A function without name and parameter:
```emacs-lisp
(lambda ()
    ; do stuffs here
)
```

Add an anonymous function to a hook, a hook is a list where stores some functions to be run at appropriate time.
```emacs-lisp
(add-hook 'c++-mode-hook '(lambda ()
    ;; do stuffs
))
```


<a id="orgbbfb9db"></a>

# Extra code to make life easier

Put the following code in your `.emacs` or `.emacs.d/init.el` to make configuring key binding easier:

```emacs-lisp
(defun trivialfis/local-set-keys (key-commands)
  "Set multiple local bindings with KEY-COMMANDS list."
  (let ((local-map (current-local-map)))
    (dolist (kc key-commands)
      (define-key local-map
	(kbd (car kc))
	(cdr kc)))))
```

The `trivialfis` is just my prefix, you can remove it safely if you are not happy about it. Just remember to remove it in all the following code snippets. I will use the tiny function to define local key bindings in the rest of this tutorial.


<a id="orgbd54f0d"></a>

# Generating compilation database


<a id="orgdd8d1c2"></a>

## What is compilation database

Compilation database is defined by the clang project. A compilation database is a json file which specifies the compile commands for each file in the project. With such a record, clang can have a proper compile configuration to support tools that utilizing its AST without actually running the build system. For detailed description of compilation database, please refer to clang's [official documentation](http://clang.llvm.org/docs/JSONCompilationDatabase.html) or just look at it with your Emacs. As mentioned above, some of the packages we will require support from clang to work, so generating a compilation database is one way to make our environment fully working. Don't worry about the extra steps, they were mostly automated. Just don't be alarm if there is a json file suddenly pop out in your project root. Here is a simple one:

```javascript
// compile_commands.json
[
    {
	"arguments": [
	    "c++",
	    "-c",
	    "-std=c++14",
	    "-o",
	    "main",
	    "main.cpp"
	],
	"directory": "/home/fis/Workspace",
	"file": "main.cpp"
    }
]
```


<a id="orgdbe0275"></a>

## Generate the database

With a cmake project, the Emacs package `cmake-ide` will generate the proper compilation base for us automatically. For other projects that doesn't come with cmake, we can use some help from the `bear` project to generate the required compilation database. Generating a compilation database for these project using `bear` are quite straight forward, just run `bear make` in the directory where you place your Makefile with system command shell.


<a id="org5720fa8"></a>

# Using irony


<a id="org01604bc"></a>

## What is irony

`Irony` is a powerful tool utilizing `clang` to do symbol completion, syntax checking and symbol information extraction. Before configuring it, we need `clang-devel` (on Fedora) first. After `clang-devel` is installed on your system path, `irony` can look it up by itself. Now you need to use `M-x irony-install-server` inside Emacs to compile and install the binary executable of `irony` as an external program to help the elisp package communicate with `clang`. Once the server is installed, you are ready to make `irony` work with Emacs.

One thing worth noting is, due to the fact that `irony` is completely dependent on `clang`, so generating a compilation database using the methods described above when you are writing code is best practice. Although `irony` comes with default settings, but it's very primitive and might not the sufficient for your project. If you have a compilation database in your project root, opening any c/c++ files inside the project should have `irony` load the database automatically.


<a id="orgbbb15e6"></a>

## Auto completion

For auto completion, we will make use of `company` and `irony`. `Company` (Complete anything) is a frontend for auto completion. The package defined a set of backends stored in a list named `company-backends` and we can always plug any other backends into the list. But since we won't be needing those predefined backends and sometimes they would complete with each others, I will removes all the predefined backends at first:

```emacs-lisp
(setf company-backends '())
```

Then get the `company-keywrods` backend back:

```emacs-lisp
(add-to-list 'company-backends 'company-keywords)
```

Now we can configure `irony` with `company`, it's just two more line of code:

```emacs-lisp
(add-to-list 'company-backends 'company-irony)
(add-to-list 'company-backends 'company-irony-c-headers)
```

You can group the above code into a function and then add it to the `c++-mode-hook`:
```emacs-lisp
(add-hook 'c++-mode-hook #'(lambda ()
;;	...code here...
	))
```
There you have it:
![company]({{ site.url }}/assets/C|C++-Development-Environment-on-Emacs/company.png "Emacs IDE company")

<a id="orga9d8d5b"></a>

## Syntax check

For syntax checking, again we will use `irony`. Having `irony` as a backend, we still need a frontend to highlight the problematic code and tell us about the error and warning, which is exactly the job for `flycheck`. Add the following line to your configuration file:

```
(add-hook 'flycheck-mode-hook 'flycheck-irony-setup)
```

Then Emacs should display the error information properly for you. But I have added some extra configuration for `flycheck` so that it can display all errors in a buffer properly.

```emacs-lisp
(defun trivialfis/flycheck ()
  "Configurate flycheck."
  (add-to-list 'display-buffer-alist
	       `(,(rx bos "*Flycheck errors*" eos)
		 (display-buffer-reuse-window
		  display-buffer-in-side-window)
		 (side            . bottom)
		 (reusable-frames . visible)
		 (window-height   . 0.23)))
  (setq flycheck-display-errors-function
	#'flycheck-display-error-messages-unless-error-list))
```

Just add this function to Emacs's `prog-mode-hook` with the following line of code:

```emacs-lisp
(add-hook 'prog-mode-hook 'trivialfis/flycheck)
```

You can use the default key binding `C-c ! l` or use `M-x flycheck-list-errors` to display all errors in a buffer. Another way to do the same thing is via tool bar `<Tools> <Syntex Checking> <Show all errors>`. The above code would make the window displaying all errors persistence, which means it won't affected by windows manipulation commands like `delete-other-windows` ( `C-x 1` ). Other than this, it will also ask `flycheck` not to display error message in echo area while the **Flycheck errors** buffer is displayed.

All in all, your Emacs should look something like this:
![flycheck]({{ site.url }}/assets/C|C++-Development-Environment-on-Emacs/flycheck.png "Emacs IDE flycheck")

<a id="org37d8b53"></a>

## Brief summary for symbols

`Irony` also integrates well with the Emacs build in `eldoc`. With this integration, you can view a brief summary of the symbol under cursor in echo area. Just add this line of code to your configuration file:

```emacs
(add-hook 'irony-mode-hook 'irony-eldoc)
```

Here is a screenshot:

![eldoc]({{ site.url }}/assets/C|C++-Development-Environment-on-Emacs/eldoc_summary.png "Emacs IDE symbol summary")
<a id="orgb32fc19"></a>

## As a whole

I wrote a little function to configure `irony`. In my configuration file, this function is added to the C/C++-mode-hook as a part of another function named `trivialfis/cc-base` (where the code of removing company's backends live). Following is the `irony` configuration function:

```emacs-lisp
(defun trivialfis/irony ()
  "Irony mode configuration."
  (add-hook 'irony-mode-hook 'irony-eldoc)
  (add-to-list 'company-backends 'company-irony)
  (add-to-list 'company-backends 'company-irony-c-headers)
  (add-hook 'irony-mode-hook 'irony-cdb-autosetup-compile-options)
  (add-hook 'flycheck-mode-hook 'flycheck-irony-setup)
  (when (or (eq major-mode 'c-mode)	; Prevent from being loaded by c derived mode
	    (eq major-mode 'c++-mode))
    (irony-mode 1)))
```

I turned on the `irony` minor mode at the end of the function. You can wrote your own `cc-base` function to configure various C/C++ setting and add it to the `c++-mode-hook` yourself. Something like this:

```emacs-lisp
(defun trivialfis/cc-base ()
  "Common configuration for c and c++ mode."
  ;; Company mode
  (setf company-backends '())
  (add-to-list 'company-backends 'company-keywords)
  (trivialfis/irony))
;; Add it to c++-mode-hook
(add-hook 'c++-mode-hook 'trivialfis/cc-base)
```


<a id="org97a10f6"></a>

# Using rtags for navigation and refactoring


<a id="org7b9a494"></a>

## Why rtags

Rtags is a tagging system for C/C++ utilizing `clang` as its backend. Actually there are other choices than `rtags`. And `rtags` is really not well documented with little comment in its source code. But here are the reasons why I chose to stick with it.

There is a traditional option `Global` with `gtags` for generating tag databases which supports many different languages and used in lots of projects. It can do a great job for C, but when it comes to C++, `Gloabl` lacks the ability to deal with the complex syntax rules and name scope since it parses the code literally without the ability to generate a proper AST and understand the context. Besides, C++ is a fast evolving language. I believe it's the best not to ask the tagging system to do the compiler's job.

On the other hand, we have the build in `semantic` package which can generate a C++ AST all by itself so that it can do context sensitive operations. And this package has many neat features that other packages doesn't provide like displaying a sticky function name on head line or providing support for `ECB` for code browsing. But that is also where the problem lies in. It have to follow the development of C++ and reimplement a part of compiler. Reinventing the wheel is one thing, maintaining it is another. When parsing modern C++ code it will generate a large amount of errors. Even it works correctly, parsing code is a heavy job, doing it with Emacs lisp is not really a good idea. Maybe one day its features can be ported to some other tools like `rtags` or `xref` and I'm really looking forward to it.


<a id="org8d14e06"></a>

## Set it up then

Using `rtags` requires you to compile it from source, there is no binary distribution, yet. But don't worry, the procedure is fairly straight forward thinks to Cmake and git. As a prerequest, you need a compiler that can properly compile C++ 11 code, cmake for building system and git for cloning the code from Github. Run the following commands to install `rtags` from source:

```sh
git clone --recursive https://github.com/Andersbakken/rtags.git
cd rtags
mkdir build
cd build
cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=1 ..
make
sudo make install
```

You might want to `-j` option to utilize your multi-cores CPU. The above commands should install the `rtags` to you system. If things didn't run smoothly as expected, please consult the project's README file. After installing `rtags` the external program, you still need to install the corresponding Emacs package. The best way to do it is via `melpa`. Go and find the package named `rtags` and `rtags-helm` and install them. `rtags-helm` requires `helm` to work. The `helm` package can provide us a better interface for viewing tag references.

As described earlier, `clang` needs a compilation database to work and `rtags` relies on `clang`. So if you are not working on the Cmake project, you need to generate the compilation database yourself with `bear`. After having a the compilation database, `cmake-ide` will do the rest of the work of taking cares of `rtags`. `rtags` also provides code completion and syntax check for C++ code, but it doesn't integrate with `eldoc` for displaying symbols' summary (I did see something related to eldoc in rtags' source code, not sure what it does). And most importantly, `irony` is easier to set up and works great.

After installing the `rtags`, put the following code in your C/C++ configuration file. If you don't have a separated file for C/C++, then just put it into your `.emacs` of `.emacs.d/init.el` (It's highly recommended to have separated file for different configuration and use autoload so that you can gain extra speedup for starting Emacs):

```emacs-lisp
(use-package rtags
  :commands rtags-start-process-unless-running
  :config (progn
	    (message "Rtags loaded")
	    (use-package company-rtags)))

(defun trivialfis/rtags ()
  "Rtags configuration.
Used only for nevigation."
  (interactive)
  (rtags-start-process-unless-running)
  (setq rtags-display-result-backend 'helm)
  (trivialfis/local-set-keys
   '(
     ("M-."     .  rtags-find-symbol-at-point)
     ("M-,"     .  rtags-find-references-at-point)
     ("C-c r r" .  rtags-rename-symbolrtags-next-match)
     ))
  (add-hook 'kill-emacs-hook 'rtags-quit-rdm))
```

The above code makes use of the package `use-packge` to load `rtags`, if you don't want to bother, just use `(require 'rtags)` and remove `(use-package ...)`. Here I have binded 3 key bindings with `rtags`, the name of binded function should be self-explainetary, just try it, see what happens. There are some other function you can use interactivity. With `helm`, you can use `C-h f` (`describe-function`) and enter "rtags" to find out what other function from `rtags` you can use. Further more, you can read through the README file of `rtags` to hack it yourself. But before that, this simple configuration should be enough for daily usage.


<a id="orgd256286"></a>

# Use semantic as an alternative


<a id="orgfd7e4fa"></a>

## Introduction

`Semantic` is a build in package of Emacs and is a sub-package of `CEDET` (Collection of Emacs Development Tools). It provides a great deal of powerful features for programmers. You can even define your own language syntax rules to make it work with your new language. And it's a self-contained packages that doesn't require any dependency other than a recent Emacs to work properly. As mentioned in the `rtags` section, it itself can parse the source code, build the corresponding AST and do the tagging. Relying on the tags database generated, it can be used to navigate the code, highlight tags, do auto completion, refactor, show symbol summary, display current function at header line and many other things. Basically it's a package for everything you need. There is an excellent tutorial written by Alex Ott: [A Gentle introduction to CEDET](http://alexott.net/en/writings/emacs-devenv/EmacsCedet.html). and the official document can always be accessed by `C-h i` and search for `semantic`. When you are only dealing with C source code or C++ 11 code provided the code base is not scary, it's your best option at hand, no doubt. Even when the code base is large, you can still shift the completion work to other packages like `irony` or `company-clang` so that semantic won't block the editor while preserving most of its functionality.

`Semantic` itself is a global minor mode, which means once you turned it on, it will affect all other buffers. If you use the Emacs server-client model, you will always have `semantic mode` in there until you turn it off explicitly. `Semantic` provides its functionality via many "sub-modes", you need to add these sub-modes to the list **semantic-default-submodes** to make it work.


<a id="orgf4b56be"></a>

## Basic settings

Here is some basic configuration for semantic mode, place the code snippet to your configuration file and call the function **trivialfis/semantic** with your current major(c-mode or c++-mode) as the parameter. The configuration added some sub-modes for `semantic` and enabled `Global` support. For the details of these submodes, you can use `C-h o RET {mode name} RET` to view a short description after turning on `semantic-mode`.

```emacs-lisp
(defun trivialfis/semantic (MODE)
  "Custom semantic mode.
MODE: the major programming mode"
  (let ((semantic-submodes '(global-semantic-decoration-mode
			     global-semantic-idle-local-symbol-highlight-mode
			     global-semantic-highlight-func-mode
			     global-semanticdb-minor-mode
			     global-semantic-mru-bookmark-mode
			     global-semantic-idle-summary-mode
			     global-semantic-stickyfunc-mode
			     )))
    (setq semantic-default-submodes (append semantic-default-submodes semantic-submodes)
	  semantic-idle-scheduler-idle-time 1)
    (semanticdb-enable-gnu-global-databases 'MODE)
    (semantic-mode 1)))
```

Here is a brief description for above activated modes mostly copied from the doc:

-   **global-semantic-decoration-mode**: Decoration mode turns on all active decorations as specified by ‘semantic-decoration-styles’.
-   **global-semantic-idle-local-symbol-highlight-mode**: Highlight the tag and symbol references of the symbol under point at every supported buffers.
-   **global-semantic-highlight-func-mode**: Minor mode to highlight the first line of the current tag.
-   **global-semanticdb-minor-mode**: In Semantic DB mode, Semantic parsers store results in a database, which can be saved for future Emacs sessions.
-   **global-semantic-mru-bookmark-mode**: You can navigate backward with this mode enabled.
-   **global-semantic-idle-summary-mode**: Display a short summary for the symbol under cursor when idle.
-   **global-semantic-stickyfunc-mode**: Minor mode to show the title of a tag in the header line.

And for the line `(semanticdb-enable-gnu-global-databases 'MODE)`, we will enable `semantic` to ask help from `Global` for parsing code. Please note that even the option is enabled, it doesn't generate a actual `Global` tags database. `Semantic` still needs to build its own AST and generate its own database.


<a id="orgd6477df"></a>

## Srefactor

`Srefactor` is a powerful package that makes use of the functionality from `semantic` to help us do refactoring which also has a simple interface. After calling the function **srefactor-refactor-at-point**, `srefactor` will display a list of options for you to choose. I am not calling it powerful only for polite. It provides some features that even a modern IDE cannot achieve. Like extracting code fragment as the separated function, generating class implementation templates and renaming symbol under cursor, etc. Here is the default key bindings for it when it display the options list:

-   `1..9` to quickly execute an action.
-   `o` to switch to next option, `O` to switch to previous option.
-   `n` to go to the next line, `p` to got to previous line.
-   `q` or `C-g` to quit.

For more info and demos, please checkout the [original repo](https://github.com/tuhdo/semantic-refactor/blob/master/srefactor-demos/demos-elisp.org).

Here is some other key bindings I have defined. As usual, you can check what other functions you can call by using `C-h f RET` with `helm`.

```emacs-lisp
(defun trivialfis/cc-base-semantic ()
  "Configuration for semantic."
  (trivialfis/local-set-keys
   '(
     ("M-RET"   .  srefactor-refactor-at-point)
     ("C-c t"   .  senator-fold-tag-toggle)
     ("C-."     .  semantic-symref)
     ("M-."     .  semantic-ia-fast-jump)
     ("C-,"     .  semantic-mrub-switch-tags)))
  (eval-after-load 'helm-trivialfis
    (local-set-key (kbd "C-h ,") 'helm-semantic-or-imenu)))
```


<a id="org2247c70"></a>

## As a whole

Add the previously defined function listed below to `c++-mode-hook` if you want to make use of `semantic`.

```emacs-lisp
(trivialfis/cc-base-semantic)
(trivialfis/semantic 'c++-mode)
```

For adding functions to hook:

```emacs-lisp
(add-hook 'c++mode-hook 'trivialfis/cc-base-semantic)
(add-hook 'c++mode-hook #'(lambda ()) (trivialfis/semantic 'c++-mode))
```


<a id="orgfd61392"></a>

# Code browsing

The canonical code browser for Emacs is the `ECB` package which is a short hand for `Emacs code browser`. It's a powerful code browsing tool and a part of `CEDET`. It can display C++ classes, variables and functions in a structural manner. But it also relies on the semantic package to work properly. You can activate it by calling `ecb-activate`, and use `ecb-toggle-layout` to switch windows layout. The switching function is bind to `C-c . l t` by default which is quite long for something called "shortcut". You can bind it to whatever you want. I myself just use `M-x` and call the function thinks to the convenience brought by `helm`. Just like any other packages provided by `CEDET`, `ECB` has `info` documentation in Emacs, use `C-h i` to check it out.

![ecb]({{ site.url }}/assets/C|C++-Development-Environment-on-Emacs/flycheck_ecb.png "Emacs IDE ecb")
One problem of `ECB` is that it has its own windows management system, which doesn't get along with .. , well, any other windows. I copied some code from `flycheck`'s doc to make the **flycheck errors** buffer to be independent from other windows operations. One way to make the flycheck buffer work with ecb is use my configuration and open flycheck error buffer before activating `ECB`.


<a id="org6d3156f"></a>

# Format the code

There is no doubt that clang indeed is a powerful and flexible compiler frontend. The package `clang-format` makes use of the clang build in formatting facility to help us format the code. Actually you can run the command `clang-format` in shell, this package is a wrapper over the command that makes Emacs user's life much easier.


<a id="orgf8ee6e5"></a>

# Disassemble

There are many people who doesn't like C/C++. One of the many reasons is that you need to understand some low level properties of the language and sometimes you need to figure out how the compiler works. But from my point of view, it never hurts to know more about these black boxes. On Emacs, there is a convenient package to help you checkout what happens to your code during compilation and that is `disaster`. It works by compiling your file to object first and then disassemble the object file back into assembly code using system command `objdump`. You can call the function `disaster` using `M-x disaster` to do the trick. I prefer to bind it to `"C-c d a"`. Here is a screenshot after calling `disaster`:

![disaster]({{ site.url }}/assets/C|C++-Development-Environment-on-Emacs/disaster.png "Emacs IDE disassemble")
As you can see, `disaster` will display your disassembled code in a split window. Not only that, in the assembly buffer, `disaster` will jump to the line corresponding to where you place your cursor in C++ buffer and highlight that line. In this case, I place my cursor under `#include` directive and the corresponding line in assembly buffer got highlighted.

One thing to note about that is the function needs to compile your code first. So you can't do the trick to your "dirty file" which isn't ready for compilation. If the compilation process failed, it will just display a error message.


<a id="orgd738ec9"></a>

# Debugging

Many people stick with IDE for its full blown GUI debugging facility. Actually you can do the same with Emacs as long as you have GDB installed on your system. But debugging is a long and complex process for what ever tool you use. I think it's more suitable to discuss the subject along at other place and I have to admit that reading the whole documentation might be the best way to know about it. Here is some resources you can refer to:


<a id="orgb1b94c5"></a>

## Emacs build in support:

[GDB Graphical Interface](https://www.gnu.org/software/emacs/manual/html_node/emacs/GDB-Graphical-Interface.html#GDB-Graphical-Interface) A simple screenshot for it:

![gdb]({{ site.url }}/assets/C|C++-Development-Environment-on-Emacs/gdb.png "Emacs IDE debugging")
Please note that you can get much more than the stuffs in the screenshot. But it might take you some time to configure it.


<a id="org44c60c5"></a>

## RealGUD

`RealGUD` is a package that support multiple debuggers, you can refer to the official github repo via this [link](https://github.com/realgud/realgud/).


<a id="org4d0da4c"></a>

# Project

`Projectile` is a general project management tool. But for the purpose of using it for C/C++ development environment, we won't be using lots of its features. Like many other Emacs project aware packages, it makes use of the directory as the project hierarchy. In most of the time, projectile doesn't define projects by itself. Instead, if you are in a directory or sub-directory of which controlled by some VCS or build systems, it will recognize it as a project. The list of supported of supported systems is followed:

- git
- mercurial
- darcs
- bazaar
- lein
- maven
- sbt
- scons
- rebar
- bundler

You can define create a file named `.projectile` under your project root to control which sub-directors or files should be included in the project and vice verse.

To exclude sub-directors (files) from project, enter the name of it in `.projectile` like this:
```
-/log
-/tmp
-/vendor
-/public/uploads
-tmp
-*.rb
-*.yml
-models
```
Note that any other file (sub-directors) not in the above list will be included into the project.

You can do the opposite like:
```
+/src/foo
+/tests/foo
```
In this case, all the files (sub-directors) not in the list will be excluded.

To invoke it, do `M-x RET projectile-mode RET`

So, why do we need a separated project while we already have stuffs like `cmake-ide`? For one thing, if you have multiple projects at hand, a general management tool would be convenient. For another, it comes with a particular handy feature -- jumping between ".h" and ".cpp" files.

Here is some key bindings and the brief description:
```
C-c p 4 a 	Switch between files with the same name but different extensions in other window.
C-c p 4 f 	Jump to a project's file using completion and show it in another window.
C-c p s g 	Run grep on the files in the project.
C-c p e 	Shows a list of recently visited project files.
```
Other than these, if you were attracted by the concept of general project management, don't miss the [official documentation](http://projectile.readthedocs.io/en/latest/).

<a id="org2954831"></a>

# Summary

For Emacs, I doubt that there are many real experts. I can write some simple lisp code, have read part of the Emacs source and helped debugging some packages for others but I am still a rookie. However, as you might have noticed, hacking into it is always fun. For the purpose of this tutorial, you might need something more powerful and flexible or you might just want some casual coding. You can always add your own configuration or ignore some of the features provided by various packages. And most importantly, all the Emacs documentation can be accessed locally via `M-x info` or `C-h i`. And all packages I have mentioned have proper documentation, you can find them at their respective repositories.

<!--  LocalWords:  bundler rebar scons sbt lein darcs VCS cmake cpp
 -->
<!--  LocalWords:  Srefactor img RealGUD github IDE flycheck ecb
 -->
<!--  LocalWords:  eldoc
 -->
