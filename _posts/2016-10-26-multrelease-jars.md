---
layout: post
title: Generating Multi-Release JARs with Maven
categories: 
date: 2016-10-26 12:00:00
---

Java 9 is diverging from Java 8 in more and more ways - including changes which are not strictly compatible (in contrast to previous major version transitions).  Recognition of this reality has lead to the development of <a href="http://openjdk.java.net/jeps/238">JEP 238: Multi-Release JAR Files</a>, which allows some or all of a JAR file to utilize overlays on newer JDK versions.  However, as of the date of this publication, Maven does not yet have an official solution for this problem.  Some complex approaches have been tried, including <a href="https://github.com/hboutemy/maven-jep238/blob/master/multirelease/pom.xml#L21">using the assembly plugin with submodules</a>, but as of yet I have found no simple approach described.  Thus I have come up with an approach of my own.

## The Test Subject

The subject of this experiment is <a href="https://github.com/jboss-modules/jboss-modules">JBoss Modules</a>.  This project deals intimately with class loaders, the new Jigsaw modules, and the JDK in general, and is quite sensitive to changes in the JDK, so it has become necessary to provide different implementations of certain operations depending on what JDK is being run.  I could have done this a few different ways, but ultimately I settled on MR JARs.

### Requirements

My requirements are as follows:

0. The project has to run correctly in both Java 8 and Java 9
0. The main code base must be able to be built with Java 8
0. The Java 9 portion must be built with Java 9 (therefore the Java 9 portion of the build must be isolated from the Java 8 portion)
0. The end result must be a single JAR

In addition I had these goals:

0. Do not require the project to switch to a multi-module format
0. Keep the build configuration simple

### Strategy

The basic implementation strategy has several parts.

#### Isolate JDK-specific code

First we want all the code that relates specifically to the JDK version to be isolated.  In JBoss Modules, I've established a group of classes, including one key class called <code>JDKSpecific</code>, which contains the bulk of the differences.  You can see this class at <a href="https://github.com/jboss-modules/jboss-modules/blob/64cae3ca527d881cd7123e5f86624e89fc51c864/src/main/java/org/jboss/modules/JDKSpecific.java">this link</a> to get an idea of what is in there.  Create a commit containing this code, and push up and release a tag.

#### Create a new Maven project for the Java 9 supplement

The next step is to create a new Maven project where the Java 9 parts will be built.  For JBoss Modules we wind up with a project that has a POM that looks <a href="https://github.com/jboss-modules/jboss-modules-jdk9/blob/master/pom.xml">like this</a>.  The project POM has a single dependency on the primary source base, and the only source files present are replacements for the JDK-specific class(es) established in the previous step.  Note that you will ultimately need a stable build to reference, which is why I recommend pushing up a tag in the previous step.

Once you have a basic implementation, create a commit, and push up and release a tag for this new project.  I recommend keeping a separate version stream for this part as it simplifies maintenance later on.

#### Bring the supplement in to the main code

Maven has a lot of functionality that is not well-explained by documentation but is nevertheless exceedingly useful.  One such bit of functionality exists in the form of the Maven Dependency Plugin.  Using this plugin, we can unroll the supplement project right into our final JAR in one simple step.  The critical piece of the POM is the configuration of this plugin:

```xml
    <plugin>
        <artifactId>maven-dependency-plugin</artifactId>
        <executions>
            <execution>
                <id>add-java9-supplement</id>
                <phase>generate-resources</phase>
                <goals>
                    <goal>unpack-dependencies</goal>
                </goals>
                <configuration>
                    <includeGroupIds>org.jboss.modules</includeGroupIds>
                    <includeArtifactIds>jboss-modules-jdk9-supplement</includeArtifactIds>
                    <excludeTransitive>true</excludeTransitive>
                    <includes>**/*.class</includes>
                    <outputDirectory>${project.build.directory}/generated-resources/META-INF/versions/9</outputDirectory>
                </configuration>
            </execution>
        </executions>
    </plugin>
```

Make sure you include the resources:

```xml
    <build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
            </resource>
            <resource>
                <directory>target/generated-resources</directory>
                <filtering>false</filtering>
            </resource>
        </resources>
        <!-- ...continued... -->
    </build>
```

To avoid circular build problems, it is a good idea to have an exclusion in the dependency declaration of the supplement:

```xml
    <dependency>
        <groupId>org.jboss.modules</groupId>
        <artifactId>jboss-modules-jdk9-supplement</artifactId>
        <version>1.0.Final</version>
        <scope>provided</scope>
        <exclusions>
            <exclusion>
                <groupId>org.jboss.modules</groupId>
                <artifactId>jboss-modules</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
```

## Conclusion

I think this is about as simple as a thing like this can be in Maven.  All of my requirements and all of my goals are solidly met with this approach and so far the result is looking pretty good.

If you try this out, let me know how it goes in the comments!
