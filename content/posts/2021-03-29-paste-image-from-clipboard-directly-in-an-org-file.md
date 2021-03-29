+++
title = "Paste image from clipboard directly in an org file"
author = ["Marco Pas"]
date = 2021-03-29T18:55:00+02:00
tags = ["emacs", "orgmode", "elisp", "screenshot", "image"]
draft = false
+++

Personally I really like using org-mode files for creating documentation and presentations.
When working with screenshots my normal workflows always has been.

1.  Capture screenshot to clipboard
2.  Save the screenshot to a relative `./images` folder and give it a descriptive name.
3.  Manually create link to the screenshot

This works ok but after a while you get the feel that the whole process can be automated.

The following function can help in automating the whole process.

1.  Capture screenshot
2.  In your org-document paste using C-M-y (or call `my/insert-clipboard-image`) and automatically add the screenshot to an =./images- directory and insert the link to the image.

<!--listend-->

```emacs-lisp
;; Overview
;; --------
;; Inserts an image from the clipboard by prompting the user for a filename.
;; Default extension for the pasted filename is .png

;; A ./images directory will be created relative to the current Org-mode document to store the images.

;; The default name format of the pasted image is:
;; filename: <yyyymmdd>_<hhmmss>_-_<image-filename>.png

;; Important
;; --------
;; This function depends on 'pngpaste' to paste the clipboard image
;; -> $ brew install pngpaste

;; Basic Customization
;; -------------------
;; Include the current Org-mode header as part of the image name.
;; (setq my/insert-clipboard-image-use-headername t)
;; filename: <yyyymmdd>_<hhmmss>_-_<headername>_-_<image-filename>.png

;; Include the buffername as part of the image name.
;; (setq my/insert-clipboard-image-use-buffername t)
;; filename: <yyyymmdd>_<hhmmss>_-_<buffername>_-_<image-filename>.png

;; Full name format
;; filename: <yyyymmdd>_<hhmmss>_-_<buffername>_-_<headername>_-_<image-filename>.png
(defun my/insert-clipboard-image (filename)
  "Inserts an image from the clipboard"
  (interactive "sFilename to paste: ")
  (let ((file
         (concat
          (file-name-directory buffer-file-name)
          "images/"
          (format-time-string "%Y%m%d_%H%M%S_-_")
          (if (bound-and-true-p my/insert-clipboard-image-use-buffername)
              (concat (s-replace "-" "_"
                                 (downcase (file-name-sans-extension (buffer-name)))) "_-_")
            "")
          (if (bound-and-true-p my/insert-clipboard-image-use-headername)
              (concat (s-replace " " "_" (downcase (nth 4 (org-heading-components)))) "_-_")
            "")
          filename ".png")))

    ;; create images directory
    (unless (file-exists-p (file-name-directory file))
      (make-directory (file-name-directory file)))

    ;; paste file from clipboard
    (shell-command (concat "pngpaste " file))
    (insert (concat "[[./images/" (file-name-nondirectory file) "]]"))))

(map! :desc "Insert clipboard image"
      :n "C-M-y" 'my/insert-clipboard-image)
```

A nice setting in org-mode that also helps when viewing large screenshots is the following:

This display the taken screenshot in a acceptable format in your org-mode file.

```emacs-lisp
(after! org
  (setq org-image-actual-width (/ (display-pixel-width) 2)))
```
