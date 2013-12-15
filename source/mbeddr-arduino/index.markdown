---
layout: page
title: "mbeddr arduino"
date: 2013-09-16 20:48
comments: true
sharing: true
footer: true
---

mbeddr.arduino is a DSL to write programs for the Arduino platform. This page is a short overview
about the project and its goal.

### Motivation

The idea behind it is to make programming the Arduino hardware easier and to avoid common pitfalls
because of lacking typesystem integration or limitations of the preprocessor on which all of the other
libraries for Arduino rely.
This is done by adding first class language concepts for commonly used patterns in the Arduino world.
For example conditional sleep modes:

{% codeblock original C code to avoid race conditions for conditional sleep  lang:c %}
#include <avr/interrupt.h>
#include <avr/sleep.h>

//some code here
set_sleep_mode(<mode>);
cli();
if (some_condition)
{
sleep_enable();
sei();
sleep_cpu();
sleep_disable();
}
sei();
{% endcodeblock %}

{% codeblock mbeddr.arduino equivalent lang:c %}
sleep(mode: Idle Mode, condition: some_condition);
{% endcodeblock %}

### mbeddr

The foundation for all of this is mbeddr which is a extensible implementation of C. You can find tons
of informations and papers about how it works in detail on the [mbeddr website](http://mbeddr.com).

### State

Currently mbeddr.arduino supports most of the Arduino Uno hardware features. It is constantly
evolving and I plan to add support for other board in the future. For now I plan to complete the Uno
support and then move on to some more sophisticated boards. But many of the language concepts arn't
tied to any particular hardware. Further boards will be quite easy to add because mbeddr.arduino itself
uses a DSL to describe its own supported hardware. This also enables you to write descriptions for
missing boards or custom designed boards that are Arduino compatible.
I plan to make a first release early in 2014 when I have finished some necessary refactoring. But you
should be able to play with in the meantime, only be aware that some stuff might break.

### Open Source

Obviously mbeddr.arduino is open source, it is licensed under the Eclipse Public License (EPL). All
source code is available at [Github](https://github.com/coolya/mbeddr.arduino).

### Further Reading

* [Blogpost about Sleep Modes](/b/2013/10/06/sleep-modes-with-mbeddr-dot-arduino/)
* [Blogpost about Serial Reporting](/b/2013/09/15/custom-reporting-with-mbeddr/)

The are also some slides of a short presentation I gave about the project some time ago on speakerdeck:

{% speakerdeck 495ffaa000d601311adb06c85d6e8a99 %}
