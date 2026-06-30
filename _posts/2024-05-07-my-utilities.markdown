---
layout:     post
title:      "My utilities"
subtitle:   "Some command line utilities I developed"
date:       2024-05-07 06:00:00
author:     "Daniel Vela"
og_image: "/img/post-bg-06.jpg"
locale:     en
lang-ref:   my-utilities
---

# Utilities

## Command line

### bolt_panel.sh

- [github(danieltvela/bin)](https://github.com/danieltvela/bin)

Opens several sessions in different Terminals, showing cpu temperature, nvidia-smi, htop, and opens a new terminal.

### flushdns

- [github(danieltvela/bin)](https://github.com/danieltvela/bin)

Refresh DNS cache for macOS. Use with sudo.

Usage: `$ sudo flushdns`

## git

### git-alldiff

- [github(danieltvela/bin)](https://github.com/danieltvela/bin)

Execute in a git repository to show all current diferences

### git-lazypush

- [github(danieltvela/bin)](https://github.com/danieltvela/bin)

This script add all changes, commit and push all at once.

Usage: `$ git-lazypush "Your comment"

### git-remote-diff

- [github(danieltvela/bin)](https://github.com/danieltvela/bin)

Shows all differences between current branch and remote branch

### gitlab-backup

- [github(danieltvela/bin)](https://github.com/danieltvela/bin)

Makes a backup of a gitlab server remotely and copies the just generated backup file locally.

Usage: `$ gitlab-backup gitlab.local ./gitlab-backups`

### remotize

- [github(danieltvela/bin)](https://github.com/danieltvela/bin)

Creates a remote bare git repository in a remote server, and configure the local git repository to use it as a remote

Execute command from parent directory of the git repo:

`$ remotize project server.local`

## Ruby

### BenderRuby

- [gitlab(danieltvela/BenderRuby)](https://github.com/danieltvela/BenderRuby)

Ruby template project to use ActiveRecord and migrations without the rest of Rails

## SQL

### sql-api

- [github(danieltvela/sql-api)](https://github.com/danieltvela/sql-api)

This is an experimental API. Queries are received in SQL, Sqlite3 interpret them, and return the response encoded in json (or plain if set this way).
