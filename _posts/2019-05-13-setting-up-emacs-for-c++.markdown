---
title: "Emacs setup for C++"
layout: post
date: 2019-05-13 09:00
image: /assets/images/emacslsp.png
headerImage: true
tag:
- emacs
- c++
star: true
category: blog
author: nambiar
description: Setting up emacs for C++ development
---

For C++ development, I use a combination of [emacs-lsp](https://github.com/emacs-lsp/lsp-mode) along with [ccls](https://github.com/MaskRay/ccls). Emacs-lsp is a client for [Language Server Protocol](https://microsoft.github.io/language-server-protocol/) which, in Microsoft's words:-
> Adding features like auto complete, go to definition, or documentation on hover for a programming language takes significant effort. Traditionally this work had to be repeated for each development tool, as each tool provides different APIs for implementing the same feature. A Language Server is meant to provide the language-specific smarts and communicate with development tools over a protocol that enables inter-process communication. The idea behind the Language Server Protocol (LSP) is to standardize the protocol for how such servers and development tools communicate. This way, a single Language Server can be re-used in multiple development tools, which in turn can support multiple languages with minimal effort.

ccls is a stand-alone server implementing the Language Server Protocol for C, C++, and Objective-C languages.

## Getting ccls

You can follow the instructions to [build](https://github.com/MaskRay/ccls/wiki/Build) and then [install](https://github.com/MaskRay/ccls/wiki/Install) ccls. TLDR: Make sure you have all the dependencies, clone ccls repository, download llvm binaries, build with cmake, install with cmake.

At the end of the process, you should have ccls in your path. If you don't want to install or cannot install ccls, just remember the path to the release binary that you get after the build process.

## Emacs-lsp

First lets ensure we have emacs-lsp installed. [lsp-ui](https://github.com/emacs-lsp/lsp-ui) provides additional ui flair along with flycheck integration. [company-lsp](https://github.com/tigersoldier/company-lsp) integrates company with lsp.

{% highlight elisp %}
(use-package lsp-mode :commands lsp :ensure t)
(use-package lsp-ui :commands lsp-ui-mode :ensure t)
(use-package company-lsp
  :ensure t
  :commands company-lsp
  :config (push 'company-lsp company-backends)) ;; add company-lsp as a backend
{% endhighlight %}

Then we can setup ccls. It uses flymake for its syntax checking. We can disable that to let flycheck do its thing.

{% highlight elisp %}
(use-package ccls
  :ensure t
  :config
  (setq ccls-executable "ccls")
  (setq lsp-prefer-flymake nil)
  (setq-default flycheck-disabled-checkers '(c/c++-clang c/c++-cppcheck c/c++-gcc))
  :hook ((c-mode c++-mode objc-mode) .
         (lambda () (require 'ccls) (lsp))))
{% endhighlight %}

That is it. When you open a new c++ file, ccls should ask you for the root of the project and make sense of how your project is structured more or less on its own. You can also configure a project to help ccls by following the instructions [here](https://github.com/MaskRay/ccls/wiki/Project-Setup).

Happy Coding!
