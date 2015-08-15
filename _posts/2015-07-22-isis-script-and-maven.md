---
layout: post
title:  "Isis Script and Maven"
date:   2015-07-22 18:27:00
author: Torsten Juergeleit
tags: isis dsl xtext maven
comments: true
---
This post answers the question from [issue #5](https://github.com/vaulttec/isis-script/issues/5) on how to generate Java code from an [Isis Script file](https://github.com/vaulttec/isis-script#the-dsl) in a [Maven](http://maven.apache.org) build.

For using Xtext-based languages in Maven builds, the [Xtext project provides a dedicated Maven plugin](https://eclipse.org/Xtext/documentation/350_continuous_integration.html#standalone-build) - `xtext-maven-plugin`. Here a Maven user just adds the corresponding Xtext language artifact as a dependency to this plugin and defines the language-specific configuration. This configuration contains the name of the `StandaloneSetup` class and the output folder. If your project has a large classpath then you can also specify which jars to search for model files (via `classPathLookupFilter`) to speeding up your build.

{% highlight xml %}
<plugin>
	<groupId>org.eclipse.xtext</groupId>
	<artifactId>xtext-maven-plugin</artifactId>
	<executions>
		<execution>
			<goals>
				<goal>generate</goal>
			</goals>
		</execution>
	</executions>
	<configuration>
		<languages>
			<language>
				<setup>org.vaulttec.isis.script.IsisStandaloneSetup</setup>
				<outputConfigurations>
					<outputConfiguration>
						<outputDirectory>${project.build.directory}/generated-sources/isis</outputDirectory>
					</outputConfiguration>
				</outputConfigurations>
			</language>
		</languages>
		<classPathLookupFilter>.*isis.*</classPathLookupFilter>
	</configuration>
	<dependencies>
		<dependency>
			<groupId>org.vaulttec.isis.script.eclipse</groupId>
			<artifactId>org.vaulttec.isis.script</artifactId>
			<version>1.0.0-SNAPSHOT</version>
		</dependency>
	</dependencies>
</plugin>
{% endhighlight %}

**The language artifact of Isis Script `org.vaulttec.isis.script` is not available in [Maven Central](http://search.maven.org) yet. In the meantime you have to checkout the [Isis Script project from GitHub](https://github.com/vaulttec/isis-script) and install the language artifact in your local Maven repository via `mvn install`.**

In order to let the Maven Java compiler pick up the generated Java code the corresponding output folder has to be added to the Maven projects list of Java source folders. This is done by using the [`build-helper` Maven plugin](http://www.mojohaus.org/build-helper-maven-plugin/).

{% highlight xml %}
<plugin>
	<groupId>org.codehaus.mojo</groupId>
	<artifactId>build-helper-maven-plugin</artifactId>
	<executions>
		<execution>
			<goals>
				<goal>add-source</goal>
			</goals>
			<configuration>
				<sources>
					<source>${project.build.directory}/generated-sources/isis</source>
				</sources>
			</configuration>
		</execution>
	</executions>
</plugin>
{% endhighlight %}

A complete example can be found in the [POM of the simpleapp example](https://github.com/vaulttec/isis-script/blob/develop/isis-script-examples/simpleapp/pom.xml) (part of the [Isis Script GitHub project](https://github.com/vaulttec/isis-script/)).