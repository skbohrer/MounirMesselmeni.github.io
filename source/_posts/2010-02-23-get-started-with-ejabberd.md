---
layout:     post
title:      "Get started with ejabberd"
date:       2010-02-23 12:00:00
categories: [Ejabberd]
tags:       [ejabberd,erlang,xmpp]
published:  true
---

## Installation

Before installing ejabberd make sure you have Erlang environment set up. Run the following command and verify the output

    $ erl -version
    Erlang (ASYNC_THREADS,HIPE) (BEAM) emulator version 5.10.1

I used to install ejabberd with apt-get command, which is convenient but always several versions behind. That's why I switched to building from sources. All scripts below should be run under *ejabberd* user.

First, create a directory where all ejabberd versions will be installed, if it does not exist yet

    $ sudo mkdir /opt/ejabberd
    $ sudo chown -R ejabberd /opt/ejabberd

Download the sources

    $ git clone git@github.com:processone/ejabberd.git
    $ cd ejabberd

Assuming we want to install version 2.1.x

    $ git checkout 2.1.x
    $ cd src
    $ ./configure --prefix=/opt/ejabberd/ejabberd-2.1.12 --enable-user=ejabberd

To see all available options, run `./configure --help`.

    $ make
    $ make install
    $ cd /opt/ejabberd
    $ ln -s ejabberd-2.1.12 ejabberd

## Configuration

Ejabberd comes with reasonable default configuration. Only two lines of configuration need to be changed to make it work in your environment. Open */opt/ejabberd/ejabberd/etc/ejabberd/ejabberd.cfg* file and find *SERVED HOSTNAMES* section. By default Ejabberd listens to localhost only. Update this line with your machine's host names. I usually add the short name and the long name. You can find them by running `hostname -s` and `hostname -A` commands. Here is what I have in my config

{% codeblock /opt/ejabberd/ejabberd/etc/ejabberd/ejabberd.cfg lang:erlang %}
%%%'   SERVED HOSTNAMES
{hosts, ["localhost", "ubuntu", "jabber.ndpar.com"]}.
{% endcodeblock %}

The second thing we need to do is to configure admin user. Here is mine

{% codeblock /opt/ejabberd/ejabberd/etc/ejabberd/ejabberd.cfg lang:erlang %}
%%%'   ACCESS CONTROL LISTS
{acl, admin, {user, "andrey", "jabber.ndpar.com"}}.
{% endcodeblock %}

After these two changes are done, start the server

    $ cd /opt/ejabberd/ejabberd/sbin
    $ ./ejabberdctl start

## Registering users

The first (admin) user has to be registered in command line

    $ ./ejabberdctl register andrey jabber.ndpar.com ******

This user is the same as admin you configured in the previous section. If you go to [localhost:5280/admin](http://localhost:5280/admin) in the browser, you should be able to login with the same password you registered the user

{% img center /images/posts/ejabberd-web.png %}

Now you are ready to add newly created account to your Jabber client. In Adium, for example, go to *File* -> *Add Acount* -> *Jabber* and provide server hostname/IP, JID and password.

{% img center /images/posts/ejabberd-adium-1.png %}

{% img center /images/posts/ejabberd-adium-2.png %}

To really enjoy IM you need more users on your server. The best part here is that you can create new users just from your Jabber client. You can actually do many things from the client, and you don't need to ssh to the remote server and run command for that. Just go to *File* -> *your ejabberd account*, and chose whatever you need from the menu

{% img center /images/posts/ejabberd-admin-client.png %}
