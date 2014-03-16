---
layout: post
title: "java extensions 101 with MPS"
date: 2014-03-16 16:06
comments: true
categories:
- MPS
- LOP
- samples
---
In the past years I have done lots of work in C# and recently came back to work with Java because I am
now working in a project that involves MPS. The Meta Programming System from JetBrains. Which is a
language workbench with a projectional editor. I got quite used to some of the features C# offers.
One of the features I love the most is implicit strong typed variables, expressed with the `var` keyword
as type in a variable declarations. Due to the extensible nature of MPS it is quite easy to extend existing
languages. So I decided to build this feature in MPS for Java.

Now you might think this ridiculously crazy since it is hard to change an existing language. But in MPS
everything is about modularization, extensibility and composition of languages. And It ships with a
Java implementation called `baselanguage` out of the box. For me it took 30 minutes of work to build
this extension. Though I have a lot of experience with MPS and a good knowledge of baselanguage
but even if you don't have that it should take you not longer than an hour to do it on your own. And that is
what this tutorial is about. What I will show here is how to extend
baselanguage with a new kind of variable declaration that uses its initializer to calculate its type.

<!--more-->

The result should look like this:

{% codeblock var sample  lang:java %}

public class Foo { 
   
  public void main() { 
    var test = 42; 
       
    if (test == 42) { 
      System.out.println("yeah!"); 
    } 
       
  } 
}
{% endcodeblock %}

So what we do first is create a new language in MPS and we add `jetbrains.mps.baselanguage` to the
extended languages like this:

{% img /images/java-101/1.png 650 %}

After that we create a new language concept called `VarDeclaration` which extends `LocalVariableDeclaration`
the local variable declaration concept in baselanguage. We also assign the alias `var` to it. This is later
used to enter the declaration. All together it should look like this:

{% img /images/java-101/2.png 650 %}

Now that we have a language concept it needs a editor. So we go for it and define a editor similar to
one of  `LocalVariableDeclaration` from baselanguage. Since I am lazy I just copied it from there
and replaced the type field with a static text `var`:

{% img /images/java-101/3.png 650 %}

Because we extend the existing `LocalVariableDeclaration` concept we also have a `Type` child in our
concept. And we need to put some data in there to make MPS happy. But we don't have it in our editor
and we don't want the user to mess with this. So we create a behavior aspect for our concept. This
allows us to write constructor code that is executed each time a instance of our concept is created. **But
shouldn't we infer the type from the initializer? How can we set a type in the constructor where the
user hasn't entered a initializer?** Here we use a little trick from baselanguage. It has a special type called
`UndefinedType` which will toggle thy typesystem to not use the `Type` child of the concept in its
rules but rather rely on the typesystems runtime information. This makes some operations a bit more
expensive but gives us more flexibility:

{% img /images/java-101/4.png 650 %}

After we have told the typesystem to use its runtime information instead of relying on static information
we need to tell it how to calculate these runtime information. To do so we need a typesystem rule. We
want to infer a type so its a inference typesystem rule. The rule itself is quite simple. The rule says that
the type of the variable declaration is the same type as its initializer. In MPS we can express this with a
single line:

{% img /images/java-101/5.png 650 %}

But we also want to ensure that user enters a initializer so that we are able to calculate the type from it.
And point out that the user has to enter one in the declaration. We can also us the typesystem to do so. This is called a none
typesystem rule or a check rule. Those rules don't contribute type information at runtime but they ensure
properties of the code. In our case we want to check if the initializer is null and if so we want to show
a error:

{% img /images/java-101/6.png 650 %}

Ok, now we are finished aren't we? Nope! There is still some aspect missing. We need to have a Java/Text
representation of our concept. Through its extensible nature MPS would invoke the text generator of
the `LocalVariableDeclaration` which we extended. But this one tries to use the `Type` child to write
the Java type to the text buffer. In our case this one is always `UndefinedType`. That is why we need
to create our own. Once again we can use the existing one as a blueprint. And replace the `Type` child
with runtime information from the typesystem. To do so we ask the typechecker for the type of the
initializer and pass it where the type of the variable is expected:

{% img /images/java-101/7.png 650 %}

That's all! We are finished now. You can cretate a sanbox solution add the newly created language to the
used languages and see what happen when you type `var`. A new variable declaration shows up that
just looks like the one in C#. And when you generate Java code from it it will look like plain Java:

{% codeblock var sample  lang:java %}
var test = 42; 
   
if (test == 42) { 
  System.out.println("yeah!"); 
}
{% endcodeblock %}

{% codeblock generated java code  lang:java %}
    int test = 42;

    if (test == 42) {
      System.out.println("yeah!");
    }
{% endcodeblock %}

You can get the source code for this sample [here](https://github.com/coolya/ws.logv.baselanguage).