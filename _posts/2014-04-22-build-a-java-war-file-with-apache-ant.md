---
layout: post
title: Build a Java WAR file with Ant
localgov: false
tag:
date: "2014-04-22"
meta: "How to build a Java War file using Apache Ant"
---

I'd never really used Ant build files directly until we moved to using Jenkins as a separate build server. Prior to moving to Jenkins we were just exporting our built projects from Eclipse and deploying a .war file directly into Tomcat.

The interesting part came when we had to hand off the build step to Apache Ant because Eclipse was no longer responsible for building our projects. The team had grown, our code was now stored centrally and Eclipse became our development tool, TFS became our version control system and Apache Ant would build out the project on a remote server as an entirely separate process.

<blockquote><p>Apache Ant is a Java library and command-line tool whose mission is to drive processes described in build files as targets and extension points dependent upon each other.</p><b class="cite">From the <a href="http://ant.apache.org/" title="Apache Ant Documentation">Apache Ant Documentation</a></b></blockquote>

Essentially, the Ant library will follow a set of instructions in a specified configuration xml file called a build file. In our case, we're going to create a build file that will take a Java EE project and output a WAR file. In this instance, the WAR file we create will also include the source files for our project, this means that the WAR file can be imported back into Eclipse (or another IDE) should we need to edit it.

Thankfully Ant doesn't rely on any particular project structure.  We're able to specify in our build file where the source files sit as well as how and where we want to output our compiled class files. The same goes for our assets and other resources, the beauty of Ant is that you point it in the right direction rather than it relying on you creating your project in a  certain way.

To me, the core principle of Ant build files is that of the 'target.' A target is a effectively a task, a small module of instructions that Ant will handle. You can link targets together by implementing a principle of dependencies which allows you to write small modules for handling individual aspects of a build and then link them together one after the other. 

## Basic Build

The basic build file starts with creating a 'build.xml' file, create a new xml file in your projects root directory, save it as 'build.xml.'

Now we know roughly what targets are, we're going to begin with the root xml node and a description and then implement our first target:

{% highlight xml %}
<?xml version="1.0" encoding="utf-8" standalone="no"?>
<project name="project-name" basedir="." default="war">
	<description>
		simple example build file
	</description>
</project>
{% endhighlight %}


This root node describes a couple of things, firstly the project name, typically the name of your application. The other attributes 'basedir' and 'default' specify the base directory of the project and the default target to run when Ant attempts to build this project.

I tend to work from start to finish in terms of targets, looking at our project structure and beginning the build process from there.

Our first target needs to include any libraries we may need, otherwise our code isn't going to have the libraries available when it comes to actually compiling the classes.

So, we write our first target to create a directory to store all of our compiled files and then copy our source files into the new directory:

{% highlight xml %}
<project>...

<target name="init">
	<mkdir dir="build/classes"/>
	<copy includeemptydirs="false" todir="build/classes">
	    <fileset dir="src" />
	</copy>
</target>

...</project>	
{% endhighlight %}

By giving the target a 'name' we can refer to it from other targets, you'll see that in each of the targets we add. Inside the 'target' node we place the instructions for Ant, in this case we're just saying 'make a new directory' and then specifying the path of the new directory using the 'dir' attribute.

The 'copy' element then tells Ant to copy files from one directory into another directory. In this case we're asking it to copy all files from our 'src' directory into our new 'build/classes' directory.

Next we want to include any libraries our code needs to compile.  Remember, Ant is just going to read our instructions, if we don't tell it where things like libraries are, it'll fail. We'll include our 'lib' directory like so:

{% highlight xml %}
<project ...

<path id="WebAppLibraries.libraryclasspath">
	<fileset dir="WebContent/WEB-INF/lib">
		<include name="**/*.jar"/>
	</fileset>
	<pathelement location="ImportedClasses"/>
</path>

...</project>
{% endhighlight %}

Here we're creating a path reference to our library files stored in 'WEB-INF/lib', the include tag tells Ant to only care about .jar files. We'll include this path reference with some other libraries when we ask Ant to compile our code.

In my project, I also have a set of required Tomcat libraries so I'll be including those, it's the same principle as the lib directory, this time we're including some external libraries from Tomcat:

{% highlight xml %}
<project ...

<property name="TOMCAT" value="${tomcatdir}"/>

<path id="Apache Tomcat v7.0 [Apache Tomcat v7.0].libraryclasspath">
    <pathelement location="${TOMCAT}/lib/annotations-api.jar"/>
    <pathelement location="${TOMCAT}/lib/jsp-api.jar"/>
    <pathelement location="${TOMCAT}/lib/servlet-api.jar"/>
    <pathelement location="${TOMCAT}/lib/tomcat-api.jar"/>
    <pathelement location="${TOMCAT}/lib/tomcat-coyote.jar"/>
    <pathelement location="${TOMCAT}/lib/tomcat-jdbc.jar"/>
    <pathelement location="${TOMCAT}/lib/tomcat-util.jar"/>
    <pathelement location="${TOMCAT}/lib/tomcat7-websocket.jar"/>
    <pathelement location="${TOMCAT}/lib/websocket-api.jar"/>
</path>

...</project>
{% endhighlight %}


This is essentially the same as the last example, except you'll notice something slightly different here, the '${TOMCAT}' notation appears to have come out of nowhere. What I haven't mentioned until now is that Apache Ant allows you to declare 'property' elements in your build file. Property tags act in a similar way to variables in your code, you can reference something by a name to retrieve its value. In the example above, '${TOMCAT}' references the property tag but the value of 'TOMCAT' is '${tomcatdir}' what?! Well '${tomcatdir}' is passed in at run time, so when we tell Ant to build our project, we'll also have to tell it where our Tomcat directory is. The reason? Environment's are different, so when we move from one machine to another, we don't have to assume that Tomcat is in the same place.


Thankfully we've now included everything needed to build our project, however, we could tidy it up a bit since we have two class path references going, we should really combine them:

{% highlight xml %}
<project ...

<path id="my-project.classpath">
    <pathelement location="build/classes"/>
    <path refid="WebAppLibraries.libraryclasspath"/>
    <path refid="Apache Tomcat v7.0 [Apache Tomcat v7.0].libraryclasspath"/>
</path>

...</project>
{% endhighlight %}

That's it — a single path reference containing references to all the libraries we need to build our project.

Now every thing's been included we can tell any to build the project files:

{% highlight xml %}
<project ...

<target depends="init" name="build-project">
    <echo message="${ant.project.name}: ${ant.file}"/>
    <javac debug="true" debuglevel="source,lines,vars" destdir="build/classes" includeantruntime="false" source="1.7" target="1.7">
        <src path="src"/>
        <classpath refid="my-project.classpath"/>
    </javac>
</target>

...</project>
{% endhighlight %}

This target will do the actual building of our project. Lets break it down a bit...

- depends="init" tells the target that it requires a target called 'init' to run before it can be run
- name="build-project" is the name of this target which can be referenced from other targets as a dependency
- echo prints a message to the console so we know what's doing on, useful for debugging
- javac does the compiling, it has a 'destdir' which sets where the classes will be compiled to, a 'source' and 'target' which state the java version expected at compile time and finally 'src' and 'classpath' set the source file directory and classpath path reference respectively.


Finally, we want to output our WAR file:

{% highlight xml %}
<project ...

<target name="war" depends="build-project">
    <mkdir dir="WebContent/WEB-INF/classes"/>
     
    <copy includeemptydirs="false" todir="WebContent/WEB-INF/classes">
            <fileset dir="build/classes" />
    </copy>
	<fileset dir="src">
	    <include name="**/*.*"/>
	</fileset>
     
    <war destfile="${warfilename}"
        basedir="WebContent" webxml="WebContent/WEB-INF/web.xml">
    </war>
</target>

...</project>
{% endhighlight %}

That's the last piece of the puzzle. 'mkdir' creates an output directory for the compiled class files and the following 'copy' tag moves everything from our 'build/classes' directory into the final 'classes' directory in our WEB-INF folder.

We then include everything from 'src' because if we want to reimport this war file into Eclipse for editing, we'll need the original source files.

Finally the 'war' element tells Ant to zip everything up into a war file, we tell Ant what to call the war file, the base directory to use and where the web.xml config file is. 

If you're keen-eyed, you'll notice that the name of the war file is another run time variable ${warfilename}. This simply allows us to decide the name of the war file each time we run the build. It's great for us because we append the version name from our VCS at build time so that we know which war file was created by which change for easy rollbacks.

### Running the Build

You can run the build from the command line, assuming you've got Apache Ant installed fire up terminal and head to your projects directory. The line we'll need is

{% highlight bash %}
build -buildfile build.xml -Dtomcatdir=path/to/tomcat -Dwarfilename=name-of-warfile.war
{% endhighlight%}

Once you run the command, Ant will take over and read our build file. Ant will register all the 'property' values and run the 'default' target at from the 'project' tag. In this case the 'war' target gets called first but that has a dependency on the 'build-project' target so that gets called. The 'build-project' target has a dependency on the 'init' target so that gets called. Once each target completes, Ant falls back up the chain to finally output our warfile contain both our source and compiled code.

That's it! Hopefully that gives you some idea of how Apache Ant can help you build your projects, by setting up a build file you have an opportunity to have a consistent shared build method that can easily be migrated to a remote server, should you need to.

