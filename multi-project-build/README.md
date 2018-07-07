This is an example application of my blog post:

* [Getting Started with Gradle: Creating a Multi-Project Build](http://www.petrikainulainen.net/programming/gradle/getting-started-with-gradle-creating-a-multi-project-build/)

You can package the application by running one of the following commands at command prompt:

    gradle assemble (runs only the tasks required to create the jar file)
    gradle build (runs the full build)
    
You can can run the application by the following command at command prompt:

    gradle run

You create runnable binary distribution by running one of the following commands at command prompt:

    gradle distZip (creates a runnable binary distribution that is packaged into a .zip file)
    gradle distTar (creates a runnable binary distribution that is packaged into a .tar file)

Both of the commands creates the packaged binary distribution to the _app/build/distributions_ directory.

You can run the application by following these steps:

1. Unpackage the binary distribution.
2. Run the application by invoking the correct startup script found from the _bin_ directory.  


The Requirements of Our Gradle Build
Our example application has two modules:

The core module contains the common components that are used by the other modules of our application. In our case, it contains only one class: the MessageService class returns the string ‘Hello World!’. This module has only one dependency: it has one unit test that uses Junit 4.11.
The app module contains the HelloWorld class that starts our application, gets a message from a MessageService object, and writes the received message to a log file. This module has two dependencies: it needs the core module and uses Log4j 1.2.17 as a logging library.
Our Gradle build has also two other requirements:

We must be able to run our application with Gradle.
We must be able to create a runnable binary distribution that doesn’t use the so called “fat jar” approach.
If you don’t know how you can run your application and create a runnable binary distribution with Gradle, you should read the following blog post before you continue reading this blog post:
Getting Started with Gradle: Creating a Binary Distribution
Let’s move on and find out how we can create a multi-project build that fulfills our requirements.

Creating a Multi-Project Build
Our next step is to create a multi-project Gradle build that has two subprojects: app and core. Let’s start by creating the directory structure of our Gradle build.

Creating the Directory Structure
Because the core and app modules use Java, they both use the default project layout of a Java project. We can create the correct directory structure by following these steps:

Create the root directory of the core module (core) and create the following the subdirectories:
The src/main/java directory contains the source code of the core module.
The src/test/java directory contains the unit tests of the core module.
Create the root directory of the app module (app) and create the following subdirectories:
The src/main/java directory contains the source code of the app module.
The src/main/resources directory contains the resources of the app module.
We have now created the required directories. Our next step is to configure our Gradle build. Let’s start by configuring the projects that are included in our multi-project build.

Configuring the Projects that Are Included in Our Multi-Project Build
We can configure the projects that are included in our multi-project build by following these steps:

Create the settings.gradle file to the root directory of the root project. A multi-project Gradle build must have this file because it specifies the projects that are included in the multi-project build.
Ensure that the app and core projects are included in our multi-project build.
Our settings.gradle file looks as follows:

1
2
include 'app'
include 'core'
Additional Reading:
Gradle User Guide: 55.2 Settings file
Gradle DSL Reference: Settings
Let’s move on and configure the core project.

Configuring the Core Project
We can configure the core project by following these steps:

Create the build.gradle file to the root directory of the core project.
Create a Java project by applying the Java plugin.
Ensure that the core project gets its dependencies from the central Maven2 repository.
Declare the JUnit dependency (version 4.11) and use the testCompile configuration. This configuration describes that the core project needs the JUnit library before its unit tests can be compiled.
The build.gradle file of the core project looks as follows:

1
2
3
4
5
6
7
8
9
apply plugin: 'java'
 
repositories {
    mavenCentral()
}
 
dependencies {
    testCompile 'junit:junit:4.11'
}
Additional Reading:
Getting Started with Gradle: Our First Java Project
Getting Started with Gradle: Dependency Management
Let’s move on and configure the app project.

Configuring the App Project
Before we can configure the app project, we have to take a quick look at the dependency management of such dependencies that are part of the same multi-project build. These dependencies are called project dependencies.

If our multi-project build has projects A and B, and the compilation of the project B requires the project A, we can configure this dependency by adding the following dependency declaration to the build.gradle file of the project B:

1
2
3
dependencies {
    compile project(':A')
}
Additional Reading:
Gradle User Guide: 50.4.3 Project dependencies
Gradle User Guide: 56.7 Project lib dependencies
We can now configure the app project by following these steps:

Create the build.gradle file to the root directory of the app project.
Create a Java project by applying the Java plugin.
Ensure that the app project gets its dependencies from the central Maven2 repository.
Configure the required dependencies. The app project has two dependencies that are required when it is compiled:
Log4j (version 1.2.17)
The core module
Create a runnable binary distribution.
The build.gradle file of the app project looks as follows:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
apply plugin: 'application'
apply plugin: 'java'
 
repositories {
    mavenCentral()
}
 
dependencies {
    compile 'log4j:log4j:1.2.17'
    compile project(':core')
}
 
mainClassName = 'net.petrikainulainen.gradle.client.HelloWorld'
 
task copyLicense {
    outputs.file new File("$buildDir/LICENSE")
    doLast {
        copy {
            from "LICENSE"
            into "$buildDir"
        }
    }
}
 
applicationDistribution.from(copyLicense) {
    into ""
}
Additional Reading:
Getting Started with Gradle: Creating a Binary Distribution
Let’s move on and remove the duplicate configuration found from the build scripts of the core and app projects.

Removing Duplicate Configuration
When we configured the subprojects of our multi-project build, we added duplicate configuration to the build scripts of the core and app projects:

Because both projects are Java projects, they apply the Java plugin.
Both projects use the central Maven 2 repository.
In other words, both build scripts contain the following configuration:

1
2
3
4
5
apply plugin: 'java'
 
repositories {
    mavenCentral()
}
Let’s move this configuration to the build.gradle file of our root project. Before we can do this, we have to learn how we can configure our subprojects in the build.gradle file of our root project.

If we want to add configuration to a single subproject called core, we have to add the following snippet to the build.gradle file of our root project:

1
2
3
project(':core') {
    //Add core specific configuration here
}
In other words, if we want to move the duplicate configuration to the build script of our root project, we have to add the following configuration to its build.gradle file:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
project(':app') {
    apply plugin: 'java'
 
    repositories {
        mavenCentral()
    }
}
 
project(':core') {
    apply plugin: 'java'
 
    repositories {
        mavenCentral()
    }
}
This doesn’t really change our situation. We still have duplicate configuration in our build scripts. The only difference is that the duplicate configuration is now found from the build.gradle file of our root project. Let’s eliminate this duplicate configuration.

If we want to add common configuration to the subprojects of our root project, we have to add the following snippet to the build.gradle file of our root project:

1
2
3
subprojects {
    //Add common configuration here
}
After we have removed the duplicate configuration from the build.gradle file of our root project, it looks as follows:

1
2
3
4
5
6
7
subprojects {
    apply plugin: 'java'
 
    repositories {
        mavenCentral()
    }
}
If we have configuration that is shared by all projects of our multi-project build, we should add the following snippet to the build.gradle file of our root project:
1
2
3
allprojects {
    //Add configuration here
}
Additional Reading:

Gradle User Guide: 56.1 Cross project configuration
Gradle User Guide: 56.2 Subproject configuration
We can now remove the duplicate configuration from the build scripts of our subprojects. The new build scripts of our subprojects looks as follows:

The core/build.gradle file looks as follows:

1
2
3
dependencies {
    testCompile 'junit:junit:4.11'
}
The app/build.gradle file looks as follows:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
apply plugin: 'application'
 
dependencies {
    compile 'log4j:log4j:1.2.17'
    compile project(':core')
}
 
mainClassName = 'net.petrikainulainen.gradle.client.HelloWorld'
 
task copyLicense {
    outputs.file new File("$buildDir/LICENSE")
    doLast {
        copy {
            from "LICENSE"
            into "$buildDir"
        }
    }
}
 
applicationDistribution.from(copyLicense) {
    into ""
}
We have now created a multi-project Gradle build. Let’s find out what we just did.

What Did We Just Do?
When we run the command gradle projects in the root directory of our multi-project build, we see the following output:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
> gradle projects
:projects
 
------------------------------------------------------------
Root project
------------------------------------------------------------
 
Root project 'multi-project-build'
+--- Project ':app'
\--- Project ':core'
 
To see a list of the tasks of a project, run gradle <project-path>:tasks
For example, try running gradle :app:tasks
 
BUILD SUCCESSFUL
As we can see, this command lists the subprojects (app and core) of our root project. This means that we have just created a multi-project Gradle build that has two subprojects.

When we run the command gradle tasks in the root directory of our multi-project build, we see the following output (only relevant part of it is shown below):

1
2
3
4
5
6
7
8
9
10
11
12
13
> gradle tasks
:tasks
 
------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------
 
Application tasks
-----------------
distTar - Bundles the project as a JVM application with libs and OS specific scripts.
distZip - Bundles the project as a JVM application with libs and OS specific scripts.
installApp -Installs the project as a JVM application along with libs and OS specific scripts
run - Runs this project as a JVM application 
As we can see, we can run our application by using Gradle and create a binary distribution that doesn’t use the so called “fat jar” approach. This means that we have fulfilled all requirements of our Gradle build.  