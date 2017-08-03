# Checkmate for OBI Quickstart
This Quickstart demonstrates the basic functionality of the Checkmate for OBI Build Framework, using version 12.2.1.2 of Oracle Business Intelligence. The project folder includes sample OBIEE content from [SampleAppLite](http://docs.oracle.com/middleware/12212/biee/BIESG/GUID-E439E473-DD4D-48FE-9BF1-7AED4ADD73B6.htm#BIESG9340) already checked into the [`src/main`](src/main) directory. All you need to use this Quickstart is an OBIEE 12.2.1.2 environment. We are assuming you are using Linux, so adjust commands if using Windows.

Checkmate is built using [Gradle](https://www.gradle.org): a declarative, DSL-based build tool most commonly associated with building JVM-based software. Specifically, Checkmate is a series of [Gradle Plugins](https://guides.gradle.org/designing-gradle-plugins/). The following OBI functionality exists in the [com.redpillanalytics.checkmate.obi](https://plugins.gradle.org/plugin/com.redpillanalytics.checkmate.obi) plugin: source control integration, content versioning and publishing, automated regression and integration testing, and automated deployments.

# Basic Configuration
This Quickstart uses the standard apporach of configuring Checkmate using a [`build.gradle`](https://docs.gradle.org/current/userguide/tutorial_using_tasks.html#sec:hello_world) file in the project directory, but there are options for placing hierachical `build.gradle` files throughout the build filesystem. This repository already contains a functioning [`build.gradle`](build.gradle) file, with all the necessary configurations already made, with several of the advanced features that we will apply later commented out.

The `plugins` block is the first and most important aspect to the build script: it applies any desired plugins from the [Gradle Plugin Portal](https://plugins.gradle.org) using the unique ID associated with that plugin:

```groovy
plugins {
  id 'com.redpillanalytics.checkmate.obi' version '8.0.6'
  id 'maven-publish'
}
```

* `com.redpillanalytics.checkmate.obi`: This is the Checkmate for OBI plugin.
* `maven-publish`: a Core Gradle plugin that enables publishing to Maven repositories. Checkmate for OBI uses Maven Publish to publish distributions of OBI content.

With the Checkmate for OBI plugin applied, our Gradle [project](https://docs.gradle.org/current/userguide/tutorial_using_tasks.html#sec:projects_and_tasks) exists with all the core tasks enabled. We can use the [Gradle Wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html) checked in to this repository to see all the tasks associated with the project directory, specified with the `-p` option, and the `tasks` command. If we comment everything else out of the `build.gradle` file except the `plugins` block, the `tasks` command gives us the following:

```bash
./gradlew -p obi/sample-12c tasks --console=plain
:tasks

------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Analytics tasks
---------------
analytics - Analytics workflow task for processing all configured analytics jobs.

Build tasks
-----------
assemble - Assembles the outputs of this project.
build - Execute 'metadataBuild' and 'catalogBuild'
catalogBuild - Copy catalog from SCM to 'catalog/current' and generate catalog archive file 'catalog/current.catalog'.
clean - Deletes the build directory.
metadataBuild - Build binary repository 'repository/current.rpd' from the MDS-XML repository in SCM.

Build Setup tasks
-----------------
init - Initializes a new Gradle build.
wrapper - Generates Gradle wrapper files.

Distribution tasks
------------------
buildZip - Create a ZIP distribution archive for downstream deployments containing metadata and catalog artifacts.
cleanDist - Delete the Distributions directory.
deployZip - Create a ZIP distribution archive for downstream deployments containing metadata and catalog artifacts and incremental patches.

Export tasks
------------
barExport - Build BI Archive (BAR) File using the 'ssi' Service Instance.
catalogExport - Export the online presentation catalog and synchronize with the offline presentation catalog in SCM.
connPoolsExport - Export target OBIEE server connection pool information in JSON format to 'repository/conn-pools.json'.
export - Execute 'catalogExport' and 'metadataExport'.
metadataDownload - Download the online metadata repository to 'repository/current.rpd'.
metadataExport - Synchronize 'repository/current.rpd' with the MDS-XML repository in SCM.
variablesExport - Export target OBIEE server variable information in JSON format to 'repository/variables.json'.

Help tasks
----------
buildEnvironment - Displays all buildscript dependencies declared in root project 'sample-12c'.
components - Displays the components produced by root project 'sample-12c'. [incubating]
dependencies - Displays all dependencies declared in root project 'sample-12c'.
dependencyInsight - Displays the insight into a specific dependency in root project 'sample-12c'.
dependentComponents - Displays the dependent components of components in root project 'sample-12c'. [incubating]
displayConfigurations - Display information about existing configurations
help - Displays a help message.
model - Displays the configuration model of root project 'sample-12c'. [incubating]
projects - Displays the sub-projects of root project 'sample-12c'.
properties - Displays the properties of root project 'sample-12c'.
tasks - Displays the tasks runnable from root project 'sample-12c'.

Import tasks
------------
barImportSAL - Import SampleAppLite.bar BI Archive (BAR) File into the 'ssi' Service Instance.
barReset - Reset the 'ssi' Service Instance, equivalent to using an empty BI Archive (BAR) File.
catalogImport - Import the presentation catalog from SCM into the online presentation catalog using 'catalog/current', and Reload Files and Metadata.
connPoolsImport - Import server connection pool information in JSON format from 'repository/conn-pools.json' to the target OBIEE server.
import - Execute 'catalogImport' and 'metadataImport'.
metadataImport - Import 'repository/current.rpd' into the online metadata repository, and Reload Files and Metadata.
variablesImport - Import server variable information in JSON format from 'repository/variables.json' to the target OBIEE server.

Publishing tasks
----------------
generatePomFileForBuildPublication - Generates the Maven POM file for publication 'build'.
generatePomFileForDeployPublication - Generates the Maven POM file for publication 'deploy'.
publish - Publishes all publications produced by this project.
publishBuildPublicationToMavenLocal - Publishes Maven publication 'build' to the local Maven repository.
publishBuildPublicationToMavenLocalRepository - Publishes Maven publication 'build' to Maven repository 'MavenLocal'.
publishDeployPublicationToMavenLocal - Publishes Maven publication 'deploy' to the local Maven repository.
publishDeployPublicationToMavenLocalRepository - Publishes Maven publication 'deploy' to Maven repository 'MavenLocal'.
publishToMavenLocal - Publishes all Maven publications produced by this project to the local Maven cache.

SCM tasks
---------
catalogSCM - Synchronize 'catalog/current' with SCM and then commit.
metadataSCM - Synchronize 'repository/current.rpd' with SCM and then commit.
scmCheckout - Checkout a branch in the local SCM repository.
scmCommit - Issue a commit to the local SCM repository. Customize with 'scmComment', 'scmAuthor', 'scmEmail', and 'scmCommitPath' build parameters.
scmPush - Push to the origin for the local SCM repository. Requires 'sourceBase' build parameter pointing to Git repo root directory, if running task outside of a Git repository.

Services tasks
--------------
metadataReload - Execute the 'Reload Files and Metadata' option.

Testing tasks
-------------
baselineTest - Execute all Baseline regression tests for the entire project.
compareTest - Execute all Compare regression tests for the entire project.
extractTestSuites - Extract the compiled test suites and copy them to the 'build/classes' directory.
regressionBaselineLibrary - Create the Baseline regression test library CSV file for Test Group 'regression'.
regressionBaselineTest - Execute the Baseline regression test library file for Test Group 'regression'.
regressionCompareTest - Compare the differences between the Baseline and Revision regression test results for Test Group 'regression'.
regressionRevisionLibrary - Create the Revision regression test library CSV file for Test Group 'regression'.
regressionRevisionTest - Execute the Revision regression test library file for Test Group 'regression'.
revisionTest - Execute all Revision regression tests for the entire project.

Verification tasks
------------------
check - Runs all checks.

Rules
-----
Pattern: clean<TaskName>: Cleans the output files of a task.
Pattern: build<ConfigurationName>: Assembles the artifacts of a configuration.
Pattern: upload<ConfigurationName>: Assembles and uploads the artifacts belonging to a configuration.

To see all tasks and more detail, run gradlew tasks --all

To see more detail about a task, run gradlew help --task <task>

BUILD SUCCESSFUL in 1s
1 actionable task: 1 executed
```

The first time a command is executed, Gradle will pull down any library dependencies used by Checkmate for OBI from the central Maven repository called [Bintray jCenter](https://bintray.com/bintray/jcenter), including the Gradle distribution itself.

A quick note on the `--console=plain` option. Gradle detects whether the CLI is being executed interactively, or by daemon processes, such as Continuous Delivery servers. You will generally want to run with `--console=auto` (the default) so it detects this, as the interactive capabilities are quite powerful. However... the interactive option is not great for a Quickstart, as the task executions are not displayed in full at the end. We suggest that you not add the `--console=plain` option in everyday use... but feel free to do it here to work through the Quickstart.

The Checkmate for OBI enables the following Task groups:
* **Analytics**: Tasks associated with loading data generated from Checkmate builds to downstream data platforms. We won't be looking at the Analytics functionality in this Quickstart.
* **Distribution**: Managing the creation and deletion of different types of OBIEE distribution files.
* **Export**: Tasks that facilitate exporting content from an OBIEE instance into the build location or into source control.
* **Import**: Tasks that facilitate importing content into an OBIEE instance, usually using artifacts built in the build location, or downloaded from Maven.
* **SCM**: Tasks for integrating with Source Control Management, specifically [Git](https://git-scm.com).
* **Services**: Just the one `metadataReload` task, which executes *Reload Files and Metadata* in OBIEE.
* **Testing**: Tasks for Regression Testing OBIEE. We describe this in detail later on.

The Maven Publish plugin enables the following Task groups:
* **Publishing**: Publishing content to Maven repositories.

Gradle enables certain default Task groups as well:
* **Build Setup**: For generating new projects, the Gradle Wrapper, etc.
* **Help**: Basic help tasks.
* **Verification**: Runs any configured checks enabled in the project.

# Checkmate Environment Setup
Checkmate for OBI needs to know the basics about the OBIEE environment it will execute against. Keep in mind: with the Gradle Wrapper in the source control repository, we can use Checkmate for OBI on any environment where we can check out a Git repository without doing an install. We use Checkmate **build parameters** to configure  properties for the OBIEE environment, as well as other things we'll see later.

Build parameters can be enabled one of four ways, in reverse-prioritized order... meaning the last item in the list overrides the second-to-the-last item, and so forth:
* Specified in the `build.gradle` file
* Specified in a `gradle.properties` file, which is a standard Java properties file existing in the project directory
* Specified with environment variables
* Specified using Gradle project properties, which are passed to with the Gradle Wrapper command-line using `-P<property>=<value>`

Checkmate provides this degree of flexibility because many build properties are environment specific, and may need to change from one environment to the next. Additionally, some parameters--such as passwords--are sensitive, and need to be treated as such. For instance, [Jenkins Credentials](https://jenkins.io/doc/book/pipeline/syntax/#environment), which are typically used to store passwords in Continuous Delivery environments, are exposed as environment variables, so this is a very handy way to pass sensitive build parameters to Checkmate for OBI.

For the sake of simplicity and clarity, we'll declare all the build parameters in the `build.gradle` file... even the sensitive ones. Just remember... you would want to use another approach in a real delivery pipeline.

```java
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


Additionally... we need to make all of the command-line tools that OBIEE 12c uses available to Checkmate for OBI. There are numerous ways to configure this, but the easiest is just to add the `$DOMAIN/bitools/bin` directory available to the `$PATH` environment variable:

```bash
export PATH=$PATH:/home/oracle/bin:/home/oracle/fmw/config/domains/bi/bitools/bin
```

# Building and Publishing
The workflow for building OBIEE content usually occurs in the following steps:
* **Build:** The building of OBIEE deployment artifacts from source control. In source control we have the following checked in: the metadata repository as MDS-XML, and the presentation catalog in filesystem structure. The **build** steps involve building a binary RPD, as well as a catalog archive file of the Shared Folders of the catalog.
* **Bundle:** Building **distribution files** of all OBIEE content generated in the **Build** phase. In effect, these are zip files containing repository and catalog content.
* **Publish:** Publishing distribution files to one or more [Maven repositories](https://maven.apache.org/pom.html#Repositories).

Currently, we have the following publications configured in our `build.gradle`:

```groovy
publishing {
  repositories {
    // publish to Maven local repository, which is usually ~/.m2
    // This is only for testing purposes
    mavenLocal()
  }
}
```

This will only publish our OBI distribution files to the [Maven Local](https://docs.gradle.org/3.5/userguide/publishing_maven.html#publishing_maven:install) repository, which defaults to `$HOME/.m2`. Obviously, this is for testing purposes only. In a real continuous delivery pipeline, content should be published to a real Maven-compatible repository. At the very least, publish to an S3 bucket, which Gradle supports, using the following syntax:

```groovy
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

```bash
./gradlew -p obi/sample-12c build --console=plain
:assemble UP-TO-DATE
:catalogBuild
:check UP-TO-DATE
:metadataBuild
:build

BUILD SUCCESSFUL in 21s
2 actionable tasks: 2 executed
```

You'll notice that the `build` task doesn't really do anything on its own: it's really just a container for two other tasks that do all the work: `metadataBuild` and `catalogBuild`. This introduces Gradle's powerful dependencies and ordering features, which uses a [DAG](https://docs.gradle.org/3.5/userguide/build_lifecycle.html) implementation.

Furthermore... we can run the entire Build, Bundle and Publish workflow by simply running the `publish` task, which has dependencies on building and bundling all the content.

```bash
./gradlew -p obi/sample-12c publish --console=plain
:assemble UP-TO-DATE
:catalogBuild UP-TO-DATE
:check UP-TO-DATE
:metadataBuild UP-TO-DATE
:build UP-TO-DATE
:buildZip
:generatePomFileForBuildPublication
:publishBuildPublicationToMavenLocalRepository
:deployZip
:generatePomFileForDeployPublication
:publishDeployPublicationToMavenLocalRepository
:publish

BUILD SUCCESSFUL in 8s
8 actionable tasks: 6 executed, 2 up-to-date
```

In the output, you'll notice the *UP-TO-DATE* checks that Checkmate for OBI is doing when running the dependent tasks. Checkmate is written to take advantage of the [Gradle Incremental Build](https://docs.gradle.org/3.5/userguide/more_about_tasks.html#sec:up_to_date_checks) feature. The catalog and metadata build tasks are not executed again, because none of the task input and output files have changed. This keeps Checkmate from re-running tasks that it doesn't have to. Rerunning tasks can always be forced by providing the `--rerun-tasks` command-line option.

Let's take a look at what our Build, Bundle and Publish process generated. If we look at the [`build`](build) directory in the project directory, we can see all the things that Checkmate for OBI built, including some of the following:

```bash
ls -l obi/sample-12c/build/*

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

# Build Groups
Checkmate for OBI uses the concept of a **build group**: a collection of tasks associated with a particular dependency, which in our case, is a dependency on a published distribution file of OBI content. Up until now, all the tasks demonstrated exist without being tied to a dependency: they are the core Checkmate for OBI tasks that revolve around working with content checked into a Git repository.

A build group allows us to declare a dependency on a prior release of a distribution file, and then get a bunch of new, dynamically generated tasks that belong to that build group. Checkmate contains three build groups by default:
* **feature:** used primarily to regression test new feature branches prior to their being merged into a mainline of code, usually the **develop** or **master** branch.
* **release:** used to regression test new releases prior to being deployed to downstream environments, or prior to being merged into release branches such as **master** or **release.x.x.x**.
* **promote:** used for promoting content to downstream environments.

Because these build groups are already built-in, we don't have to do much to enable them; all we have to do is declare a dependency on a prior distribution version, and the tasks in this build group will magically appear. Since we now have the 0.0.9 distribution published to our Maven Local repository, we can use that distribution as our dependency for these build groups. We could also define custom build groups using the `obi.buildGroups {}` DSL structure if for some reason the combination of **feature**, **release** and **promote** was not sufficient for our workflow.

The first thing we have to tell Checkmate for OBI is where to go looking for our prior distribution files. We're still using Maven Local for this:

```gradle
repositories {
  // resolve local maven repository for published distributions, which is usually ~/.m2
  // this is really only for testing purposes
  mavenLocal()
}
```

We'll make the following changes to our [`build.gradle`](build.gradle) file, which should be possible by simply commenting out a few lines:

```groovy
dependencies {
  // Using the Checkmate Testing library which is recommended.
  obiee group: 'gradle.plugin.com.redpillanalytics', name: 'checkmate', version: '8.0.6'
  // You can also use Baseline Validation Tool
  // The installation needs to be available in one of your Maven repositories
  // If it exists, Checkmate will unzip and install it for you
  //obiee group: 'com.oracle', name: 'oracle-bvt', version: '12.2.1.0.0'

  // Dependencies on previous OBIEE builds
  // Used for building incremental patches, regression testing, and deployments
  feature group: 'obiee', name: 'sample-12c-build', version: '+'
  release group: 'obiee', name: 'sample-12c-build', version: '0.0.9'
  //promote group: 'obiee', name: 'sample-12c-deploy', version: '0.0.9'
  //promote group: 'obiee', name: 'sample-12c-bar', version: '0.0.9'
}
```

There's a lot more in the dependencies closure that we'll discuss later. For now, we'll just focus on the two build group dependencies that we want to uncomment out:

```groovy
feature group: 'obiee', name: 'sample-12c-build', version: '+'
release group: 'obiee', name: 'sample-12c-build', version: '0.0.9'
```

The DSL might be a bit confusing, because we are using Gradle's built-in dependency resolution functionality to resolve our OBI distribution files. Basically, we use a Gradle [configuration](https://docs.gradle.org/current/dsl/org.gradle.api.artifacts.Configuration.html) to declare dependencies on distribution files that we want Checkmate for OBI to pull down and unzip whenever we use one of the tasks in that build group. We are declaring a particular distribution file... in this case, the **build** distribution, with a particular version. Notice for the **feature** build group, we simply have a plus sign (+): this signifies to Checkmate for OBI that we simply want to pull down the most recent distribution file. After you uncomment these two dependencies, pay attention to the new tasks that are enabled:

```bash
./gradlew -p obi/sample-12c tasks --console=plain
:tasks

------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Analytics tasks
---------------
analytics - Analytics workflow task for processing all configured analytics jobs.

Build tasks
-----------
assemble - Assembles the outputs of this project.
build - Execute 'metadataBuild' and 'catalogBuild'
catalogBuild - Copy catalog from SCM to 'catalog/current/' and generate catalog archive file 'catalog/current.catalog'.
clean - Deletes the build directory.
featureCatalogCompare - Build incremental file 'catalog/feature-diff.txt' and 'catalog/feature-undiff.txt' using the 'feature' configuration.
featureCompare - Execute 'featureCatalogCompare' and 'featureMetadataCompare'.
featureMetadataCompare - Build incremental files 'repository/feature-patch.xml', 'repository/feature-unpatch.xml' and 'repository/feature-compare.csv' using the 'feature' configuration.
metadataBuild - Build binary repository 'repository/current.rpd' from the MDS-XML repository in SCM.
releaseCatalogCompare - Build incremental file 'catalog/release-diff.txt' and 'catalog/release-undiff.txt' using the 'release' configuration.
releaseCompare - Execute 'releaseCatalogCompare' and 'releaseMetadataCompare'.
releaseMetadataCompare - Build incremental files 'repository/release-patch.xml', 'repository/release-unpatch.xml' and 'repository/release-compare.csv' using the 'release' configuration.

Build Setup tasks
-----------------
init - Initializes a new Gradle build.
wrapper - Generates Gradle wrapper files.

Distribution tasks
------------------
buildZip - Create a ZIP distribution archive for downstream deployments containing metadata and catalog artifacts.
cleanDist - Delete the Distributions directory.
deployZip - Create a ZIP distribution archive for downstream deployments containing metadata and catalog artifacts and incremental patches.
featureExtractBuild - Extract OBIEE dependencies for 'feature'.
releaseExtractBuild - Extract OBIEE dependencies for 'release'.

Export tasks
------------
barExport - Build BI Archive (BAR) File using the 'ssi' Service Instance.
catalogExport - Export the online presentation catalog and synchronize with the offline presentation catalog in SCM.
connPoolsExport - Export target OBIEE server connection pool information in JSON format to 'repository/conn-pools.json'.
export - Execute 'catalogExport' and 'metadataExport'.
metadataDownload - Download the online metadata repository to 'repository/current.rpd'.
metadataExport - Synchronize 'repository/current.rpd' with the MDS-XML repository in SCM.
variablesExport - Export target OBIEE server variable information in JSON format to 'repository/variables.json'.

Help tasks
----------
buildEnvironment - Displays all buildscript dependencies declared in root project 'sample-12c'.
components - Displays the components produced by root project 'sample-12c'. [incubating]
dependencies - Displays all dependencies declared in root project 'sample-12c'.
dependencyInsight - Displays the insight into a specific dependency in root project 'sample-12c'.
dependentComponents - Displays the dependent components of components in root project 'sample-12c'. [incubating]
displayConfigurations - Display information about existing configurations
help - Displays a help message.
model - Displays the configuration model of root project 'sample-12c'. [incubating]
projects - Displays the sub-projects of root project 'sample-12c'.
properties - Displays the properties of root project 'sample-12c'.
tasks - Displays the tasks runnable from root project 'sample-12c'.

Import tasks
------------
barImportSAL - Import SampleAppLite.bar BI Archive (BAR) File into the 'ssi' Service Instance.
barReset - Reset the 'ssi' Service Instance, equivalent to using an empty BI Archive (BAR) File.
catalogImport - Import the presentation catalog from SCM into the online presentation catalog using 'catalog/current/', and Reload Files and Metadata.
connPoolsImport - Import server connection pool information in JSON format from 'repository/conn-pools.json' to the target OBIEE server.
featureCatalogImport - Import the presentation catalog from SCM into the online presentation catalog using 'catalog/feature', and Reload Files and Metadata.
featureImport - Execute 'featureCatalogImport' and 'featureMetadataImport'.
featureMetadataImport - Import 'repository/feature.rpd' into the online metadata repository, and Reload Files and Metadata.
import - Execute 'catalogImport' and 'metadataImport'.
metadataImport - Import 'repository/current.rpd' into the online metadata repository, and Reload Files and Metadata.
releaseCatalogImport - Import the presentation catalog from SCM into the online presentation catalog using 'catalog/release', and Reload Files and Metadata.
releaseImport - Execute 'releaseCatalogImport' and 'releaseMetadataImport'.
releaseMetadataImport - Import 'repository/release.rpd' into the online metadata repository, and Reload Files and Metadata.
variablesImport - Import server variable information in JSON format from 'repository/variables.json' to the target OBIEE server.

Patch tasks
-----------
featureCatalogPatch - Apply 'catalog/feature-diff.txt' to the online presentation catalog, and Reload Files and Metadata.
featureCatalogUnpatch - Apply 'catalog/feature-diff.txt' to the online presentation catalog.
featureMetadataPatch - Apply 'repository/feature-patch.xml' to the metadata repository in offline mode, and Reload Files and Metadata.
featureMetadataUnpatch - Apply 'repository/feature-unpatch.xml' to the metadata repository in offline mode
featurePatch - Execute 'featureCatalogPatch' and 'featureMetadataPatch'.
featureUnpatch - Execute 'featureCatalogUnpatch' and 'featureMetadataUnpatch'.
releaseCatalogPatch - Apply 'catalog/release-diff.txt' to the online presentation catalog, and Reload Files and Metadata.
releaseCatalogUnpatch - Apply 'catalog/release-diff.txt' to the online presentation catalog.
releaseMetadataPatch - Apply 'repository/release-patch.xml' to the metadata repository in offline mode, and Reload Files and Metadata.
releaseMetadataUnpatch - Apply 'repository/release-unpatch.xml' to the metadata repository in offline mode
releasePatch - Execute 'releaseCatalogPatch' and 'releaseMetadataPatch'.
releaseUnpatch - Execute 'releaseCatalogUnpatch' and 'releaseMetadataUnpatch'.

Publishing tasks
----------------
generatePomFileForBuildPublication - Generates the Maven POM file for publication 'build'.
generatePomFileForDeployPublication - Generates the Maven POM file for publication 'deploy'.
publish - Publishes all publications produced by this project.
publishBuildPublicationToMavenLocal - Publishes Maven publication 'build' to the local Maven repository.
publishBuildPublicationToMavenLocalRepository - Publishes Maven publication 'build' to Maven repository 'MavenLocal'.
publishDeployPublicationToMavenLocal - Publishes Maven publication 'deploy' to the local Maven repository.
publishDeployPublicationToMavenLocalRepository - Publishes Maven publication 'deploy' to Maven repository 'MavenLocal'.
publishToMavenLocal - Publishes all Maven publications produced by this project to the local Maven cache.

SCM tasks
---------
catalogSCM - Synchronize 'catalog/current/' with SCM and then commit.
featureCatalogMerge - Use OBIEE merging instead of SCM merging for presentation catalog.
featureMerge - Execute 'featureMetadataMerge' and 'featureCatalogMerge'.
metadataSCM - Synchronize 'repository/current.rpd' with SCM and then commit.
releaseCatalogMerge - Use OBIEE merging instead of SCM merging for presentation catalog.
releaseMerge - Execute 'releaseMetadataMerge' and 'releaseCatalogMerge'.
scmCheckout - Checkout a branch in the local SCM repository.
scmCommit - Issue a commit to the local SCM repository. Customize with 'scmComment', 'scmAuthor', 'scmEmail', and 'scmCommitPath' build parameters.
scmPush - Push to the origin for the local SCM repository. Requires 'sourceBase' build parameter pointing to Git repo root directory, if running task outside of a Git repository.

Services tasks
--------------
metadataReload - Execute the 'Reload Files and Metadata' option.

Testing tasks
-------------
baselineTest - Execute all Baseline regression tests for the entire project.
compareTest - Execute all Compare regression tests for the entire project.
extractTestSuites - Extract the compiled test suites and copy them to the 'build/classes' directory.
regressionBaselineLibrary - Create the Baseline regression test library CSV file for Test Group 'regression'.
regressionBaselineTest - Execute the Baseline regression test library file for Test Group 'regression'.
regressionCompareTest - Compare the differences between the Baseline and Revision regression test results for Test Group 'regression'.
regressionRevisionLibrary - Create the Revision regression test library CSV file for Test Group 'regression'.
regressionRevisionTest - Execute the Revision regression test library file for Test Group 'regression'.
revisionTest - Execute all Revision regression tests for the entire project.

Verification tasks
------------------
check - Runs all checks.

Workflow tasks
--------------
featureBaselineWorkflow - Import 'feature' metadata and catalog artifacts, manage connection pools, and execute the Baseline Test Library.
featureRevisionPatchWorkflow - Apply 'feature' metadata and catalog patches, manage connection pools, and execute the Revision Test Library.
featureRevisionWorkflow - Import 'feature' metadata and catalog artifacts, manage connection pools, and execute the Revision Test Library.
releaseBaselineWorkflow - Import 'release' metadata and catalog artifacts, manage connection pools, and execute the Baseline Test Library.
releaseRevisionPatchWorkflow - Apply 'release' metadata and catalog patches, manage connection pools, and execute the Revision Test Library.
releaseRevisionWorkflow - Import 'release' metadata and catalog artifacts, manage connection pools, and execute the Revision Test Library.

Rules
-----
Pattern: clean<TaskName>: Cleans the output files of a task.
Pattern: build<ConfigurationName>: Assembles the artifacts of a configuration.
Pattern: upload<ConfigurationName>: Assembles and uploads the artifacts belonging to a configuration.

To see all tasks and more detail, run gradlew tasks --all

To see more detail about a task, run gradlew help --task <task>

BUILD SUCCESSFUL in 1s
1 actionable task: 1 executed
```

You should see a bunch of new tasks enabled that begin with *feature* and *release*. These tasks will perform whatever Checkmate for OBI requires, but will use the content inside the distribution file to faciliate the tasks. In some cases... the build group tasks will use both the content in the distribution file as well as content checked into the Git repository. An example is `releaseCompare`, which will generate incremental patch files for both the repository and the catalog by comparing the content in the distribution file with whatever is in source control. Expect to see some *UP-TO-DATE* checks as Checkmate for OBI skips tasks that don't need to be rerun:

```bash
./gradlew -p obi/sample-12c releaseCompare --console=plain
:releaseExtractBuild
:catalogBuild UP-TO-DATE
:releaseCatalogCompare
:metadataBuild UP-TO-DATE
:releaseMetadataCompare
:releaseCompare

BUILD SUCCESSFUL in 36s
5 actionable tasks: 3 executed, 2 up-to-date
```

Now, we can take a look at the enhanced content in our build directory:

```bash
ls -l obi/sample-12c/build/*

obi/sample-12c/build/catalog:
total 3648
drwxr-xr-x. 1 501 games     136 Jul 26 11:42 current
-rw-r-----. 1 501 games 1863173 Jul 26 11:42 current.catalog
drwx------. 1 501 games     102 Jul 26 11:42 init
drwxr-xr-x. 1 501 games     136 Jul 26 11:42 release
-rw-r-----. 1 501 games 1863215 Jul 26 11:42 release.catalog
-rw-r-----. 1 501 games    1739 Jul 26 11:42 release-diff.txt
-rw-r-----. 1 501 games    1739 Jul 26 11:42 release-undiff.txt

obi/sample-12c/build/repository:
total 64
-rwxr-----. 1 501 games 28456 Jul 26 11:42 current.rpd
-rw-------. 1 501 games     0 Jul 26 11:42 release-compare.csv
-rwxr-----. 1 501 games   126 Jul 26 11:42 release-patch.xml
-rwxr-----. 1 501 games 28456 Jul 26 11:42 release.rpd
-rwxr-----. 1 501 games   126 Jul 26 11:42 release-unpatch.xml
drwxr-xr-x. 1 501 games    68 Jul 26 11:42 xml-variables

```

We generated all the incremental patch files, including the rollback patches, but the content of those patch files is empty, because there is currently no difference in what was published to version 0.0.9 and what is currently in source control. But you get the idea. Let's publish again, so we can create a *deploy* distribution, which contains all the patch files, as well as the original repository and catalog artifacts.

```bash
./gradlew -p obi/sample-12c publish --console=plain
:assemble UP-TO-DATE
:catalogBuild UP-TO-DATE
:check UP-TO-DATE
:metadataBuild UP-TO-DATE
:build UP-TO-DATE
:buildZip UP-TO-DATE
:generatePomFileForBuildPublication
:publishBuildPublicationToMavenLocalRepository
:deployZip
:generatePomFileForDeployPublication
:publishDeployPublicationToMavenLocalRepository
:publish

BUILD SUCCESSFUL in 7s
8 actionable tasks: 5 executed, 3 up-to-date
```

Now, let's enable the **promote** build group (distriubtion file only... we'll get to the BAR file later), which is what we use for deploying content to downstream environments.

```groovy
dependencies {
  // Using the Checkmate Testing library which is recommended.
  obiee group: 'gradle.plugin.com.redpillanalytics', name: 'checkmate', version: '8.0.6'
  // You can also use Baseline Validation Tool
  // The installation needs to be available in one of your Maven repositories
  //obiee group: 'com.oracle', name: 'oracle-bvt', version: '12.2.1.0.0'

  // Dependencies on previous OBIEE builds
  // Used for building incremental patches, regression testing, and deployments
  feature group: 'obiee', name: 'sample-12c-build', version: '+'
  release group: 'obiee', name: 'sample-12c-build', version: '0.0.9'
  promote group: 'obiee', name: 'sample-12c-deploy', version: '0.0.9'
  //promote group: 'obiee', name: 'sample-12c-bar', version: '0.0.9'
}
```

Notice that we've uncommented `promote group: 'obiee', name: 'sample-12c-deploy', version: '0.0.9'`, which gives us a series of new tasks to work with the **promote** build group. We'll use the `promotePatch` task, which uses the metadata and catalog incremental patches from the **deploy** distribution file, and applies them to the online OBIEE instance:

```bash
./gradlew -p obi/sample-12c promotePatch --console=plain
:promoteExtractDeploy
:promoteMetadataPatch
:promoteCatalogPatch
:promotePatch

BUILD SUCCESSFUL in 50s
3 actionable tasks: 3 executed
```

You'll notice that `promotePatch` is a container task for executing `promoteMetadataPatch` and `promoteCatalogPatch`, either of which can be run individually.

# Regression Testing
The high-level process Checkmate for OBI uses to regression test pull requests, source commits, or branch merges, is described below:
* Build deployment artifacts from the current content in the Git repository.
* Pull down a distribution file of OBI content published previously.
* Upload the previous content into an OBIEE environment.
* Run a series of analyses, saving the logical SQL and query results. This is called the *baseline* phase.
* Deploy the current version of content into the OBIEE environment, either using incremental patches, or by deploying whole catalogs and repositories, automatically managing the connection pool information.
* Run the same series of analyses that were run during the baseline phase, saving the output. This is called the *revision* phase.
* Compare the output generated from the baseline phase with the output generated from the revision phase. Naturally, we call this the *compare* phase.


For regression testing, Checkmate for OBI has the concept of a **test group**, which is configured using the `obi.testGroups {}` DSL structure. The default test group called **regression** is already built in, but we can use the same DSL structure to customize that build group:

```groovy
obi.testGroups {
  regression {
    // the catalog folder to test. Recursively tests all analyses in that folder
    // accepts a colon (:) separeted list of directories
    libraryFolder = '/shared'
    // By default, hash values are used for comparision of results
    // You can force the full comparision of logical SQL and results (takes longer)
    compareText = false
    // Provide more output
    showOutput = false
  }
}
```

Let's take a look at some of the parameters configured here, as well as one other that is not shown, but still worth mentioning:
* **libraryFolder:** The folder(s) in the presentation catalog that we want to regression test. This can be a colon (:) separated list of catalog folders, and Checkmate for OBI recursively includes every analysis in those folders.
* **compareText:** By default, Checkmate for OBI uses hash values of logical SQL and query results to facilitate faster comparisons, which improves performance when logical SQL is very complicated, or the analysis returns a lot of records in the result set. Setting this to `true` enables the full text search of the results.
* **showOutput:** Output more information about the execution of each regression test. Feel free to set this to `true` to see the logical SQL being executed, any error stack generated, etc.
* **impersonateUser:** Specify a user that you want to impersonate for this test group. This functionality enables us to configure multiple test groups that run the same regression tests but with different security profiles. This requires that the `adminUser` specified above be granted the `impersonate` application role, which is not granted by default.

Regression Testing involves some reasonably complex workflows, but we've tried to make that easier with several container tasks that provide the dependencies and ordering to put all of this together. From our `tasks` command, we've highlighted the output of the relevant tasks, listing the granular tasks first, followed by the complex workflow tasks:

```
Testing tasks
-------------
baselineTest - Execute all Baseline regression tests for the entire project.
compareTest - Execute all Compare regression tests for the entire project.
extractTestSuites - Extract the compiled test suites and copy them to the 'build/classes' directory.
regressionBaselineLibrary - Create the Baseline regression test library CSV file for Test Group 'regression'.
regressionBaselineTest - Execute the Baseline regression test library file for Test Group 'regression'.
regressionCompareTest - Compare the differences between the Baseline and Revision regression test results for Test Group 'regression'.
regressionRevisionLibrary - Create the Revision regression test library CSV file for Test Group 'regression'.
regressionRevisionTest - Execute the Revision regression test library file for Test Group 'regression'.
revisionTest - Execute all Revision regression tests for the entire project.

Workflow tasks
--------------
featureBaselineWorkflow - Import 'feature' metadata and catalog artifacts, manage connection pools, and execute the Baseline Test Library.
featureRevisionPatchWorkflow - Apply 'feature' metadata and catalog patches, manage connection pools, and execute the Revision Test Library.
featureRevisionWorkflow - Import 'feature' metadata and catalog artifacts, manage connection pools, and execute the Revision Test Library.
releaseBaselineWorkflow - Import 'release' metadata and catalog artifacts, manage connection pools, and execute the Baseline Test Library.
releaseRevisionPatchWorkflow - Apply 'release' metadata and catalog patches, manage connection pools, and execute the Revision Test Library.
releaseRevisionWorkflow - Import 'release' metadata and catalog artifacts, manage connection pools, and execute the Revision Test Library.
```

Executing a regression testing workflow, managing all three phases of the process (**baseline**, **revision**, **compare**), is as easy as executing the following two workflow tasks.

```bash
./gradlew -p obi/sample-12c releaseBaselineWorkflow --console=plain
:connPoolsExport
:releaseExtractBuild
:releaseMetadataImport
:releaseCatalogImport
:releaseImport
:connPoolsImport
:regressionBaselineLibrary
:extractTestSuites
:regressionBaselineTest
Results: SUCCESS (42 tests, 42 successes, 0 failures, 0 skipped)
:baselineTest
:releaseBaselineWorkflow

BUILD SUCCESSFUL in 3m 11s
8 actionable tasks: 7 executed, 1 up-to-date

./gradlew -p obi/sample-12c releaseRevisionWorkflow --console=plain
:metadataBuild UP-TO-DATE
:metadataImport
:catalogBuild UP-TO-DATE
:catalogImport
:import
:connPoolsImport
:regressionRevisionLibrary
:extractTestSuites UP-TO-DATE
:regressionRevisionTest
Results: SUCCESS (42 tests, 42 successes, 0 failures, 0 skipped)
:revisionTest
:regressionCompareTest
Results: SUCCESS (85 tests, 84 successes, 0 failures, 1 skipped)
:compareTest
:releaseRevisionWorkflow

BUILD SUCCESSFUL in 3m 8s
9 actionable tasks: 6 executed, 3 up-to-date
```

Notice that the `releaseRevisionWorkflow` task goes ahead and executes the *compare* phase of the regression testing, which is why we see more tests executed during that phase. This is the default, and is of course configurable. It's also worth noting that Checkmate for OBI generates JUnit XML files during this process, which is the industry-standard way of expressing a testing result: all Continuous Delivery and DevOps platforms recognize this standard, and can be configured to act on the results of those files.

# OBIEE 12c BAR Files
The first releases of Checkmate for OBI pre-dated OBIEE 12c, and therefore, pre-dated the new BAR file functionality in 12c. In a way, the Checkmate for OBI distribution file was our way of building a BAR file... we were just slightly ahead of the game. There's an interesting decision to be made when configuring deployment workflows... to use the BAR file or the distribution file.

At this point in the OBIEE Roadmap, we don't see a lot of value that the BAR file provides over the distribution file... especially since the distribution file also contains incremental patch files, which the BAR file simply doesn't have. Additionally, distribution files can be generated without requiring a running instance of OBIEE; Checkmate for OBI generates the distribution files by building content that exists in a Git repository using offline tools.

However, we recognize that the BAR file is the future for OBI, so we certainly support it. We can generate a BAR file using the `barExport` task, and can import a BAR file into OBIEE using the `<buildGroup>BarImport` task. Additionally, by simply setting `publishBAR = true` in our build script, Checkmate for OBI will automatically generate and publish a BAR file to our Maven repository along with the distribution file.

```groovy
obi.publishBar = true
```

```bash
./gradlew -p obi/sample-12c publish --console=plain
:barExport
:generatePomFileForBarPublication
:publishBarPublicationToMavenLocalRepository
:assemble UP-TO-DATE
:catalogBuild UP-TO-DATE
:check UP-TO-DATE
:metadataBuild UP-TO-DATE
:build UP-TO-DATE
:buildZip UP-TO-DATE
:generatePomFileForBuildPublication
:publishBuildPublicationToMavenLocalRepository
:deployZip
:generatePomFileForDeployPublication
:publishDeployPublicationToMavenLocalRepository
:publish

BUILD SUCCESSFUL in 4m 11s
11 actionable tasks: 8 executed, 3 up-to-date
```

Additionally, it's just as easy to import the BAR file we just created. We have to configure it as a dependency, just as we did with the distribution file. This is because Checkmate for OBI pulls down the desired BAR file from Maven in the same fashion it pulls down the distribution file, and we use the **promote** build group to deploy it:

```groovy
promote group: 'obiee', name: 'sample-12c-bar', version: '0.0.9'
```

Now we are able to run the `promoteBarImport` task:

```bash
./gradlew -p obi/sample-12c promoteBarImport --console=plain
:promoteBarImport

BUILD SUCCESSFUL in 51s
1 actionable task: 1 executed
```

We can also easily import the predefined SampleAppLite BAR file if you ever need to:

```bash
./gradlew -p obi/sample-12c barImportSAL --console=plain
:barImportSAL

BUILD SUCCESSFUL in 1m 7s
1 actionable task: 1 executed
```
Or, use the "empty BAR file" that also ships with OBIEE 12c:

```bash
./gradlew -p obi/sample-12c barReset --console=plain
:barReset

BUILD SUCCESSFUL in 43s
1 actionable task: 1 executed
```
