---
layout: post
title: "custom reporting with mbeddr"
date: 2013-09-15 12:21
comments: true
author: Kolja Dummann
categories: 
- mbeddr
- tutorial
---

### Introduction

As some of you might already know I have worked on an Arduino extension for mbeddr. Because such system run headless you can't just use `printf` for displaying messages. These systems usually ship with a serial or USB port which can be used to communicate with the world. There are actually two ways to deal with this, first we could replace the printf backing file handle with a handle that writes to the serial port, or second use mbeddrs own reporting infrastructure to deal with this. The mebddr approach also give us the more flexibility because we could either write messages to the serial port or store critical errors in the EPROM for further investigation. 

Here I will talk about how to build your own reporting backend into mbeddr, in this case a backend that writes to the serial port.

### Architecture

First I will give you an overview about the reporting architecture of mbeddr. It consists of four main parts: `MessagesDefinition`, `MessageDefinitionTable`, the`ReportStatement` and a `ReportingStrategy`. 

#### MessagesDefinition

Messages as the name says define messages. They have a name to reference them, a text that is written out when the message is reported and the **can** have parameters. These parameters are basically key-value pairs. A message also has additional attributes: 

**MessageSeverity**:  similar to a priority it can be `ERROR`,`WARN` and `INFO`

**active**: a boolean flag that defines if the message is active or not. 

**count**: a boolean flag that defines if the times the messages war reported should be counted.

#### MessageDefinitionTable

A collection on messages. Any `MessagesDefinition` belongs to a `MessageDefinitionTable`. It acts as a kind of namespace for messages.

#### ReportStatement

The `ReportStatement` is used to report a message. It references to a message and provides the actual values for parameters, if the message defines any. The parameters can be any value matching the type of definition, for instance a local variable or a value obtained from an external sensor.  The statement also provides so called `checks` which is a guard only. If it evaluates to `true` the actual message will be reported.

#### ReportingStrategy

The `ReportingStrategy` is used inside the build configuration to specify what kind of reporting the project uses. In mbeddr there already to predefined reporting strategies: printf and nothing. As the name suggest the first one uses simple `printf` statements to report a message and the later does nothing with them.

### Implementation

Now that we have a overview about what we need let's start implementing our own reporting.

<!--more-->

#### ReportingStrategy

 The first thing we need is a `ReportingStrategy`.  To do so we create a new Concept and name it `SerialReportingStrategy`:

{% img /images/arduino-reporting/1.png 750 'Screenshot of SerialReportingStrategy '  %}

The editor also looks straight forward, just a constant cell with the words "serial reporting" in it:

{% img /images/arduino-reporting/2.png 750 'Screenshot of SerialReportingStrategy  editor'  %}

Now we are able to select this in the mbeddr build configuration:

{% img /images/arduino-reporting/3.png 750 'Selection in build config'  %}

So thats it for the visible part. We can select it but it doesn't really do anything. Because there is no generator that would generate the code to write the serial port. To change this we need to add a generator that reduces the concepts if serial reporting is selected. 

#### Generator

First we create a new Generator in MPS:

{% img /images/arduino-reporting/4.png 750 'generator skeleton'  %}

As you can see the `is applicable` rule checks if the selected reporting in the build config is the `SerialReportingStrategy`otherwise this generator will not do anything.  

Let's start with the easiest thing the generator should handle: disabled messages! We create a reduction rule that only applies to messages that have the `active` flag set to `false` and then abandons the input:

{% img /images/arduino-reporting/5.png 750 'abandon inactive message'  %}

The reporting architecture of mbeddr contains two core elements: a `MessageDefinition`  and a `MessageDefinitionTable` which combines multiple messages. Messages are always part of a message definition table. So we need to reduce the table first. The generator is also quite simple. It take a `MessageDefinitionTable` and calls the `$COPY_SRCL$` for its messages. Which loops over all of the massages and tries to reduce them:

{% img /images/arduino-reporting/6.png 750 'reduce message definition table'  %}

To get all the messages to reduce we use the `nonEmptyMessages` method of the `MessageDefinitionTable` behavior:

{% img /images/arduino-reporting/7.png 750 'copy source list macro definition'  %}

Next we need to reduce the message definition itself. 
We do two things in this reduction rule. First we generate a counter variable for the message. This variable can be used in tests to check how often a message occurred. In normal C code this variable is not relevant. And second we generate a `char*` variable that is initialized with the message text for later usage when the message is fired.[^1]

{% img /images/arduino-reporting/8.png 750 'message reduction'  %}

The property macros for the names are quite simple, they just generate a unique name for each variable so that we can reference them later. The macro for the initializer of the `string` is a bit more interesting. It generates its value from the name of the message and the text but also appends a `\n` at the end for line ending.

{% img /images/arduino-reporting/9.png 750 'message text'  %}

Now that we have the texts and the counters generated we can start with the part that emits the code to post the message to the serial port. I our case we have a library that does all the details. All we need to do is call a function with a `char*` as a argument. The lib is added via the make file an header file is called `serial.h`.  So the next thing to do is reducing the `ReportStatements`, but in order to keep the generator clean we implemented two different reduction rules one for a message with parameters and one for messages without. Later you will see why. Here is what the generator for messages without parameters looks like:

{% img /images/arduino-reporting/10.png 750 'report reduction'  %}

The `condition` checks if there are no arguments and it does not contain an `check` guard. The reduction rule than defines some dummy variable and a dummy function. Inside the function an `ArbitraryTextExpression` is used to insert some text which is not a expressed with mbeddr semantics.  In the inspector you see that it also specifies a header to include and a type.

The both variables use the reference macro and the message counter is also guarded with an if in the generator to not emit that code when the message is not counted. 

Next is the `ReportStatement` with properties. It requires some more work to be done because we can not use printf we have to use sprintf and store that result in a buffer before we can put it to the serial port:

{% img /images/arduino-reporting/11.png 750 'report reduction'  %}

The major difference here is that it first generates a buffer variable to use it with sprintf, then prints the message and after that loops over all properties of the message where it prints them to the buffer and finally puts them to the serial port.  The size of the buffer is also calculated in the reduction rule. It takes the maximum length of the property name and adds 24 to which is the maximum length of a 64 bit integer printed as a string plus the characters added to make it look pretty. There is downside of this hard coded number, it will cause long strings to be corrupted. 

### What's next?

As you might have noticed the hole part of checks is missing. I omitted this because it would blow up the post and shouldn't be that hard to do for you if you are familiar with the generator. To prevent data corruption of long string parameters you can either prohibit the usage of anything else than a number as parameter by using a type system check rule or calculate the length at runtime. It's up to you!

The main part of this code was written while adding serial reporting support to the [mbeddr.arduino](https://github.com/coolya/mbeddr.arduino) project. You can have a look at the code there for further references. 

 [^1]: Note: Due some limitations of older GCC versions char literals in C code would end up in the flash area of the memory which is a different address space. This is why the generator generates char* variables for each of messages. This done to either accomplish backward compatibility and to illustrate the difference between this generator and the printf generator. If I would have been lazy I could have simply copied the old generator and replaced the printf calls.


