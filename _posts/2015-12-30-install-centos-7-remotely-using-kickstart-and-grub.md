---
layout: post
title: Install CentOS 7 remotely using Kickstart and GRUB
tags: [linux]
---

This guide assumes the target host is already running CentOS (a derivate of
Red Hat Enterprise Linux) or at least running the GRUB boot loader and that you
have root access to this host.

<!--more-->

## What's this all about?

I'm going to install CentOS 7 onto a machine which I do not have physical access
to. In order to achieve this, I'm going to need a Kickstart file, some files
from the CentOS 7 installation and create a custom GRUB boot loader entry.


## The Kickstart file

A Kickstart file will automate the whole installation process. The Red Hat 7
Enterprise documentation does a good job explaining:

> Red Hat Enterprise Linux 7 offers a way to partially or fully automate the installation process using a Kickstart file. Kickstart files contain answers to all questions normally asked by the installation program, such as what time zone do you want the system to use, how should the drives be partitioned or which packages should be installed. Providing a prepared Kickstart file at the beginning of the installation therefore allows you to perform the entire installation (or parts of it) automatically, without need for any intervention from the user. This is especially useful when deploying Red Hat Enterprise Linux on a large number of systems at once.

<cite>[RedHat 7 Enterprise documentation](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/chap-kickstart-installations.html)</cite>

A Kickstart file is generated in `/root` after a successful
installation of CentOS. You can use this as a start to create your custom
Kickstart file. Please take a minute or two to read through the [Kickstart How-To](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/sect-kickstart-howto.html) for more information on commands and options.

As an example, here's a Kickstart file which was created automatically when
installing CentOS 7. However, we'll need to make some changes to it so that it
will work when remotely installing CentOS 7.

First, we'll have to change the installation media from "cdrom" to "url". I'm
using one of the [mirrors](https://www.centos.org/download/mirrors/) available:

{% highlight bash %}
# Use CDROM installation media
#cdrom

# Use network installation
url --url="http://mirror.zetup.net/CentOS/7/os/x86_64/"
{% endhighlight %}

We'll also have to tell the installation to clear out any previous partitions
on "sda", the primary disk:

{% highlight bash %}
# Partition clearing information
#clearpart --none --initlabel
clearpart --all --drives=sda
{% endhighlight %}

Since we want the machine to automatically finish the installation, we will
have to tell it to reboot after completed installation:

{% highlight bash %}
# Reboot after installation?
reboot
{% endhighlight %}

It's possible that we won't know how to access the machine remotely after the
installation if we don't specify e.g. a static IP address. Here's how we could
do that:

{% highlight bash %}
# Network information
#network  --bootproto=dhcp --device=eth0 --ipv6=auto --activate
network --bootproto=static --device=eth0 --gateway=10.0.0.1 --ip=10.0.0.100 --nameserver=8.8.8.8 --netmask=255.255.255.0 --ipv6=auto --activate
network  --hostname=mymachine
{% endhighlight %}

Please review all options in the Kickstart file. There are additional options
which I will not cover here:

* [Kickstart options](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/sect-kickstart-syntax.html#sect-kickstart-commands) - A list of all commands and options
* [%pre](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/sect-kickstart-syntax.html#sect-kickstart-preinstall) - Pre-installation scripts
* [%post](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/sect-kickstart-syntax.html#sect-kickstart-postinstall) - Post-installation scripts
* [%addon](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/sect-kickstart-syntax.html#sect-kickstart-addon) - Add-ons for Anaconda which expand the functionality of the installer
* [%packages](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/sect-kickstart-syntax.html#sect-kickstart-packages) - Software packages to install


## Verify the Kickstart file

You can make sure your Kickstart file is valid by using `ksvalidator`:

Install ksvalidator:

    yum install pykickstart

Run ksvalidator on your Kickstart file:

    ksvalidator /path/to/anaconda-ks.cfg

Please note: ksvalidator will not attempt to validate the `%pre`, `%post` and `%packages` sections of the Kickstart file.


## Make the Kickstart file available on a web server

During the installation phase, CentOS will attempt to read the Kickstart file
from somewhere. I'm serving it using [a basic web server](http://fredrikaverpil.github.io/2015/12/28/python-web-server/).


## Download vmlinuz and initrd.img

Download vmlinuz and initrd.img from the desired CentOS version you wish to install
and place them in `/boot`. For example:

{% highlight bash %}
curl -o /boot/vmlinuz http://mirror.zetup.net/CentOS/7/os/x86_64/isolinux/vmlinuz
curl -o /boot/initrd.img http://mirror.zetup.net/CentOS/7/os/x86_64/isolinux/initrd.img
{% endhighlight %}


## Add custom boot entry in CentOS 6.x (or [GRUB 1.x](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Installation_Guide/ch-grub.html)

Add a custom entry into `/boot/grub/grub.conf`:

{% highlight bash %}
title Install CentOS 7
kernel /vmlinuz ks=http://some-web-server.com/anaconda-ks.cfg
initrd /initrd.img
{% endhighlight %}

If you make sure that this entry is the first entry in the configuration file,
you will not have to bother defining this to become the default entry. But if
you decide to **not** place this entry first, you will have to tell grub which
entry this is by changing this line, also in `/boot/grub/grub.conf`:

    default 0


Also, please note you should replace the URL in the entry to reflect the
location of where your Kickstart file is at.

You may wish to add [options](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Installation_Guide/ap-admin-options.html) to the end of the `kernel` line of the boot stanza.


## Add custom boot entry in CentOS 7.x (or [GRUB 2.x](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/ch-Working_with_the_GRUB_2_Boot_Loader.html)

Add a custom menu entry into `/etc/grub.d/40_custom`, which is where custom
boot entries are defined when you use GRUB2:

{% highlight bash %}
menuentry "Install CentOS 7" {
    set root=(hd0,1)
    linux /vmlinuz ks=http://some-web-server.com/anaconda-ks.cfg
    initrd /initrd.img
}
{% endhighlight %}

Add any additional [boot options](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/chap-anaconda-boot-options.html) at the end of the `linux` line of the boot stanza.

Also, please note you should replace the URL in the entry to reflect the
location of where your Kickstart file is at.

Make the custom entry the default choice in `/etc/default/grub`:

{% highlight bash %}
GRUB_DEFAULT="My custom boot entry"
{% endhighlight %}

Then run the following to make your changes go into effect:

{% highlight bash %}
grub2-mkconfig --output=/boot/grub2/grub.cfg
{% endhighlight %}


# Reboot your system to install CentOS 7

Go grab a cup of coffee and reboot your system:

    reboot