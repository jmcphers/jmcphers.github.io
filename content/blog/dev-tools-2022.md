---
title: "Favorite Dev Tools, 2022"
date: 2022-09-19
---

There's an overarching theme here, and it's this: *don't settle*. If you are a software engineer, you're using a lot of tools in your terminal that mostly haven't been updated since the 80s. Modernized alternatives to many of these tools exist; with very little effort, you can remove a great deal of the typing and friction from everyday tasks. 

# Terminal Tools

## z
`z` makes changing directories less of a typing chore. It learns which directories you use often and lets you jump into them by typing just a few characters. I use it constantly and consider it the most time-saving tool I use. 

https://github.com/rupa/z


## fzf
A general purpose fuzzy finder. I have mine integrated with `zsh` as a tool for searching through shell history. I also use it to fuzzy find files deeply nested in directories -- like Command-P search in VS Code, but in the terminal.

https://github.com/junegunn/fzf

## bat
Bat is a better `cat`, featuring syntax highlighting, line numbers, and a built in pager (so you don't have to pipe it to `less`). It's also smart enough to know when it's being used in a pipe, so it can also be used as a drop-in replacement for `cat` in almost all contexts.

https://github.com/sharkdp/bat

## exa
Exa is a better `ls`, with colorization, human-readable file sizes, git status, etc. 

https://the.exa.website/

## tmux
tmux is a Terminal Multiplexer. It lets you run a whole bunch of shells inside a single terminal window. I find it's a lot more sane and organized than having 37 different terminal windows open. It also keeps stuff running after the terminal is closed; you can just reattach to your session later (even on another machine).

https://github.com/tmux/tmux

## ripgrep
ripgrep is a replacement for the tools `grep`, `fgrep`, and `git grep`. It is incredibly fast and accurate. 

https://github.com/BurntSushi/ripgrep

## tig
tig is a curses-based interface to `git`. I use it instead of `git status` most of the time. It has interactive tools for staging/unstaging/committing chunks and files, and they're extremely ergonomic. 

https://jonas.github.io/tig/

## neovim
Neovim is a modern fork of Vim. It's extensible with Lua and has a built in Language Server Protocol client, so it can give you an IDE-like environment along with all the power of Vi/Vim under the hood.

https://neovim.io/

## mosh
If you're working on a remote host in an interactive terminal, you want mosh. It's an order of magnitude better than a plain ssh connection -- your typing shows up right away, for example, and it can survive network switches and auto-resume after connection drops. 

https://mosh.org/

## prezto
Prezto is a plug-in framework for `zsh`. It has similarities to `oh-my-zsh`, in that it gives you a batteries-included shell with lots of neat features that are easy to turn on. I found it to be simpler, faster, and more modular than `oh-my-zsh`, despite the latter's greater popularity.

https://github.com/sorin-ionescu/prezto

## diff-so-fancy
This is a Git pager that makes diffs much easier to read (and prettier). It can be configured as the default Git pager, so that you don't even have to remember to run it -- after installation, commands you use all the time like `git diff` will have beautiful output.

https://github.com/so-fancy/diff-so-fancy

## fd
fd is a replacement for `find` that makes it fast and easy to locate files in a tree and do things with them. I use it all the time instead of `find . -iname ...` to look up paths and locations.

https://github.com/sharkdp/fd

## pass
Password Store (or just "pass") is a password manager that lives in your shell and is implemented almost entirely in plain `bash`. It uses `gnupg` for encryption and `git` for synchronization, so it can be installed pretty much anywhere. It's become so popular that people have made GUIs for it.

https://www.passwordstore.org/

# GUI Tools

## Obsidian
Obsidian is a Personal Knowledge Management system. I use it to write down what I plan to do and what I've done each day, and to keep a kind of lab journal while debugging or spelunking. I also use it to store bookmarks, track my reading, and write drafts of longer-form Markdown content. My favorite thing about it is that the notes are just Markdown files stored in Git, so they are easy to read and search using other tools.

https://obsidian.md/

## Alfred
Alfred is a macOS app launcher that's much faster than Spotlight, but it has a wealth of other tools that make keyboard driven workflows possible. I use it as a snippet expander for text I type frequently, like my Zoom room ID, as a clipboard history manager, and as a way to quickly dump tasks and ideas for later processing.

https://www.alfredapp.com/

## WezTerm
WezTerm is a replacement for Terminal.app. It adds GPU rendering, fancy fonts, and a lot of other quality of life improvements. Developers just love writing new terminal apps (they are secondly only to text editors); I formerly used iTerm2 and Alacritty. 

https://github.com/wez/wezterm

## warp
Warp is a unique take on modernizing the terminal. It adds predictive command completion, a cell/block based interface, and an AI command generator that is remarkably good. 

https://www.warp.dev/

## Rectangle
Rectangle makes it really easy to snap and align windows in macOS; wonderful for making two windows exactly side by side or moving a window to where you want it with a keyboard gesture. 

https://rectangleapp.com/


