---
title: "My Minimal Emacs Setup"
layout: post
date: 2019-05-11 09:00
image: /assets/images/markdown.jpg
headerImage: false
tag:
- emacs
star: true
category: blog
author: nambiar
description: A tour of my emacs setup
---

## Basic Setup

I begin by telling Emacs what my name and email address is. This is used by tools like git when creating commits.

{% highlight elisp %}
(setq user-full-name "Sandeep Nambiar"
      user-mail-address "contact@sandeepnambiar.com")
{% endhighlight %}

Garbage collection in emacs is quite simple. Every time memory crosses a certain threshold, it garbage collects. But the default value for this threshold is quite low by modern standards, just 800KB. Most extensions of emacs use considerably more memory than that and this low threshold makes emacs quite slow when using them. I set it something more reasonable like 50MB. Another low threshold is hit when opening files with sizes that are common nowadays and Emacs warns you about being wary of opening such a large file. I increase that limit too. I have RAM to spare.

{% highlight elisp %}
(setq gc-cons-threshold 50000000)
(setq large-file-warning-threshold 100000000)
{% endhighlight %}

Everything needs to be UTF-8. I have to be careful when opening files from sources other than my systems but in general, UTF-8 is preferred everywhere.

{% highlight elisp %}
(prefer-coding-system 'utf-8)
(set-default-coding-systems 'utf-8)
(set-terminal-coding-system 'utf-8)
(set-keyboard-coding-system 'utf-8)
{% endhighlight %}

## Setup package manager and use-package

Emacs becomes truly useful with its package manager and thousands of freely available packages from its wonderful community. Use-package is a utility library that lets us declaratively setup the packages we will be using.

{% highlight elisp %}
(require 'package)
(setq package-enable-at-startup nil)
(add-to-list 'package-archives '("melpa" . "http://melpa.org/packages/"))
(add-to-list 'package-archives '("gnu" . "http://elpa.gnu.org/packages/"))
(package-initialize)

(unless (package-installed-p 'use-package)
  (package-refresh-contents)
  (package-install 'use-package))

(eval-when-compile
  (require 'use-package))
{% endhighlight %}

## Visual Setup

Visual defaults for Emacs have a menu bar, scroll bars on all buffers and a toolbar as well. But most functionality can and should be reached through just keyboard commands, which means most of this is just visual clutter. I like a clean Emacs look so I get rid of the menu bar, the tool bar, the blinking cursor and the scroll bar.

{% highlight elisp %}
(menu-bar-mode -1)
(toggle-scroll-bar -1)
(tool-bar-mode -1)
(blink-cursor-mode -1)
{% endhighlight %}

Next I enable a bunch of visual helpers like line numbers, current line highlighting, column number and file size in the mode line.

{% highlight elisp %}
(global-hl-line-mode +1)
(line-number-mode +1)
(global-display-line-numbers-mode 1)
(column-number-mode t)
(size-indication-mode t)
{% endhighlight %}

I never use the startup emacs screen anymore so I disable that as well. This leads Emacs to load up with the *scratch* buffer open.

{% highlight elisp %}
(setq inhibit-startup-screen t)
{% endhighlight %}

The model-line has the current file's name but the path to the current file is still a few keystrokes away. The title bar can be used to show the full path of the current file, so lets set that up.

{% highlight elisp %}
(setq frame-title-format
      '((:eval (if (buffer-file-name)
       (abbreviate-file-name (buffer-file-name))
       "%b"))))
{% endhighlight %}

Default scrolling has some wierd quirks in emacs, so I set margins that trigger scroll to zero and preserve screen position when jumping around.

{% highlight elisp %}
(setq scroll-margin 0
      scroll-conservatively 100000
      scroll-preserve-screen-position 1)
{% endhighlight %}

I use the font "Hack" which you can find at this [link](https://sourcefoundry.org/hack/) and set that to be the frame font. I am still trying to figure out a way to setup font size that works on different screen resolutions but for now it is set to 12.

{% highlight elisp %}
(set-frame-font "Hack 12" nil t)
{% endhighlight %}

One of the things I liked about Visual Studio Code editor was the default theme. Fortunately there's an emacs package for it. Doom-one is my go to modern minimal theme. We also disable the default warning audio bell and replace it with a visual "bell" where only the mode-line flashes to warn you about something.

{% highlight elisp %}
(use-package doom-themes
  :ensure t
  :config
  (load-theme 'doom-one t)
  (doom-themes-visual-bell-config))
{% endhighlight %}

Smart mode line is a sexy mode line for Emacs. I also like the powerline theme for the mode line. The other option is Spaceline, the mode line from spacemacs, but I think it is more work to set that up.

{% highlight elisp %}
(use-package smart-mode-line-powerline-theme
  :ensure t)

(use-package smart-mode-line
  :ensure t
  :config
  (setq sml/theme 'powerline)
  (add-hook 'after-init-hook 'sml/setup))
{% endhighlight %}

## Backups

Emacs likes to strew its backup and temporary files everywhere. Lets give them a home in the temporary file directory.

{% highlight elisp %}
(setq backup-directory-alist
      `((".*" . ,temporary-file-directory)))
(setq auto-save-file-name-transforms
      `((".*" ,temporary-file-directory t)))
{% endhighlight %}

## Ease of life

You can get fairly tired by typing a full "yes" or "no" to Emacs' questions. So I replaced that with a simple "y" or "n"

{% highlight elisp %}
(fset 'yes-or-no-p 'y-or-n-p)
{% endhighlight %}

If I edit a file outside of emacs, the default setting is for Emacs to ask you to reload the file manually. I task Emacs to reload the file automatically.

{% highlight elisp %}
(global-auto-revert-mode t)
{% endhighlight %}

Tabs are 4 spaces.

{% highlight elisp %}
(setq-default tab-width 4
              indent-tabs-mode nil)
{% endhighlight %}

When I say kill a buffer (close a file), Emacs asks which one. I'd rather it kill the current buffer I am in.

{% highlight elisp %}
(global-set-key (kbd "C-x k") 'kill-this-buffer)
{% endhighlight %}

Accidental whitespaces are annoying. And I let Emacs know it. So it cleans up behind me when I save a file.

{% highlight elisp %}
(add-hook 'before-save-hook 'whitespace-cleanup)
{% endhighlight %}

Diminish lets you hide minor modes from showing in the mode line, keeping it minimal.

{% highlight elisp %}
(use-package diminish
  :ensure t)
{% endhighlight %}

Smartparens is a minor-mode for dealing with paren pairs in Emacs. It inserts them in pairs, deletes them in pairs, highlights them and generally makes life easier when dealing with braces, quotes and the like.

{% highlight elisp %}
(use-package smartparens
  :ensure t
  :diminish smartparens-mode
  :config
  (progn
    (require 'smartparens-config)
    (smartparens-global-mode 1)
    (show-paren-mode t)))
{% endhighlight %}

Selecting a region becomes smarter with expand region which keeps selecting an increasing region based on dwim syntax.

{% highlight elisp %}
(use-package expand-region
  :ensure t
  :bind ("M-m" . er/expand-region))
{% endhighlight %}

Some useful defaults are provided by the crux package of [Prelude](https://github.com/bbatsov/prelude) fame. "C-k" now kills a line if nothing is selected. "C-a" now toggles between first letter on the line, or beginning of the line.

{% highlight elisp %}
(use-package crux
  :ensure t
  :bind
  ("C-k" . crux-smart-kill-line)
  ("C-c n" . crux-cleanup-buffer-or-region)
  ("C-c f" . crux-recentf-find-file)
  ("C-a" . crux-move-beginning-of-line))
{% endhighlight %}

Given the thousands of commands at your disposal, its only reasonable to have a package help you figure out the next keystroke. which-key package opens a small buffer at the bottom with suggestions for next keystroke and possible commands that are available.

{% highlight elisp %}
(use-package which-key
  :ensure t
  :diminish which-key-mode
  :config
  (which-key-mode +1))
{% endhighlight %}

I can jump around in the current visual field using avy's go to char.

{% highlight elisp %}
(use-package avy
  :ensure t
  :bind
  ("C-=" . avy-goto-char)
  :config
  (setq avy-background t))
{% endhighlight %}

## Autocomplete and syntax checking

I use company mode to provide completion and flycheck to do syntax checking and enable them globally.

{% highlight elisp %}
(use-package company
  :ensure t
  :diminish company-mode
  :config
  (add-hook 'after-init-hook #'global-company-mode))

(use-package flycheck
  :ensure t
  :diminish flycheck-mode
  :config
  (add-hook 'after-init-hook #'global-flycheck-mode))
{% endhighlight %}

## Project setup

No project is begun without git and emacs community's integration with git through magit is unparalleled in its expanse and ease of use. Setting that up is just as easy. "C-M-g" now triggers a git status buffer for the current file's repository.

{% highlight elisp %}
(use-package magit
  :bind (("C-M-g" . magit-status)))
{% endhighlight %}

Projectile is a project manager that lets you easily switch between files in a project and seamlessly between projects as well. I use it with helm which I set up below.

{% highlight elisp %}
(use-package projectile
  :ensure t
  :diminish projectile-mode
  :bind
  (("C-c p f" . helm-projectile-find-file)
   ("C-c p p" . helm-projectile-switch-project)
   ("C-c p s" . projectile-save-project-buffers))
  :config
  (projectile-mode +1)
)
{% endhighlight %}

## Helm

Helm requires its own heading. It is a dwim fuzzy completion framework for Emacs and makes navigating Emacs a much nicer experience overall. I like to setup Helm to be a comfortable 20 pts in height and bind the most frequent Emacs commands like "M-x" with the helm equivalents.

{% highlight elisp %}
(use-package helm
  :ensure t
  :defer 2
  :bind
  ("M-x" . helm-M-x)
  ("C-x C-f" . helm-find-files)
  ("M-y" . helm-show-kill-ring)
  ("C-x b" . helm-mini)
  :config
  (require 'helm-config)
  (helm-mode 1)
  (setq helm-split-window-inside-p t
    helm-move-to-line-cycle-in-source t)
  (setq helm-autoresize-max-height 0)
  (setq helm-autoresize-min-height 20)
  (helm-autoresize-mode 1)
  (define-key helm-map (kbd "<tab>") 'helm-execute-persistent-action) ; rebind tab to run persistent action
  (define-key helm-map (kbd "C-i") 'helm-execute-persistent-action) ; make TAB work in terminal
  (define-key helm-map (kbd "C-z")  'helm-select-action) ; list actions using C-z
  )
{% endhighlight %}

I combine it with projectile to show project files through a helm fuzzy find interface.

{% highlight elisp %}
(use-package helm-projectile
  :ensure t
  :config
  (helm-projectile-on))
{% endhighlight %}

## Daemon Mode

At the end I start the emacs server so that any new frames that I open don't have to load the configuration from scratch.

{% highlight elisp %}
(require 'server)
(if (not (server-running-p)) (server-start))
{% endhighlight %}

You can find my full emacs configuration at my [github](https://github.com/gamedolphin/.emacs.d) repository.