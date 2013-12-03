---
layout: post
title: "Installing Sublime Text 3 on Fedora 19"
date: 2013-06-30 16:03
comments: true
categories: [Sublime Text,Fedora]

keywords: sublime text, fedora, package control, sublime text shortcuts, install, sublimetext, sublimetext3, gnome, gnome 3, sublime text, shortcuts
description: install sublime text on Fedora 19, install package control, useful shortcuts
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

The simplest method of installation is through the Sublime Text console. The console is accessed via the `ctrl+` shortcut or the View > Show Console menu. Once open, paste the following Python code for your version of Sublime Text into the console.

{% codeblock lang:python %}
	import urllib.request,os; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())
{% endcodeblock %}

[Source][3]

[Here's][4] the list plugins with compatibility indicator.

## Sublime Text 3 Useful Shortcuts

{% gist 5895595 %}


[1]: http://www.sublimetext.com/blog/articles/sublime-text-3-public-beta
[2]: http://www.sublimetext.com/3
[3]: https://sublime.wbond.net/installation#st3
[4]: https://sublime.wbond.net/browse
