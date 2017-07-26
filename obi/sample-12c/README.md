# Checkmate for OBI Quickstart
This quickstart provides basic understanding of the core features of Checkmate for OBI 8.0.2, using version 12.2.1.2 of OBIEE. The project folder includes sample OBIEE content from [SampleAppLite](http://docs.oracle.com/middleware/12212/biee/BIESG/GUID-E439E473-DD4D-48FE-9BF1-7AED4ADD73B6.htm#BIESG9340) already checked into the [`src/main`](src/main) directory.

Checkmate is built using the [Gradle](www.gradle.org): a declarative, DSL-based build tool most commonly associated with building JVM-based software. Specifically, Checkmate is a series of [Gradle Plugins](https://guides.gradle.org/designing-gradle-plugins/) with the core OBI functionality existing in the [com.redpillanalytics.checkmate.obi](https://plugins.gradle.org/plugin/com.redpillanalytics.checkmate.obi) plugin that introduces the following features for Oracle Business Intelligence: source control integration, content versioning and publishing, automated regression and integration testing, and automated deployments.

# Vanilla Configuration
We only need a few parameters to get a vanilla configuration of Checkmate for OBI. Most Gradle configurations exist in the `build.gradle` file, which can exist anywhere in the build filesystem, but we usually put it in the plugin directory or the project directory: in this quickstart, it exists in the project directory. This repository already contains a vanilla `build.gradle` file, with several of the advanced features that you will apply later commented out.

For this quickstart, the `build.gradle` file is in the project directory. The very first thing is

```gradle
plugins {
  id 'com.redpillanalytics.checkmate.obi' version '8.0.2'
  id 'maven-publish'
}
```

The `plugins` block applies any desired plugins from the [Gradle Plugin Portal](https://plugins.gradle.org) using the unique ID associated with that plugin.

* `com.redpillanalytics.checkmate.obi`: This is the Checkmate for OBI plugin.
* `maven-publish`: a Core Gradle plugin that enables publishing to Maven repositories. Checkmate for OBI uses Maven Publish to publish distributions of OBI content.

This is the only configuration required to get a basic Checkmate for OBI skeleton working: a Gradle [project with several callable tasks](https://docs.gradle.org/3.5/userguide/tutorial_using_tasks.html#sec:projects_and_tasks). We can use the [Gradle Wrapper](https://docs.gradle.org/3.5/userguide/gradle_wrapper.html) checked in to this repository to see all the tasks associated with the project directory, specified with the `-p` option, and the `tasks` command:

```gradle
./gradlew -p obi/sample-12c tasks
```

The first time any command is executed, Gradle will pull down any library dependencies used by Checkmate for OBI from the central Maven repository called [Bintray jCenter](https://bintray.com/bintray/jcenter), including the Gradle distribution itself.

The Checkmate for OBI enables the following Task groups:
* **Analytics**: Tasks associated with loading data generated from Checkmate builds to downstream data platforms. Will be configured later.
* **Distribution**: Managing the creation and deletion of different types of OBIEE distribution types.
* **Export**: Tasks that facilitate exporting content from an OBIEE instance into the build location or into source control.
* **Import**: Tasks that facilitate importing content into an OBIEE instance, usually using artifacts built in the build location, or downloaded from Maven.
* **SCM**: Tasks for integrating with Source Control Management, specifically [Git](https://git-scm.com).
* **Services**: Just the one `metadataReload` task, which executes *Reload Files and Metadata* in OBIEE.
* **Testing**: Tasks for Regression Testing OBIEE. We haven't configured any Test Groups yet, so the existing tasks are shell tasks for when we configure this later on.

The Maven Publish plugin enables the following Task groups:
* **Publishing**: Publishing content to Maven repositories. The only publishing location currently configured is Maven Local, which depending on environment variables is usually $HOME/.m2. Additional Maven locations can be configured using targets such as Apache Archiva, JFrog Artifactory, or AWS S3.

Gradle enables certain default Task groups as well:
* **Build Setup**: For generating new projects, the Gradle Wrapper, etc.
* **Help**: Basic help tasks.
* **Verification**: Runs any configured checks enabled in the project.

# Checkmate Environment Setup
We need to setup the basics about our OBIEE 12.2.1.2 environment so Checkmate knows how and where to execute some of these tasks. We use Checkmate for OBI **build parameters** to enable this. Build parameters can be enabled one of four ways, in reverse-prioritized order... meaning the last item in the list overrides the second-to-the-last item, and so forth:
* Specified in the `build.gradle` file
* Specified in a `gradle.properties` file, which is a standard Java properties file existing in the project directory
* Specified with environment variables
* Specified using Gradle project properties, which are passed to with the Gradle Wrapper command-line using `-P<property>=<value>`

Checkmate provides this degree of flexibility because many build properties are environment specific, and would need to change from one environment to the next. Additionally, some parameters--such as passwords--are sensitive, and need to be treated as such. For instance, [Jenkins Credentials](https://jenkins.io/doc/book/pipeline/syntax/#environment), which are typically used to store passwords in Continuous Delivery environments, are exposed as environment variables, so this is a very handy way to pass sensitive build parameters to Checkmate for OBI.

For the sake of simplictiy and clarity, we'll declare all the build parameters in the `build.gradle` file... even the sensitive ones. Just remember... you would want to use another approach in a real delivery pipeline.

```gradle
obi.middlewareHome = '/home/oracle/fmw/product/12.2.1.2/obi1'
obi.domainHome = '/home/oracle/fmw/config/domains/bi'
obi.compatibility = '12.2.1.2'
obi.adminUser = 'weblogic'
obi.adminPassword = 'Admin123'
obi.repositoryPassword = 'Admin123'
obi.publishBar = false
```

Notes on a few of the parameters below:
* **obi.domainHome:** Defaults to `<obi.middlewareHome>/user_projects/domains/bi`
* **obi.compatibility:** Options are 12.2.1.2, 12.2.1.1, 12.2.1.0, 11.1.1.9 and 11.1.1.7
* **obi.publishBar:** When we publish content (we'll see this in practice shortly), if `obi.publishBar = true`, then Checkmate for OBI will generate the 12c BAR file using the environment specified with the details above. We'll enable this in a bit, but not yet.