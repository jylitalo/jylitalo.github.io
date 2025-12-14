---
title: "Bare repo based Dotfiles"
date: 2023-04-16T12:00:00+02:00
categories: ["index", "git"]
---
This week, I had opportunity to upgrade to new Macbooki at work. As always with new computer, it makes you think how you want to set it up. Dotfiles has been one area, where all my previous attempts have failed in a sense that they have fallen out of use. This time, I wanted to try something different and [Atlassian's tutorial](https://www.atlassian.com/git/tutorials/dotfiles) seemed to offer fresh perspective.
<!--more-->
To make it happen, I deleted my deprecated content from [dotfiles repo](https://github.com/jylitalo/dotfiles) and started to follow slightly altered version of Atlassian's instructions.

`git clone --bare git@github.com:jylitalo/dotfiles.git $HOME/.dotfiles`
`alias config='/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'`
`config config --local status.showUntrackedFiles no`
`echo "alias config='/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'" >> $HOME/.zprofile`
