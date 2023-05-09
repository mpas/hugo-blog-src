+++
title = "Capture link from Mac app into org-mode document"
author = ["Marco Pas"]
date = 2021-03-30T12:54:00+02:00
tags = ["orgmode", "emacs", "elisp"]
categories = ["development"] 
draft = false
+++

To quickly grab a link from an open Mac app and use this link into an org-mode document use the following package:

-   [Grab links from open Mac applications](https://orgmode.org/worg/org-contrib/org-mac-link.html)

An optional keybind as follows:

```emacs-lisp
(add-hook 'org-mode-hook (lambda ()
                           (define-key org-mode-map (kbd "C-c g") 'org-mac-grab-link)))
```

This enables you to quickly grab a link using `C-c g`
