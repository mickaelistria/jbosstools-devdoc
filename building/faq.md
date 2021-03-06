## How do I configure my settings.xml?

Follow these instructions to add reference to JBoss Repositories into your settings.xml. You'll also probably need access to the SNAPSHOT repository. So here is what you should see in your `~/.m2/settings.xml`

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
   ....
    <profiles>
        ....
        <profile>
            <id>jboss-default</id>
            <repositories>
                <!-- To resolve parent artifact -->
                <repository>
                    <id>jboss-public-repository-group</id>
                    <name>JBoss Public Repository Group</name>
                    <url>http://repository.jboss.org/nexus/content/groups/public/</url>
                </repository>
                <repository>
                    <id>jboss-snapshots-repository</id>
                    <name>JBoss Snapshots Repository</name>
                    <url>https://repository.jboss.org/nexus/content/repositories/snapshots/</url>
                </repository>
            </repositories>

            <pluginRepositories>
                        <!-- To resolve parent artifact -->
                        <pluginRepository>
                                <id>jboss-public-repository-group</id>
                                <name>JBoss Public Repository Group</name>
                                <url>http://repository.jboss.org/nexus/content/groups/public/</url>
                        </pluginRepository>
                        <pluginRepository>
                                <id>jboss-snapshots-repository</id>
                                <name>JBoss Snapshots Repository</name>
                                <url>https://repository.jboss.org/nexus/content/repositories/snapshots/</url>
                        </pluginRepository>
                </pluginRepositories>
        </profile>
    </profiles>

    <activeProfiles>
        <activeProfile>jboss-default</activeProfile>
        ...
    </activeProfiles>
</settings>
```
 
## My build is failing due to OutOfMemory or PermGen issues! How do I give Maven more memory?

To configure the amount of memory used by Maven, you can define MVN_OPTS as follows, either in the mvn / mvn.bat script you use to run Maven, or set as global environment variables. Here's how to do so for Fedora, Ubuntu, Windows, OSX.

```bash
set MAVEN_OPTS=-Xms512m -Xmx1024m -XX:PermSize=128m -XX:MaxPermSize=256m
```
 
##  How do I build via commandline?
 
1. Fetch code from github
2. Run `mvn`
3. Repeat for other projects

```bash
cd ~/jbosstools
git clone https://github.com/jbosstools/jbosstools-base
cd ~/jbosstools/jbosstools-base
mvn verify
```

## How do I build in Eclipse (with m2e)?

To be written

## How do I build a single project?

Generally, you need only fetch the sources, and run `mvn verify`. See 'How do I build via commandline?' or 'How do I build in Eclipse (with m2e)?' above.

## How do I build a series of projects (eg., Base, Server, Webservices) ?

Assuming your goal is to change sources in multiple projects, then build them all to verify everything still compiles, you will need to fetch sources, then build the projects in dependency order.

This workflow is useful when you're looking to change framework code in an upstream project like Base on which Webservices or Server depends, and verify that API changes don't break the downstream project's code.

```bash
cd ~/jbosstools
git clone https://github.com/jbosstools/jbosstools-base
git clone https://github.com/jbosstools/jbosstools-server
git clone https://github.com/jbosstools/jbosstools-webservices
```

### Orchestrate builds manually, or
Rather than using `mvn verify`, you must use `mvn install` so that the downstream build will use YOUR locally-built content in your local repository (found inside `~/.m2`), rather than looking to remote repositories for the last published upstream dependencies.
```bash
cd ~/jbosstools/jbosstools-base; mvn install
cd ~/jbosstools/jbosstools-server; mvn install
cd ~/jbosstools/jbosstools-webservices; mvn verify
```
### Create an orchestrator pom

TODO

## Which is better - `mvn clean install` or `mvn verify` ?

Use _install_ to create artifacts in your ~/.m2 folder which will be checked when building anything which depends on those artifacts (in addition to checking the internet). Also useful for doing subsequent offline builds with the `-o` or `--offline` flag.
Use `verify` to build, but NOT install anything into your `~/.m2` folder.

## How do I clean out artifacts I might have installed to my ~/.m2/repo ?

 

## What if I've already built something locally, but I want to build against the server version instead of my local repo?

There are two approaches that work here:
* override temporarily when building, using -Dtycho.localArtifacts=ignore, or
* delete ~/.m2/repository/.meta/p2-local-metadata.properties

## How do I build a target platform?
Currently, you should not need to build a target platform locally if you're building w/ maven via commandline. But if you'd like to materialize a target platform for use in Eclipse, you can do so like this:
1. Fetch target platforms project from github
2. Switch to the correct branch
3. Run maven

Results will be in jbosstools/multiple/target/jbosstools-multiple.target.repo/ or jbdevstudio/multiple/target/jbdevstudio-multiple.target.repo/

```xml
cd ~/jbosstools
git clone https://github.com/jbosstools/jbosstools-target-platforms
cd ~/jbosstools/jbosstools-target-platforms
git checkout 4.3.0
mvn install
cd ~/jbosstools/jbosstools-target-platforms/jbosstools/multiple/target/jbosstools-multiple.target.repo/
```

## Why is there more than one target platform?

Every time we make changes to the target platform, either to add/remove something, or to change the included version, we release a new version. 

In order to verify we can build against the oldest version of a target platform (eg., one based on Eclipse 4.2.0, or "minimum" target platform) but also run tests against the latest for that stream (eg., based on Eclipse 4.2.2, or "maximum" target platform), we need to maintain multiple versions.

By default, your build will use the default "minimum" target platform specified in the JBoss Tools parent pom. To easily build against the default "maximum", use -Pmaximum. See also 'What profiles do I need to build? What Maven properties are useful when building?'

## How do I specify which target platform to use when building?

See 'What profiles do I need to build? What Maven properties are useful when building?'

## How to I skip running  tests? How do I make tests not fail? Or only fail after ALL tests run?

To skip running tests, you can use these Maven flags:
* `-Dmaven.test.skip=true` (also skip compilation)
* `-DskipTests` (recommended)

If your reason for skipping tests is to see if everything can run without being stuck on the first test failure, you might also like these flags:
* `-fae`, `--fail-at-end` : Fail at end of build only
* `-fn`, `--fail-never` : Never fail the build regardless of result

You can also cause test failures to result in JUnit output without failing the build using these flags:
* `-Dmaven.test.error.ignore=true`
* `-Dmaven.test.failure.ignore=true`

## How can I debug tests in Eclipse when run from Tycho (with Surefire)?

See http://www.jboss.org/tools/docs/testing.html

## How do I build docs?

See Building JBoss Tools Documentation.

## What profiles do I need to build? What Maven properties are useful when building?

Most of the time, you don't need any profiles or -D properties. Here are some profiles and properties you might want to use in special cases.
* `-Pmaximum` : selects the default maximum target platform version instead of the default minimum one. Useful when running tests to verify that your code works against a newer target platform (eg., Eclipse 4.2.2 instead of 4.2.0)
* `-DTARGET_PLATFORM_VERSION` : allows you to pick a specific target platform version from those available in Nexus. 

See also 'How to I skip running  tests? How do I make tests not fail? Or only fail after ALL tests run?' above for test-related properties.