Repository with the code for the "Java Build Tools" article in [TechnologyConversations](http://technologyconversations.com/). 

# Ant, Maven, and Gradle

The three dominant Java build tools.

Ant and Maven both use XML to write build scripts - named `build.xml` and `pom.xml`, respectively. While ant's `build.xml` is procedural (tells the computer exactly how to build the project), Maven's `pom.xml` is declarative (tells the computer the desired state of the project). Gradle, instead of using XML, provides its own Groovy-based domain-specific-langauge for use in a `build.gradle` build script.

Ant is the most barebones, is intended as a replacement for `make`, and relies upon Ivy for dependency management. Maven and Gradle are much more robust and provides a lot of more functionality out-of-the-box, like dependencies management and project inheritance. Maven is highly-opinionated, whereas Gradle code is typically much more concise and readable than equivalent Ant or Maven code, though the Gradle tool itself has a greater learning curve than the much simpler Ant. Gradle is also the build system supported by Android Studio. Based on all this, Gradle seems to be the better option at the time of writing.

The product of build tools are called 'artifacts' - they are frequently an executable type such as a `jar` or `exe`.

# Ant

To build, just run [`ant`](https://ant.apache.org/manual/running.html) in the same directory as your `build.xml` file.

## [build.xml](https://ant.apache.org/manual/using.html)

An ant buildfile contains one `project`, which has a `name` attribute and a `default` attribute - the `default` attribute specifies the default `target` to pursue.

Each `project` contains *at least* one `target`. Targets are the desired goals for the buildfile, like compiling or creating a distributble. Targets may have dependencies between them, defined by the `depends` attribute - for example, compilation must happen before building the distributable. Targets consist of one or more [`tasks`](https://ant.apache.org/manual/tasksoverview.html) - steps necessary to achieve the target. An example can be seen below.

    <target name="compile" depends="resolve">
        <mkdir dir="${classes.dir}"/>
        <javac srcdir="${src.dir}" destdir="${classes.dir}" classpathref="lib.path.id"/>
    </target>

You can create shortcuts for commonly-used strings with the `property` tag, and paths with the `path` and `classpath` tags.

To use ant with ivy, add the attribute `xmlns:ivy="antlib:org.apache.ivy.ant` to your `project` tag, as well as a target to retrieve ivy (see the `resolve` target defined below). Make sure that another critical task depends on the ivy task (for example, see the `compile` target defined above). [Here](http://ant.apache.org/ivy/history/latest-milestone/ant.html) is a detailed explanation on what happens when you call `<ivy:retrieve/>` (not really necessary to know).

    <target name="resolve">
        <ivy:retrieve/>
    </target>

The provided `build.xml` file retrieves ivy, cleans (deletes) the build directory, compiles the source code into a new `classes` directory via `javac`, and then creates a jar in a new `jar` directory.

## ivy.xml

Defines dependencies for your project. [This link](http://ant.apache.org/ivy/history/latest-milestone/ivyfile.html) demonstrates how to structure an ivy.xml file, and [this link](http://ant.apache.org/ivy/history/latest-milestone/tutorial/start.html) provides a quick start. In general, the structure is as below:

    <ivy-module version="2.0">
        <info organisation="my.org" module="my-module"/>
        <dependencies>
            <dependency org="junit" name="junit" rev="4.11"/>
            <dependency org="org.hamcrest" name="hamcrest-all" rev="1.3"/>
        </dependencies>
    </ivy-module>

The values for [`organisation`](http://ant.apache.org/ivy/history/latest-milestone/terminology.html#organisation) and [`module`](http://ant.apache.org/ivy/history/latest-milestone/terminology.html#module) can be whatever you wish them to be. The `dependencies` section can contain an arbitrary number of dependencies - the `org` and `name` attributes are used to identify dependencies, while `rev` is used to specify a version. Dependencies can be found from [the Maven repository](https://mvnrepository.com/).


# Maven

Maven expects a standard directory layout, seen [here](https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html). The three main components required are the `pom.xml` buildfile, the `src/main` source code directory, and the `src/test` testing directory.

To build, just run `mvn package` in the same directory as your `pom.xml` file.

[This](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html) is a quick start guide for Maven, while [this](https://maven.apache.org/guides/getting-started/index.html) is a more comprehensive resource.

## [pom.xml](https://maven.apache.org/pom.html)

POM stands for Project Object Model.

At the top, the `project` tag defines the XML namespace, XML schema instance, and schema location with the `xmlns`, `xmlns:xsi`, and `xsi:schemaLocation`.

In Maven, `groupId`, `artifactId`, and `version` compose ["coordinates"](https://maven.apache.org/pom.html#Maven_Coordinates) - these work as an address struecture, and together, they uniquely identify an object, and are used to define where artifacts are produced. `groupId` represents an organization, while `artifactId` represents a project, and `version` is obviously the version number. 

For example, if the `groupId` is `org.codehaus.mojo`, the `artifactId` is `my-project`, and the `version` is `1.0`, the directory where artifacts are produced will be `$M2_REPO/org/codehaus/mojo/my-project/1.0`, where `$M2_REPO` is your [Maven repository](https://maven.apache.org/guides/introduction/introduction-to-repositories.html).

`packaging` specifies the type of artifact produced, such as `jar`, `war`, or `rar`.

### [Build](https://maven.apache.org/pom.html#Build)

Build options initiated by the `build` tag. Builds have a `defaultGoal`, which is used if no goal is specified in the command line, and a `directory`, which specifies which dirctory to dump its files into and defaults to `${basedir}/target`. Below is an example build.

    <build>
        <defaultGoal>install</defaultGoal>
        <directory>${basedir}/target</directory>
    </build>

[Resources](https://maven.apache.org/pom.html#Resources) or [plugins](https://maven.apache.org/pom.html#Plugins) that are to be bundled with your project can also be specified in the build.

#### Goals

Maven's *goals* are similar to Ant's *targets*.

Below is a non-comprehensive list of goals.

* clean
* compile
* test
* package
* verify
* integration-test
* install
* deploy

These goals are accomplished within *phases* of the [*build lifecycle*](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html).

To run a particular goal, run `maven <goal>` in the command line. For example, to create the jar distributable, you can run `maven package`. If no goal is specified (e.g. if you simply run `maven`), the `defaultGoal` specified in the `pom.xml` will be used.

### Dependencies

Dependencies can be defined inside the [`dependencies`](https://maven.apache.org/pom.html#Dependencies) tag - each dependency is identified by coordinates `groupId`, `artifactId`, and `version`, just like the project itself. [Version requirements](https://maven.apache.org/pom.html#Dependency_Version_Requirement_Specification) can be hard ('exactly 1.0', soft ('we recommend 1.0'), or bounded ('at least 1.0', 'at most 1.0', 'between 1.0 and 1.9', etc.). Dependencies can also have a `type` (defaults to `jar`). In Maven, dependencies must be other Maven artifacts, so that they can be identified by Maven coordinates.

### [Properties](https://maven.apache.org/pom.html#Properties)

POM files can also have `properties`, just like in ant's `build.xml` files. A property `X` can be accessed elsewhere in the `pom.xml` file using the notation `${X}`. The provided `pom.xml` does not use `properties`; below is an example.

    <properties>
        <maven.compiler.source>1.7</maven.compiler.source>
        <maven.compiler.target>1.7</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    </properties>

### [Inheritance](https://maven.apache.org/pom.html#Inheritance)

POM files may `inherit` from a *Super POM* - this is useful if, for example, multiple projects in the same organization require a shared set of dependencies.


# Gradle

Gradle is very robust and features almost all the same features as Maven, including plugins, repositories, dependencies, etc.

## [build.gradle](https://docs.gradle.org/current/userguide/tutorial_using_tasks.html) 

