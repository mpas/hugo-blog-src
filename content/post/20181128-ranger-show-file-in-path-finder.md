---
title: "Ranger - Show File in Path Finder"
tags: ["ranger", "pathfinder"]
date: "2018-11-28"
---

Ranger is a VIM-inspired filemanager for the console (https://ranger.github.io/) and can easily be installed by using
`brew install ranger`. When working in the terminal sometimes it is nice to open the files in the default `Finder` app
or use the excellent alternative called `Path Finder`. (https://cocoatech.com/#/)

To reveal your files in the Finder or Path Finder create a `commands.py` in `~/.config/ranger` and paste the following
code.

```python
from ranger.api.commands import Command

class show_files_in_path_finder(Command):
    """
    :show_files_in_path_finder

    Present selected files in finder
    """

    def execute(self):
        import subprocess
        files = ",".join(['"{0}" as POSIX file'.format(file.path) for file in self.fm.thistab.get_selection()])
        reveal_script = "tell application \"Path Finder\" to reveal {{{0}}}".format(files)
        activate_script = "tell application \"Path Finder\" to activate"
        script = "osascript -e '{0}' -e '{1}'".format(reveal_script, activate_script)
        self.fm.notify(script)
        subprocess.check_output(["osascript", "-e", reveal_script, "-e", activate_script])

class show_files_in_finder(Command):
    """
    :show_files_in_finder

    Present selected files in finder
    """

    def execute(self):
        import subprocess
        files = ",".join(['"{0}" as POSIX file'.format(file.path) for file in self.fm.thistab.get_selection()])
        reveal_script = "tell application \"Finder\" to reveal {{{0}}}".format(files)
        activate_script = "tell application \"Finder\" to set frontmost to true"
        script = "osascript -e '{0}' -e '{1}'".format(reveal_script, activate_script)
        self.fm.notify(script)
        subprocess.check_output(["osascript", "-e", reveal_script, "-e", activate_script])
```

Restart Ranger and now you can execute the commands :show_files_in_pathfinder or :show_files_in_finder.
