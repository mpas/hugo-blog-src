---
title: "Get pretty org-bullets in Doom Emacs"
date: 2020-10-16T07:09:45+02:00
draft: false
tags: ["emacs", "doom", "org"]
---

When installing [Doom Emacs][doom-emacs] and using org-mode the defaults bullets are `*`. In order to get some fancy bullets the following steps need to be taken.

1. Add the org-mode `+pretty` flag to your org settings in `init.el` To read more on the available flags check the [org-mode Doom Emacs module `lang/org`[doom-emacs-org-mode-flags][doom-emacs-org-mode-flags]

```lisp
:lang
(org +pretty ) ; organize your plain life in plain text
```

2. Set the bullets you want to use in your `config.el`

```lisp
(setq
    org-superstar-headline-bullets-list '("⁖" "◉" "○" "✸" "✿")
)
```

3. Reload your Doom configuration and you should be fine :)

[doom-emacs]: https://github.com/hlissner/doom-emacs
[doom-emacs-org-mode-flags]: https://github.com/hlissner/doom-emacs/tree/develop/modules/lang/org#module-flags
