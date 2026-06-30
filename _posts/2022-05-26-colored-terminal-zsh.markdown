---
layout:     post
title:      "Colored Terminal with zsh"
subtitle:   "for the last versions of macOS"
date:       2022-05-26 01:00:00
author:     "Daniel Vela"
og_image: "/img/post-bg-17.jpg"
locale:     en
lang-ref:   colored-terminal-zsh
---

Last **macOS** versions have _zsh_ by default, instead _bash_. Better than try to use _bash_ again, is to use a new way to add color to Terminal for _zsh_.

Create a file called `.zshrc` in your `$HOME` path, with the following content (then restart terminal):

```sh
LS_COLORS='no=00;37:fi=00:di=00;33:ln=04;36:pi=40;33:so=01;35:bd=40;33;01:'
export LS_COLORS
export CLICOLOR=1
zstyle ':completion:*' list-colors ${(s.:.)LS_COLORS}
PROMPT='%F{green}%n@%m%k %B%F{cyan}%(4~|...|)%3~%F{white} %# %b%f%k'
export LSCOLORS=ExGxBxDxCxEgEdxbxgxcxd
export LSCOLORS
```

## Additional Doc
- [How Do I Change My ZSH Prompt Name](https://linuxhint.com/change-zsh-prompt-name/)
- [Zsh not recognizing ls colors](https://superuser.com/questions/700406/zsh-not-recognizing-ls-colors)
