Spring XD Custom gunzup Processor
=============================

This is an example of a custom module project that is built and packaged for installation in a Spring XD runtime environment using maven. 
This illustrates how to uncompress .gz files and bind variables defined as module options to a groovy script.

## Requirements

In order to install the module and run it in your Spring XD installation, you will need to have installed:

* Spring XD version 1.3.x ([Instructions](http://docs.spring.io/spring-xd/docs/current/reference/html/#getting-started))

## Code Tour

This implements a simple custom module which simply takes a file reference (as in mode=ref option for file source module) as the ${payload} and a value `${destinationDir}`  then gunzips the file to `${destinationDir}` using Groovy's Apache Ant AntBuilder. 
> AntBuilder allows Ant tasks to be used with a Groovy builder-style markup. It requires that {{ant.jar}} is on your classpath. This module is designed to work with the file source module.

The example demonstrates the use of the `spring-xd-module-parent` pom to package the module.

## Building with Maven

	$ mvn clean package

The project's [pom][] declares `spring-xd-module-parent` as its parent. This adds the dependencies needed to compile and test 
the module and also configures the [Spring Boot Maven Plugin][] to package the module as an uber-jar, packaging any dependencies that are not already provided by the Spring XD container. 

In this case, `spring-xd-extension-script` is a module dependency that must be packaged with the module to be loaded by the module's class loader.
 This component has transitive dependencies to support Spring Integration Groovy scripts which are also exported to the uber-jar

## Using the Custom Module

> Copy the provided ant-1.9.6.jar and ant-launcher-1.9.6.jar to [SRINGXD_HOME]/xd/libs (it seems Spring XD is not loading these jars from the uber jar.)

The uber-jar will be in `[project-build-dir]/gunzip-script-processor-1.0.0.BUILD-SNAPSHOT.jar`. To install and register the module to your Spring XD distribution,
 use the `module upload` Spring XD shell command. Start Spring XD and the shell:


	_____                           __   _______
	/  ___|          (-)             \ \ / /  _  \
	\ `--. _ __  _ __ _ _ __   __ _   \ V /| | | |
 	`--. \ '_ \| '__| | '_ \ / _` |   / ^ \| | | |
	/\__/ / |_) | |  | | | | | (_| | / / \ \ |/ /
	\____/| .__/|_|  |_|_| |_|\__, | \/   \/___/
    	  | |                  __/ |
      	|_|                 |___/
	eXtreme Data
	1.1.0.BUILD-SNAPSHOT | Admin Server Target: http://localhost:9393
	Welcome to the Spring XD shell. For assistance hit TAB or type "help".
	xd:>module upload --type processor --name gunzipper --file [path-to]/gunzip-script-processor-1.0.0.BUILD-SNAPSHOT.jar
	Successfully uploaded module 'processor:gunzipper'
	xd:>


Now create and deploy a stream:

	stream create --name gunzip --definition "file --dir=/path/to/lots/of/gzfiles --mode=ref --pattern=*.gz | gunzipper --variables='destinationDir=/tmp/tar' |log " --deploy 


You should see the stream output in the Spring XD log:


	 [gunzip] Expanding /path/to/lots/of/gzfiles/logilab-astng-0.20.1.tar.gz to /tmp/tar/logilab-astng-0.20.1.tar
	 [gunzip] Expanding /path/to/lots/of/gzfiles/pythonSrc/logilab-common-0.50.1.tar.gz to /tmp/tar/logilab-common-0.50.1.tar


