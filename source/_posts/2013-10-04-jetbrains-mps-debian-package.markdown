---
layout: post
title: "jetbrains MPS debian package"
date: 2013-10-04 13:38
comments: true
categories: 
---
Good news for all debian and ubuntu users using the [Meta Programming System](http://www.jetbrains.com/mps/) by JetBrains: I have just created a debain package for it. I have also setup a repository on this server to allow easy handling. 

In order to use it open a root shell and run the following commands. Those will add the repository and import my public gpg key:

{% codeblock lang:sh %}
echo 'deb http://dl.logv.ws/repository ./' >> /etc/apt/sources.list
wget http://logv.ws/files/kdummann.asc
apt-key add kdummann.asc
apt-get update
{% endcodeblock %}

After that you can simply install MPS by typing;

{% codeblock lang:sh %}
apt-get install jetbrains-mps
{% endcodeblock %}

The same also applies to mbeddr:

{% codeblock lang:sh %}
apt-get install mbeddr
{% endcodeblock %}

**Update**:
Currently the mbeddr package has optional dependencies for example the `nusmv` package which aren't available on many distributions, we will provides theses packages in the same repository shortly. This will allow us to not depend on any distribution update schedule when we need new versions of this tools. Sadly we will not be able to distribute `yices` which is used by mbeddr for analyses because their license doesn't allow is to redistribute their software. You will still have to accept their license manually and install it by hand. But I will post a tutorial how to do that on the [mbeddr blog](http://mbeddr.wordpress.com/blog).

**Update 2**:
Some Linux Mint users reported that they had problems with the openJDK on that platform. If you experience any strange problems please switch to the Oracle JDK and see if this fixes your problems. For the german speaking people of you there is a detailed description how to it [here](http://wiki.ubuntuusers.de/Java/Installation/Oracle_Java#Java-7-JDK).