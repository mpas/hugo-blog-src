+++
title = "Get pretty org-bullets in Doom Emacs"
author = ["Marco Pas"]
date = 2020-10-16T20:21:00+02:00
tags = ["emacs", "doom", "org", "orgmode"]
draft = false
+++

When installing [Doom Emacs](https://github.com/hlissner/doom-emacs) and using org-mode the defaults bullets are \`\*\`. In order to get some fancy bullets the following steps need to be taken.

1.  Add the org-mode `+pretty` flag to your org settings in `init.el` To read more on the available flags check the [org-mode Doom Emacs module \`lang/org\`](https://github.com/hlissner/doom-emacs/tree/develop/modules/lang/org#module-flags)

<!--listend-->

```emacs-lisp
:lang
(org +pretty ) ; organize your plain life in plain text
```

<ol class="org-ol">
<li value="2">Set the bullets you want to use in your `config.el`</li>
</ol>

```emacs-lisp
(setq
    org-superstar-headline-bullets-list '("⁖" "◉" "○" "✸" "✿")
)
```

<ol class="org-ol">
<li value="3">Reload your Doom configuration and you should be fine :)</li>
</ol>
