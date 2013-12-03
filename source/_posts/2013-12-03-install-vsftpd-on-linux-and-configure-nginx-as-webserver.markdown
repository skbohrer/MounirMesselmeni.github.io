---
layout: post
title: "Install vsftpd on Linux and configure nginx as webserver"
date: 2013-12-03 00:49
comments: true
categories: [Linux, Nginx]
keywords: Linux, Fedora, redhat, amazon linux, amazon linux ami, vsftp, vsftpd, nginx, web server, ftp, html, debian, ubuntu, amazon, aws, ec2, ami
description: How to install an ftp server on linux using Vsftpd and dispose uploaded content on the web via nginx. Using a redhat based distribution like fedora, centos, amazon linux ami, redhat or others ditribution like debian, ubuntu, ...
---

In this tutorial we will create an FTP server in Redhat based distribution(Fedora, CentOS, Amazon Linux AMI, Redhat), this tutorial also can be applied for others distibution like Ubuntu or Debian, just change some configuration file location and some commands syntax.

We will install [VSFTPD][1] and [Nginx][2] to dispose uploaded data via FTP with Nginx.

<!-- more -->

### VSFTPD Installation

First of all, let's install vsftpd, an awesome FTP server very simple to use.

{% codeblock lang:sh %}

sudo yum install vsftpd

{% endcodeblock %}

Then, let's copy this configuration into `/etc/vsftpd/vsftpd.conf`

{% codeblock %}

anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=NO
xferlog_file=/var/log/vsftpd/vsftpd.log
xferlog_std_format=YES
ftpd_banner=Welcome to FTP service
chroot_local_user=YES
listen=YES
pam_service_name=vsftpd
tcp_wrappers=YES
userlist_file=/etc/vsftpd/vsftpd.userlist
userlist_enable=YES
userlist_deny=NO
pasv_max_port=51000
pasv_min_port=50000
port_enable=YES
pasv_enable=YES
pasv_address=YOUR_SERVER_URL
pasv_addr_resolve=YES
check_shell=NO
passwd_chroot_enable=YES

{% endcodeblock %}

As you see, in this configuration we had edited this settings:

- Disabling anonymous connections: `anonymous_enable=NO`
- Creating a custom log folder: `/var/log/vsftpd/vsftpd.log`
- Activating the custom log: `xferlog_std_format=YES`
- Adding a custom welcome message to our FTP Server: `ftpd_banner=Welcome to FTP service`
- Enabling local users: `userlist_enable=YES`
- Specifying users list: `userlist_file=/etc/vsftpd/vsftpd.userlist`
- Enabling users list: `userlist_enable=YES`
- Specifying users list as authorized list: `userlist_deny=NO`
- The ftpuser should be able to write data: `write_enable=YES`
- Set umask to 022 to make sure that all the files (644) and folders (755) you upload get the proper permissions: `local_umask=022`
- Configure vsftpd to use ports 50000-51000 for passive data transfers
- Change `YOUR_SERVER_URL` with the IP or domain name server, if you use IP you can change `pasv_addr_resolve` to `NO`

Next, we should create the folder where data should be, make this folder the home directory of the ftp user we will create. And define a password for this user.

{% codeblock lang:sh %}

sudo mkdir -p /var/log/vsftpd/
sudo mkdir -p /var/www/ftp
sudo useradd -d /var/www/ftp/ -s /sbin/nologin ftp_user
sudo passwd ftp_user

{% endcodeblock %}

Now, we add this user in the authorized users list, create a group named ftpusers and affect it to the ftp_user and make ftp_user own the ftp directory.

{% codeblock lang:sh %}

sudo echo "ftp_user" > /etc/vsftpd/vsftpd.userlist
sudo groupadd ftpusers
sudo usermod -G ftpusers ftp_user
sudo chown ftp_user /var/www/ftp/ -R

{% endcodeblock %}

Our Vsftpd server need to be restarted, we can also add it to start on boot with `chkconfig`, just update your firewall configuration to add the port 21/tcp and port range 50000-51000/tcp.

{% codeblock lang:sh %}

sudo chkconfig vsftpd on
sudo service vsftpd restart

{% endcodeblock %}

You can test your server with ftp command or FileZilla client.

### Nginx Installation

Let's install and configure Nginx to dispose our uploaded FTP data (HTML, Images, ...).

{% codeblock lang:sh %}

sudo yum install vsftpd
sudo chkconfig nginx --add
sudo chkconfig nginx on --level 235

{% endcodeblock %}

Now, copy the following configuration into `/etc/nginx/conf.d/ftp.conf` and start nginx service `sudo service nginx start`

{% codeblock %}
server {
  listen        80;
  server_name   domain;

  location /{
    autoindex on;
    root  /var/www/ftp/;
  }
}
{% endcodeblock %}

Finally, don't forget to add port 80 rule to your firewall configuration.

That's all !

[1]: https://security.appspot.com/vsftpd.html
[2]: http://nginx.com/