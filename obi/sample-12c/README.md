# Checkmate for OBI Quickstart
This quickstart demonstrates the basic functionality of Checkmate for OBI 8.0.2, using version 12.2.1.2 of OBIEE. The project folder includes sample OBIEE content from [SampleAppLite](http://docs.oracle.com/middleware/12212/biee/BIESG/GUID-E439E473-DD4D-48FE-9BF1-7AED4ADD73B6.htm#BIESG9340) already checked into the [`src/main`](src/main) directory.

Checkmate is built using [Gradle](www.gradle.org): a declarative, DSL-based build tool most commonly associated with building JVM-based software. Specifically, Checkmate is a series of [Gradle Plugins](https://guides.gradle.org/designing-gradle-plugins/) with the OBI functionality existing in the [com.redpillanalytics.checkmate.obi](https://plugins.gradle.org/plugin/com.redpillanalytics.checkmate.obi) plugin that introduces the following features for Oracle Business Intelligence: source control integration, content versioning and publishing, automated regression and integration testing, and automated deployments.

# Vanilla Configuration
We only need a few parameters to get a vanilla configuration of Checkmate for OBI. Most Gradle configurations exist in the `build.gradle` file, which can exist anywhere in the build filesystem, but we usually put it in the plugin directory or the project directory: in this quickstart, it exists in the project directory. This repository already contains a vanilla [`build.gradle`](build.gradle) file, with several of the advanced features that you will apply later commented out.

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
* **Publishing**: Publishing content to Maven repositories.

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

# Building and Publishing
The workflow for building OBIEE content usually occurs in the following steps:
* **Build:** The building of OBIEE deployment artifacts from source control. In source control we have the following checked in: the metadata repository as MDS-XML, and the presentation catalog in filesystem structure. The **build** steps involves building a binary RPD, as well as a catalog archive file of the Shared Folders of the catalog.
* **Bundle:** Building **distribution files** of all OBIEE content generated in the **Build** phase. In effect, these are zip files containing repository and catalog content.
* **Publish:** Publishing distribution files to one or more [Maven repositories](https://maven.apache.org/pom.html#Repositories).

Currently, we have the following publications configured in our `build.gradle`:

```gradle
publishing {
  repositories {
    // publish to Maven local repository, which is usually ~/.m2
    // This is only for testing purposes
    mavenLocal()
  }
}
```

This will only publish our OBI distribution files to the Maven Local repository, which defaults to `$HOME/.m2`. Obviously, this is for testing purposes only. In a real continuous delivery pipeline, content should be published to a real Maven-compatible repository. At the very least, publish to an S3 bucket, which Gradle supports, using the following syntax:

```gradle
publishing {
  repositories {
    maven {
      url 's3://bucket-name/path/to/directory'
      credentials(AwsCredentials) {
        accessKey 'access key'
        secretKey 'secret key'
      }
    }
  }
}
```

Why do we bother publishing our OBIEE distributions to Maven repositories? So we can pull that distribution later by simply referring to the version number. This allows us to automate deployments to downstream environments, as Checkmate for OBI can simply pull the distribution file from Maven prior to deployment. It also allows us to automate regression testing. As you will see later in the Quickstart, we can declare published distribution versions as baselines for testing new builds generated by Checkmate for OBI. This gives us a convenient way to pull down any prior distribution for regression testing or integration testing purposes. We can also generate incremental patch files this way: we simply pull down from Maven whatever distribution was last published to our **Production** server, and use that in the patch-creation process.

To build our OBI project, we can simply execute the following:

```gradle
./gradlew -p obi/sample-12c build
```

You'll notice that the `build` task doesn't really do anything on it's own: it's really just a container for two other tasks that do all the work: `metadataBuild` and `catalogBuild`. This introduces Gradle's powerful dependencies and ordering features, which uses a [DAG](https://docs.gradle.org/3.5/userguide/build_lifecycle.html) implementation.

Furthermore... we can run the entire Build, Bundle and Publish workflow by simply running the `publish` task, which has dependencies on building and bundling all the content.

```gradle
./gradlew -p obi/sample-12c publish
```

In the output, you'll notice the *up-to-date* checks that Checkmate for OBI is doing when running the dependent tasks. This can be seen in the following output:

```
:catalogBuild UP-TO-DATE
:metadataBuild UP-TO-DATE
:build UP-TO-DATE
```

Checkmate for OBI is written to take advantage of the [Gradle Incremental Build](https://docs.gradle.org/3.5/userguide/more_about_tasks.html#sec:up_to_date_checks) feature. The catalog and metadata build tasks are not executed again, because none of the task input and output files have changed. This keeps Checkmate from re-running tasks that it doesn't have to.

Let's take a look at what our Build, Bundle and Publish process generated. If we look at the `build` directory in the project directory, we can see all the things that Checkmate for OBI built, including some of the following:

```
ls -l catalog

catalog:
total 1820
drwxr-xr-x. 1 501 games     136 Jul 25 23:58 current
-rw-r-----. 1 501 games 1863215 Jul 25 23:58 current.catalog

repository:
total 28
-rwxr-----. 1 501 games 28456 Jul 25 23:58 current.rpd
drwxr-xr-x. 1 501 games    68 Jul 25 23:58 xml-variables

distributions:
total 8480
-rw-r--r--. 1 501 games 4339941 Jul 26 00:07 sample-12c-build-0.0.9.zip
-rw-r--r--. 1 501 games 4339941 Jul 26 00:07 sample-12c-deploy-0.0.9.zip
```

Additionally, we can look at our Maven Local repository, and see the versioned distribution files published there. You can explore the directory structure further to get a flavor for the Maven filesystem:

```
ls -l ~/.m2/repository/obiee
total 8
drwxrwxr-x. 4 oracle oracle 4096 Jul 19 10:18 sample-12c-build
drwxrwxr-x. 4 oracle oracle 4096 Jul 19 10:18 sample-12c-deploy
```

Later on, we'll expore the differences between the **build** and the **deploy** distributions. For now... just know that **build** is a subset of **deploy**.