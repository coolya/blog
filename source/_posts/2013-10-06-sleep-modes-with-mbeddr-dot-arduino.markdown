---
layout: post
title: "sleep modes with mbeddr.arduino"
date: 2013-10-06 10:25
comments: true
categories:
- mbeddr
- arduino
- MDSD
---
Here is a little status update on the mbeddr.arduino project.  We are getting closer to the first "release", which will be an early access version but we want to get it into the hands of people to try it.  The main topic on this post is not the release ;-)  it is  about the opportunity to avoid common pitfalls in Arduino programing with Model Driven Software Development (MDSD). 
This weekend I have added support for **sleep modes**. While I was reading the docs for [avr-libc](http://www.nongnu.org/avr-libc/user-manual/index.html) I stumbled op on the possibility that [race conditions](http://www.nongnu.org/avr-libc/user-manual/group__avr__sleep.html) could occur when simply using the macros from avr-libc. This can happen if you just want to put the cpu to sleep if some condition is true and you are sure that the necessary interrupts to wakeup are enabled. To avoid this problem you normally write code like this:

{% codeblock avoid race condition  lang:c %}
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

The code disables the all interrupts, then checks the condition and when the condition is meet sets the sleep mode, enables the interrupts again and then  sends the cpu to sleep. Since the instruction right behind the `sei()` macro is guaranteed to be executed before the interrupts are enabled this is a save way to do this.
In mbeddr.arduino I wanted to provide a easier way to do this. 

But lets first have a look at the basic sleep statement which looks just like the one in C:

{% codeblock simple sleep  lang:c %}
sleep(Idle Mode);
{% endcodeblock %}

Except there is some syntactic sugar with blanks instead of using underscores. If you just want to sleep if a certain condition is true you can use a intention to add a condition to the sleep statement:

{% img /images/sleep/1.png 300 %}

After you have done this the statements looks a bit different:

{% codeblock sleep with condition lang:c %}
sleep(mode: Idle Mode, condition: <expr>);
{% endcodeblock %}

You can now add you condition when you want to go to sleep mode in the sleep statement and the generator will take care that it is checked with disabled interrupts. 

This way the user just has to know that putting the controller into sleep mode under a certain condition is something special, but he doesn't need to know how to do it. In order to point to him that he might be doing something dangerous the typesystem checks if there is a IfStatement with a single SleepStatement under it and issues a warning. To fix this warning the user can use a simple intention:

{% img /images/sleep/2.png 400 %}

I hope this gave a little impression about what goal we are after with mbeddr.arduino and how MDSD can help this.
