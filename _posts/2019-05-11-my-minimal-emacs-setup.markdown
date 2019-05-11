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

We start with some basic info about myself and let emacs know who I am.

{% highlight elisp %}
(setq user-full-name "Sandeep Nambiar"
      user-mail-address "contact@sandeepnambiar.com")
{% endhighlight %}

I also like to set the garbage collection thresholds to reasonable modern values so that emacs does not decide to GC often.

{% highlight elisp %}
(setq gc-cons-threshold 50000000)
(setq large-file-warning-threshold 100000000)
{% endhighlight %}

Everything needs to be UTF-8

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

I like a clean emacs look so I get rid of the menu bar, the tool bar, the blinking cursor and the scroll bar.

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

I never use the startup emacs screen anymore so I disable that as well.

{% highlight elisp %}
(setq inhibit-startup-screen t)
{% endhighlight %}

I like to see the name of the file I am editing in the title bar, so lets set that up.

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

I use the font "Hack" which you can find at this [link](https://sourcefoundry.org/hack/) and set that to be the frame font.

{% highlight elisp %}
(set-frame-font "Hack 12" nil t)
{% endhighlight %}

One of the things I liked about Visual Studio Code editor was the default theme. Fortunately there's an emacs package for it.

{% highlight elisp %}
(use-package doom-themes
  :ensure t
  :config
  (load-theme 'doom-one t)
  (doom-themes-visual-bell-config))
{% endhighlight %}

I also like a cleaner mode line, especially with the powerline theme.

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

Emacs like to strew its backup and temporary files everywhere. Lets give them a home in the temporary file directory.

{% highlight elisp %}
(setq backup-directory-alist
      `((".*" . ,temporary-file-directory)))
(setq auto-save-file-name-transforms
      `((".*" ,temporary-file-directory t)))
{% endhighlight %}

## Ease of life

You can get fairly tired by typing a full "yes" or "no" to emacs' questions. So I replaced that with a simple "y" or "n"

{% highlight elisp %}
(fset 'yes-or-no-p 'y-or-n-p)
{% endhighlight %}

If I edit a file outside of emacs, I dont want to reload the file here.

{% highlight elisp %}
(global-auto-revert-mode t)
{% endhighlight %}

Tabs are 4 spaces.

{% highlight elisp %}
(setq-default tab-width 4
              indent-tabs-mode nil)
{% endhighlight %}

When I say kill a buffer, dont ask to confirm.

{% highlight elisp %}
(global-set-key (kbd "C-x k") 'kill-this-buffer)
{% endhighlight %}

And cleanup whitespaces in a file when I save.

{% highlight elisp %}
(add-hook 'before-save-hook 'whitespace-cleanup)
{% endhighlight %}

Highlighting of parens is handled by smartparens package.

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

I like the dwim expanding selection offered by expand region package.

{% highlight elisp %}
(use-package expand-region
  :ensure t
  :bind ("M-m" . er/expand-region))
{% endhighlight %}

Some useful defaults are provided by the crux package of prelude fame.

{% highlight elisp %}
(use-package crux
  :ensure t
  :bind
  ("C-k" . crux-smart-kill-line)
  ("C-c n" . crux-cleanup-buffer-or-region)
  ("C-c f" . crux-recentf-find-file)
  ("C-a" . crux-move-beginning-of-line))
{% endhighlight %}

Given the thousands of commands at your disposal, its only reasonable to have a package help you figure out the next keystroke.

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

I use company mode to provide completion and flycheck to do syntax checking. I enable them globally.

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

No project is begun without git and emacs community's integration with git through magit is unparalleled in its expanse and ease of use. Setting that up is just as easy.

{% highlight elisp %}
(use-package magit
  :bind (("C-M-g" . magit-status)))
{% endhighlight %}

Projectile is a project manager that lets you easily switch between files in a project.

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

Helm requires its own heading. It makes navigating emacs a much nicer experience overall.

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

I combine it with projectile to show project files through helm.

{% highlight elisp %}
(use-package helm-projectile
  :ensure t
  :config
  (helm-projectile-on))
{% endhighlight %}

## Daemon Mode

At the end I start the emacs server so that any new frames that I open don't have to load from scratch.

{% highlight elisp %}
(require 'server)
(if (not (server-running-p)) (server-start))
{% endhighlight %}

You can find my full emacs configuration at my [.emacs.d](https://github.com/gamedolphin/.emacs.d) github repository.