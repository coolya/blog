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
echo 'deb http://dl.logv.ws/repository ./' >> /etc/apt/source.list
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