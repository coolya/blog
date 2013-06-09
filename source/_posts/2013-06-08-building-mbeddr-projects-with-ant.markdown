---
layout: post
title: "Building mbeddr projects with ant"
date: 2013-06-08 18:59
comments: true
categories: 
- ant
- mbeddr
- tutorial
---

This post is about setting up ant to build an mbeddr project. It enables you to do continues integration builds or other automated build scenarios. I assume that you have already installed mbeddr. If you haven't, you should follow the instructions in the [mbeddr user guide][userguide] or [this][tutorial] tutorial. It also assumes that you are using the source code distribution of mbeddr and have build it. If are using the binary distribution have a look at the [end](#nosource) of this tutorial.

First go to you project folder and create a `build.xml` and `build.properties` file. The `build.properties` file should look like this:

```
mps.home=/path/to/mps
mps.platform.caches=/path/to/cache
mbeddr.github.core.home=/path/to/mbeddr
```

Obviously you need to set the variable to some sane values for you environmental.

The next step is to create a skeleton for the `build.xml`:

```
<project name='mbeddr.arduino' default='build'>
  <property file='build.properties'/>
  <path id='mps.ant.path'>
    <pathelement location='${mps.home}/lib/mps-backend.jar'/>
    <pathelement location='${mps.home}/lib/jdom.jar'/>
    <pathelement location='${mps.home}/lib/log4j.jar'/>
    <pathelement location='${mps.home}/lib/mps-core.jar'/>
  </path>
  <taskdef resource='jetbrains/mps/build/ant/antlib.xml' classpathref='mps.ant.path'/>
  <jvmargs id='myargs'>
    <arg value='-ea'/>
    <arg value='-Xss1024k'/>
    <arg value='-Xmx2048m'/>
    <arg value='-XX:MaxPermSize=128m'/>
    <arg value='-XX:+HeapDumpOnOutOfMemoryError'/>
    <arg value='-Didea.system.path=${mps.platform.caches}/system'/>
    <arg value='-Didea.config.path=${mps.platform.caches}/config'/>
    <arg value='-Didea.plugins.path=${mps.platform.caches}/plugins'/>
  </jvmargs>
  <!--Content goes here -->
</project>
```

Now you need to fill this skeleton with some actual content. There three kinds of content that are possible:

- Building [Languages](#languages)
- Building [Solutions](#solutions)
- Running Tests

This tutorial focuses on the first two. If you want to run tests the official [MPS docs][mpsdocs] should give you further information

### Building languages<a name="languages">&nbsp;</a>

In order to build a language you add this code at the `Content goes here` placeholder:

```
  <target name='build'>
    <mps.generate parallelMode='true' fork='true'>
      <jvmargs refid='myargs'/>
      <project file='path/to/your/mpr'/>
      <modules dir='./languages'/>
      <library name="mbeddr"
         dir="${mbeddr.github.core.home}/code/languages"/>
    </mps.generate>
  </target>
```

Make sure that the `file` attribute of the `project` node points to the .mpr file you want to build. If you have multiple mpr files you will have to have multiple `target` nodes which you can separate by `name`. To see how to build them in a single step have a look at the [putting it all together](#alltogether) section.

The `module` node is used to specify a subdirectory. You can use it to limit the amount of code that is build. For instance you can prevent building anything else than a single language.

Also make sure that the `dir` attribute of the mbeddr library is pointing to the **code/languages** subdirectory since we don't need the samples in the build.

### Building solutions<a name="solutions">&nbsp;</a>

After you have build your language you might want to use it. :-)

In order to do so you add this in the `Content goes here` section:

```
	<target name='build'>
	    <mps.generate parallelMode='true' fork='true'>
	      <jvmargs refid='myargs'/>
	      <project file='path/to/your/mpr'/>
	      <modules dir='./solutions'/>
	      <library name="yourlanguage" dir="./languages"/>
	      <library name="mbeddr"
	         dir="${mbeddr.github.core.home}/code/languages"/>
	    </mps.generate>
	</target>
```

It is quite similar to the one used to build a language, except it's `module` attribute refers to the `solutions` subdirectory. As for the languages you can specify it to be more concrete to  just build a subset of your solutions. You also notice an extra `library` node which references your languages. If you don't have any languages extending mbeddr and just have solutions using it you can remove this node.

Also make sure that the `dir` attribute of the mbeddr library is pointing to the **code/languages** subdirectory since we don't need the samples in the build.

### Putting it all together<a name="alltogether">&nbsp;</a>

Now that you know how to build languages and solutions we can put all these together in order to build a hole project. This is not really mbeddr or MPS specific, it's just plain ant stuff to do. 

But first let me explain why separating this into two steps. Yes you could simply build a hole mpr file, but most of the time you want to have some more fine grained control about what is build and when. If the languages build would fail the solutions would fail anyway and it would be hard to figure out what went wrong, except you seek though all the logs.
If you separate them you can easily see that there is a problem with the languages and simply ignore the solution build errors. This is extremely handy when running continuous integration builds against your check-ins. 

What you need to do is, in contrast to name the target just `build`, you create two different like this:

```
  <target name='build-languages'>
  	<!-- language build description goes here -->
  </target>
  <target name='build-solution'>
  	<!-- language build description goes here -->
  </target>
```

Now you can type `ant build-languages` to build the languages or `ant build-solutions` to build the solutions. But what if you would like  to build everything with a single command?

You create a new target named `build` with two `antcall` elements:

```
  <target name="build">
    <antcall target="build-languages"/>
    <antcall target="build-solutions"/>
  </target>
```

Be aware that the order matters here. So you need to build the languages first an then the solutions.

### What if I don't use mbeddr from source?<a name="nosource">&nbsp;</a>

If you want to use the mbeddr binary distribution and you have installed it already you simply remove the `mbeddr` library from your build files. A solution would look like this:

```
	<target name='build'>
	    <mps.generate parallelMode='true' fork='true'>
	      <jvmargs refid='myargs'/>
	      <project file='path/to/your/mpr'/>
	      <modules dir='./solutions'/>
	      <library name="yourlanguage" dir="./languages"/>
	    </mps.generate>
	</target>
```

[tutorial]: http://www.logv.ws/b/2013/04/27/installing-mbeddr-on-ubuntu-13-04/
[ant]: https://ant.apache.org/
[userguide]: http://voelter.de/mbeddr/1268/mbeddr-userguide.pdf
[ant-manual]: https://ant.apache.org/manual/index.html
[mpsdocs]: http://confluence.jetbrains.com/display/MPSD25/HowTo+--+MPS+and+ant