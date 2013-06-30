---
layout: post
title: "Installing Sublime Text 3 on Fedora 19"
date: 2013-06-30 16:03
comments: true
categories: [Sublime Text,Fedora]

keywords: sublime text, fedora, package control
description: install sublime text on Fedora 19
---

To install sublime text on Fedora 19 (16, 17 and 18), download this gist

[Sublime Text 3][1] Beta is now open to non-registered users. [Download][2] the tarball (32 or 64 bits).

- Go to your Downloads folder: ```cd Downloads```
- Download or copy this GIST in your Downloads folder.
- Execute this script by ```chmod +x sublime3_fedora19.sh``` and then ```sudo ./sublime3_fedora19.sh sublime_text_3_build_3047_x64.tar.bz2```, ```sublime_text_3_build_3047_x64.tar.bz2``` is the downloaded tarball, change it if you download other version.
- Now your sublime text is installed and integrated to your gnome desktop.


<!-- more -->

{% gist 5895264 %}

## Sublime Text 3 Package Control

To enable Package control in Sublime Text 3:

- Go into Sublime Text Packages : ``` cd ~/.config/sublime-text-3/Packages/ ```
- Clone this repository: ```git clone git://github.com/wbond/sublime_package_control.git Package\ Control```
- Go into Package Control: ```cd Package\ Control```
- Checkout Python 3 branch: ```git checkout python3```

Restart Sublime Text 3 and you should have Package Control working.

[Source][3]

[Here's][4] the list of compatible plugins.

## Sublime Text 3 Useful Shortcuts

{% gist 5895595 %}


[1]: http://www.sublimetext.com/blog/articles/sublime-text-3-public-beta
[2]: http://www.sublimetext.com/3
[3]: http://wbond.net/sublime_packages/package_control/installation#ST3
[4]: https://github.com/wbond/sublime_package_control/wiki/Sublime-Text-3-Compatible-Packages
