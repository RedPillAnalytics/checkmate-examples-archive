# Checkmate for OBI Quickstart
This Quickstart demonstrates basic functionality of the Checkmate for OBI Build Framework, using version 12.2.1.4 of Oracle Business Intelligence. The project folder includes sample OBIEE content from [SampleAppLite](https://docs.oracle.com/middleware/bi12214/biee/BIESG/GUID-7FCD90A3-E005-49BF-902F-30FBF9B41B07.htm#BIESG9340) already checked into the [`src/main`](src/main) directory. The repository also contains a [Docker compose](https://docs.docker.com/compose/overview/) configuration that provides an OBIEE 12.2.1.4 environment for walking through the Quickstart.

Checkmate is built using [Gradle](https://www.gradle.org): a declarative, [DSL](https://en.wikipedia.org/wiki/Domain-specific_language)-based build tool commonly associated with building JVM-based software and [Android apps](https://developer.android.com/studio/index.html). Specifically, Checkmate is a series of [Gradle Plugins](https://guides.gradle.org/designing-gradle-plugins/). The following OBI functionality exists in the [com.redpillanalytics.checkmate.obi](https://plugins.gradle.org/plugin/com.redpillanalytics.checkmate.obi) plugin: source control integration, content versioning and publishing, automated regression and integration testing, and automated deployments.

A real delivery pipeline will likely have a Continuous Delivery server involved in this process, such as Jenkins, Travis CI, Google CloudBuild, etc. All of these CD servers have Gradle integration which makes Checkmate easy to use.

# Docker Compose for OBIEE
We have configured all the necessary steps for starting an OBIEE 12.2.1.4 environment on Docker for use with this Quickstart. Beware: OBIEE is resource-intensive, and requires a considerable amount of memory and CPU for your Docker daemon. We recommend the following settings: `CPUs:4` and `Memory:8GB`.

First, clone the repository from GitHub using your favoriate Git client, or use the command-line:

```bash
git clone https://github.com/RedPillAnalytics/checkmate-examples.git
cd checkmate-examples
```

We've configured all the Docker Compose functionality to be usable with Gradle. Since we are working with the `obi` subproject folder, all of the Gradle tasks we call are prefixed with `obi:`. Additionally, I'm adding the `-q` option to remove some of the noise output to the screen by Docker Compose. This step starts up a Linux-based OBIEE instance, so it can be time-consuming. Also, the first time this statement is run, the rather-large OBIEE and Oracle XE database Docker images have to be downloaded, which can also take a long time:

```bash
./gradlew obi:composeUp -q
rcu-database uses an image, skipping
obiee uses an image, skipping
Creating rpa-checkmate-db ...
Creating rpa-checkmate-db ... done
Creating rpa-checkmate-bi ...
Creating rpa-checkmate-bi ... done
```

Once the envionment is available, we can make a bash connection to the main OBIEE container for the remainder of the Quickstart:

```bash
docker exec -u oracle -w /checkmate-examples -ti rpa-checkmate-bi bash
```

To ensure our OBIEE environment is functionality correctly, do the following:

```bash
/opt/oracle/config/domains/bi/bitools/bin/status.sh
Domain status; Using domainHome: /opt/oracle/config/domains/bi ...

Initializing WebLogic Scripting Tool (WLST) ...

Welcome to WebLogic Server Administration Scripting Shell

Type help() for help on available commands

<Feb 25, 2019 8:04:36 AM UTC> <Info> <Security> <BEA-090905> <Disabling the CryptoJ JCE Provider self-integrity check for better startup performance. To enable this check, specify -Dweblogic.security.allowCryptoJDefaultJCEVerification=true.>
<Feb 25, 2019 8:04:36 AM UTC> <Info> <Security> <BEA-090906> <Changing the default Random Number Generator in RSA CryptoJ from ECDRBG128 to HMACDRBG. To disable this change, specify -Dweblogic.security.allowCryptoJDefaultPRNG=true.>
<Feb 25, 2019 8:04:36 AM UTC> <Info> <Security> <BEA-090909> <Using the configured custom SSL Hostname Verifier implementation: weblogic.security.utils.SSLWLSHostnameVerifier$NullHostnameVerifier.>
/Servers/AdminServer/ListenPort=9500
Accessing admin server using URL t3://rpa-checkmate-bi:9500

Status of Domain: /opt/oracle/config/domains/bi
NodeManager (rpa-checkmate-bi:9506:SSL): RUNNING

Name            Type            Machine                   Restart Int Max Restart  Status
----            ----            -------                   ----------- -----------  ------
AdminServer     Server          rpa-checkmate-bi          unknown     unknown      RUNNING
bi_server1      Server          rpa-checkmate-bi          unknown     unknown      RUNNING
obips1          OBIPS           rpa-checkmate-bi          3600        2            RUNNING
obijh1          OBIJH           rpa-checkmate-bi          3600        2            RUNNING
obiccs1         OBICCS          rpa-checkmate-bi          3600        2            RUNNING
obisch1         OBISCH          rpa-checkmate-bi          3600        2            RUNNING
obis1           OBIS            rpa-checkmate-bi          3600        2            RUNNING

```

And to ensure that our Gradle CLI is functioning inside the Docker container, let's run a basic Checkmate build. The first time a command is executed, Gradle will pull down any library dependencies used by Checkmate from the central Maven repository called [Bintray jCenter](https://bintray.com/bintray/jcenter), including the Gradle distribution itself, so we should see of this happening:

```bash
./gradlew obi:build

BUILD SUCCESSFUL in 30s
2 actionable tasks: 2 executed
```

At any time throughout this Quickstart, we can add the `-i` option to any of our Gradle task executions. The output generated with this option is more verbose, which isn't great for documentation, but would be helpful in understanding some of the things going on under the covers.

# Basic Configuration
The heart of a Gradle build is the [build script](https://docs.gradle.org/current/userguide/tutorial_using_tasks.html#sec:hello_world), which by default is defined using a [`build.gradle`](https://github.com/RedPillAnalytics/checkmate-examples/blob/master/obi/build.gradle) file, with all the necessary configurations already made, with a few advanced features that we need later commented out.

The `plugins{}` closure applies any desired plugins from the [Gradle Plugin Portal](https://plugins.gradle.org) using the unique ID associated with that plugin:

```groovy
plugins {
  id 'com.redpillanalytics.checkmate.obi' version '10.0.4'
  id 'com.avast.gradle.docker-compose' version "0.8.14"
  id 'com.adarshr.test-logger' version '1.6.0'
}
```

* `com.redpillanalytics.checkmate.obi`: this is the Checkmate for OBI plugin. It enables all the features demonstrated in this Quickstart.
* `com.avast.gradle.docker-compose`: enables running the OBIEE 12.2.1.4 on Docker environment for this Quickstart.
* `com.adarshr.test-logger`: improves readability of output from *regression tests*. We'll discuss testing later on.

With the Checkmate for OBI plugin applied, our Gradle [project](https://docs.gradle.org/current/userguide/tutorial_using_tasks.html#sec:projects_and_tasks) will contain all the tasks, properties and configurations necessary to manage an Oracle Analytics lifecycle. Checked into the Git repository is the [Gradle Wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html) which is a lightweight CLI used to interact with our Gradle project. The wrapper automatically downloads the full Gradle distribution the first time it's executed by a user, and it also downloads all the required plugins fro the plugin portal. This makes Checkmate very portable: there is no installation necessary to run all the backend processes.

Let's use the wrapper and execute the `obi:tasks` command, which will show us all the configured tasks in the `obi` subdirectory:

```bash
./gradlew obi:tasks

> Task :obi:tasks

------------------------------------------------------------
Tasks runnable from project :obi
------------------------------------------------------------

Build tasks
-----------
assemble - Assembles the outputs of this project.
build - Build all OBI content for this project.
clean - Deletes the build directory.

Docker tasks
------------
composeBuild - Builds images for services of docker-compose project
composeDown - Stops and removes containers of docker-compose project (only if stopContainers is set to true)
composeDownForced - Stops and removes containers of docker-compose project
composeLogs - Stores log output from services in containers of docker-compose project
composePull - Builds and pulls images of docker-compose project
composePush - Pushes images for services of docker-compose project
composeUp - Builds and starts containers of docker-compose project

Help tasks
----------
buildEnvironment - Displays all buildscript dependencies declared in project ':obi'.
components - Displays the components produced by project ':obi'. [incubating]
dependencies - Displays all dependencies declared in project ':obi'.
dependencyInsight - Displays the insight into a specific dependency in project ':obi'.
dependentComponents - Displays the dependent components of components in project ':obi'. [incubating]
help - Displays a help message.
model - Displays the configuration model of project ':obi'. [incubating]
projects - Displays the sub-projects of project ':obi'.
properties - Displays the properties of project ':obi'.
tasks - Displays the tasks runnable from project ':obi'.

OBI Build tasks
---------------
barBuild - Build and assemble the components for the BI Archive (BAR) file from SCM.
barSync - Synchronize the BAR build directory from SCM.
catalogBuild - Copy catalog from SCM to 'catalog/current' and generate catalog archive file 'catalog/current.catalog'.
metadataBuild - Build binary repository 'repository/current.rpd' from the MDS-XML repository in SCM.

OBI Distribution tasks
----------------------
barZip - Create a BI Archive (BAR) file from the assembled component.
buildZip - Create a ZIP distribution archive for downstream deployments containing metadata and catalog artifacts.
deployZip - Create a ZIP distribution archive for downstream deployments containing metadata and catalog artifacts and incremental patches.

OBI Export tasks
----------------
barExport - Execute barStage followed by barSource.
barSource - Synchronize the staged BAR with SCM.
barStage - Export BI Archive (BAR) File using the 'ssi' Service Instance.
catalogExport - Export the online presentation catalog into SCM by calling 'catalogStage' followed by 'catalogSource'
catalogSource - Synchronize the staged catalog with the offline presentation catalog in SCM.
catalogStage - Stage the online presentation catalog at 'catalog/current'.
connPoolsExport - Export target OBIEE server connection pool information in JSON format to 'repository/conn-pools.json'.
export - Execute all configured export tasks.
metadataExport - Export the online metadata repository into SCM by calling 'metadataStage' followed by 'metadataSource'
metadataSource - Synchronize the staged repository with the repository in SCM.
metadataStage - Stage the online metadata repository to 'repository/current.rpd'.
variablesExport - Export target OBIEE server variable information in JSON format to 'repository/variables.json'.

OBI Import tasks
----------------
applyVersionJson - Set the 'project_version' repository variable to version '1.0.0'.
barImport - Import BI Archive (BAR) File into the 'ssi' Service Instance.
barImportSAL - Import SampleAppLite.bar BI Archive (BAR) File into the 'ssi' Service Instance.
barReset - Reset the 'ssi' Service Instance, equivalent to using an empty BI Archive (BAR) File.
catalogImport - Import the presentation catalog from SCM into the online presentation catalog using 'catalog/current'.
connPoolsImport - Import server connection pool information in JSON format from 'repository/conn-pools.json' to the target OBIEE server.
generateVersionJson - Generate JSON patch of version '1.0.0' for 'project_version' repository variable.
import - Execute all configured import tasks.
metadataImport - Import 'repository/current.rpd' into the online metadata repository.
variablesImport - Import server variable information in JSON format from 'repository/variables.json' to the target OBIEE server.

OBI SCM tasks
-------------
catalogSCM - Synchronize 'catalog/current' with SCM and then commit.
metadataSCM - Synchronize 'repository/current.rpd' with SCM and then commit.
scmCheckout - Checkout a branch in the local SCM repository.
scmCommit - Issue a commit to the local SCM repository. Customize with 'scmMessage', 'scmCommitter', 'scmEmail', and 'scmCommitPath' build parameters.
scmPush - Push to the origin for the local SCM repository. Requires 'sourceBase' build parameter pointing to Git repo root directory, if running task outside of a Git repository.

OBI Services tasks
------------------
metadataReload - Execute the 'Reload Files and Metadata' web service.

OBI Testing tasks
-----------------
compareTest - Execute all Compare regression tests for the entire project.
extractTestSuites - Extract the compiled test suites and copy them to the 'build/classes' directory.
regressionCompareTest - Compare the differences between the Baseline and Revision regression test results for Test Group 'regression'.
regressionResultsLibrary - Create the Results regression test library CSV file for Test Group 'regression'.
regressionResultsTest - Execute the Results regression test library file for Test Group 'regression'.
resultsTest - Execute all Results regression tests for the entire project.

OBI Workflow tasks
------------------
importWorkflow - Import 'current' metadata and catalog artifacts while managing connection pools.
resultsWorkflow - Import 'current' metadata and catalog content and execute the Revision test library.

Publishing tasks
----------------
generateMetadataFileForBuildPublication - Generates the Gradle metadata file for publication 'build'.
generateMetadataFileForDeployPublication - Generates the Gradle metadata file for publication 'deploy'.
generatePomFileForBuildPublication - Generates the Maven POM file for publication 'build'.
generatePomFileForDeployPublication - Generates the Maven POM file for publication 'deploy'.
publish - Publishes all publications produced by this project.
publishBuildPublicationToMavenLocal - Publishes Maven publication 'build' to the local Maven repository.
publishBuildPublicationToMavenLocalRepository - Publishes Maven publication 'build' to Maven repository 'MavenLocal'.
publishDeployPublicationToMavenLocal - Publishes Maven publication 'deploy' to the local Maven repository.
publishDeployPublicationToMavenLocalRepository - Publishes Maven publication 'deploy' to Maven repository 'MavenLocal'.
publishToMavenLocal - Publishes all Maven publications produced by this project to the local Maven cache.

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

BUILD SUCCESSFUL in 6s
```

You can scroll through all the tasks to get an idea of the completeness of what Checkmate handles. We'll discuss many of these tasks in detail, but at a high level, Checkmate for OBI enables the following [task groups](https://docs.gradle.org/current/dsl/org.gradle.api.Task.html#org.gradle.api.Task:group):

* **OBI Distribution**: Management of *artifacts* of OBIEE content: *distribution* files and *BAR* files. Distribution files are ZIP artifacts that Checkmate has been generating since back in the OBIEE 11g days. Depending on the workflow, a distribution artifact can contain any combination of the following: a binary metadata repository, the full web catalog, a catalog archive of the `/Shared` folders of the web catalog, incremental patches for both the metadata repository and the web catalog, and the results files from any tests run while building the artifact. We'll discuss distribution files, and the testing framework, later on. BAR files are ZIP files that were introduced in OBIEE 12c as well as tooling for importing and exporting them from OBI instances.
* **OBI Export**: Tasks that facilitate exporting content from an OBIEE instance into the build directory and eventually into source control. Although exports into source control can be run from the backend, this is usually part of the developer experience managed by [Checkmate Studio](https://github.com/RedPillAnalytics/checkmate-examples#checkmate-studio-18x).
* **OBI Import**: Tasks that facilitate importing content into an OBIEE instance from source control, or from artifacts downloaded from Maven. Although imports from source control can be run from the backend, this is usually part of the developer experience managed by [Checkmate Studio](https://github.com/RedPillAnalytics/checkmate-examples#checkmate-studio-18x).
* **OBI SCM**: Tasks for integrating `checkout` and `commit` workflows with [Git](https://git-scm.com). These tasks are for very complex workflows, and are generally unnecessary when using standard CI/CD processes.
* **OBI Services**: The `metadataReload` task, which executes *Reload Files and Metadata* in OBIEE.
* **OBI Testing**: Tasks for executing and reporting on unit and regression tests.

The Maven Publish plugin enables the following Task Groups:
* **Publishing**: Publishing content to Maven repositories.

Gradle enables certain default Task Groups as well:
* **Build Setup**: For generating new projects and for generating the Gradle Wrapper.
* **Help**: Basic help tasks.
* **Verification**: Runs any configured checks enabled in the project.

# OBIEE Environment Configuration
Checkmate needs to know the basics about the OBIEE environment it will execute against. We use Checkmate **build parameters** to configure properties for the OBIEE environment, as well as other things we'll see later.

Build parameters can be enabled one of five ways, in reverse-prioritized order... meaning the last item in the list overrides the second-to-the-last item, and so forth:
* Specified in the `build.gradle` file
* Specified in a `gradle.properties` file in the project directory. 
* Specified in a `gradle.properties` file in the `GRADLE_HOME` directory, which defaults to the `$HOME/.gradle` directory of the executing user.
* Specified with environment variables
* Specified using Gradle project properties, which are passed to the Gradle Wrapper command-line using `-P<property>=<value>`

Checkmate provides this degree of flexibility because many build properties are environment specific, and may need to change from one environment to the next. Additionally, some parameters--such as passwords--are sensitive, and need to be treated as such. For instance, [Jenkins Credentials](https://jenkins.io/doc/book/pipeline/syntax/#environment), which are typically used to store passwords in Continuous Delivery environments, are exposed as environment variables, so this is a very handy way to pass sensitive build parameters to Checkmate for OBI.

For the sake of simplicity and clarity, we'll declare all the build parameters in the `build.gradle` file... even the sensitive ones. Just remember: you would want to use another approach in a real delivery pipeline. We'll use the `obi{}` closure to define these properties:

```gradle
obi {
  middlewareHome = '/opt/oracle/product/12.2.1.4.0'
  domainHome = '/opt/oracle/config/domains/bi'
  compatibility = '12.2.1.4'
  adminUser = 'weblogic'
  adminPassword = 'Admin123'
  repositoryPassword = 'Admin123'
  contentPolicy = 'distribution'
}
```

Notes on a few of the parameters below:
* **domainHome:** Defaults to `<obi.middlewareHome>/user_projects/domains/bi`, but can be configured separately as we've done here.
* **compatibility:** Options are `12.2.1.4, 12.2.1.3, 12.2.1.2, 12.2.1.1, 12.2.1.0, 11.1.1.9 and 11.1.1.7`. There are subtle and not-so-subtle differences in the way Checkmate interacts with OBIEE in the different releases, so this parameter controls that behavior. The `11.x` functionality and corresponding parameter values will likely disappear in the near future.
* **contentPolicy:** This parameter controls whether we are primarily importing, exporting and publishing OBIEE content using traditional tools and APIs, therefore generating distribution files. Or, whether we are using the new BAR utilities and APIs introduced in OBIEE 12c. Possible values include: `distribution, bar, mixed, distribution-metadata, distribution-catalog`. The main values for this parmater are `distribution` or `bar`, with all others being for mostly depracated use cases. We are using the `distribution` setting, which is also the default

# Building and Publishing
The workflow for building OBIEE content usually occurs in the following steps:
* **Build:** The building of OBIEE content from source control. In source control we have the following checked in: the metadata repository as MDS-XML, and the presentation catalog in filesystem structure. The **build** steps involve building a binary RPD, as well as a catalog archive file of the `/Shared` folders of the catalog. Depending on the value of `contentPolicy`, we package this up as either a distribution file or a BAR file. Both artifact types are fully supported in OBIEE 12c.
* **Test (Optional):** After building our content, and prior to publishing it, we'll run a series of unit and/or regression tests to make sure our new content works as expected, and hasn't broken anything. We'll discuss testing more later on... so for now we'll exclude testing results from our distribution files.
* **Publish:** Publishing distribution files or BAR files to one or more [Maven repositories](https://maven.apache.org/pom.html#Repositories).

Currently, we have the `publishing{}` closure with a `repositories{}` closure describing our publication location, as well as a version number:

```gradle
version = '1.0.0'

publishing {
  repositories {
    mavenLocal()
  }
}
```

This will publish our OBI distribution files or BAR files to a [Maven Local](https://docs.gradle.org/current/userguide/publishing_maven.html#publishing_maven:install) repository, which defaults to `$HOME/.m2`. Obviously, this is for testing purposes only. In a real continuous delivery pipeline, content should be published to a resilient Maven-compatible repository. At the very least, publish to an S3 bucket, which Gradle supports, using the following syntax:

```gradle
publishing {
  repositories {
    maven {
      url 's3://bucket-name/path/to/repository'
      credentials(AwsCredentials) {
        accessKey 'access key'
        secretKey 'secret key'
      }
    }
  }
}
```

We've hard-coding our version number to `1.0.0` just for ease and clarity: we would normally use a Gradle plugin such as the [axion-release-plugin](https://github.com/allegro/axion-release-plugin) to handle release management with semantic versioning and automatic version bumping.

Why do we bother publishing our artifacts to Maven repositories? So we can pull that artifact later as a depdendency by simply referring to it's coordinates and version number. This allows us to automate deployments to downstream environments using published artifacts. It also allows us to do some interesting things with our testing framework: we can pull prior artifacts and do comparisons of test results. We can also use artifacts to generate incremental patch files to include in new distribution files: we metadata and catalog content in the artifact to compare with current content in the production of these patches. These incremental patches can be used for deployment in highly-continuous environments.

We'll run the `obi:build` task again, but this time we'll use the `--console=plain` option, which provides better screen output for documentation. We'll also add the `obi:clean` task to first clean our build directory from prior executions:

```bash
./gradlew obi:clean obi:build --console=plain
> Task :obi:clean
> Task :obi:assemble UP-TO-DATE
> Task :obi:catalogBuild
> Task :obi:check UP-TO-DATE
> Task :obi:metadataBuild
> Task :obi:build

BUILD SUCCESSFUL in 45s
3 actionable tasks: 3 executed
```

You'll notice that the `build` task doesn't really do anything on its own: it's really just a container for two other tasks that do all the work: `metadataBuild` and `catalogBuild`. This introduces Gradle's powerful dependencies and ordering features, which uses a [DAG](https://docs.gradle.org/current/userguide/build_lifecycle.html) implementation.

Furthermore... we can run the entire Build and Publish workflow by simply running the `publish` task, which has dependencies on building and bundling all the content.

```bash
./gradlew obi:publish --console=plain
> Task :obi:assemble UP-TO-DATE
> Task :obi:catalogBuild UP-TO-DATE
> Task :obi:check UP-TO-DATE
> Task :obi:metadataBuild UP-TO-DATE
> Task :obi:build UP-TO-DATE
> Task :obi:buildZip
> Task :obi:generatePomFileForBuildPublication
> Task :obi:publishBuildPublicationToMavenLocalRepository
> Task :obi:deployZip
> Task :obi:generatePomFileForDeployPublication
> Task :obi:publishDeployPublicationToMavenLocalRepository
> Task :obi:publish

BUILD SUCCESSFUL in 34s
8 actionable tasks: 6 executed, 2 up-to-date
```

In the output, you'll notice the *UP-TO-DATE* status that Checkmate is reporting when running the dependent tasks. Checkmate is written to take advantage of the [Gradle Incremental Build](https://docs.gradle.org/current/userguide/more_about_tasks.html#sec:up_to_date_checks) feature. The catalog and metadata build tasks are not executed again, because none of the task input and output files have changed. This keeps Checkmate from re-running tasks that it doesn't have to. Rerunning tasks can always be forced by providing the `--rerun-tasks` command-line option.

Let's take a look at what our Build and Publish process generated. If we look at the `obi/build` directory, we can see all the things that Checkmate built, with a subset displayed below:

```bash
ls -l obi/build/*
obi/build/catalog:
total 2116
drwxr-xr-x 4 oracle dba     128 Feb 22 21:42 current
-rw-r----- 1 oracle dba 2163654 Feb 22 21:42 current.catalog

obi/build/distributions:
total 10648
-rw-r--r-- 1 oracle dba 4893723 Feb 22 21:56 obi-build-1.0.0.zip
-rw-r--r-- 1 oracle dba 4893723 Feb 22 21:56 obi-deploy-1.0.0.zip

obi/build/misc:
total 0
drwxr-xr-x 2 oracle dba 64 Feb 22 21:42 xml-variables

obi/build/publications:
total 0
drwxr-xr-x 3 oracle dba 96 Feb 22 21:56 build
drwxr-xr-x 3 oracle dba 96 Feb 22 21:56 deploy

obi/build/repository:
total 32
-rwxr----- 1 oracle dba 29624 Feb 22 21:42 current.rpd
```

Additionally, we can look at our Maven Local repository, and see the versioned distribution files published there. You can explore the directory structure further to get a flavor for the Maven filesystem:

```bash
ls -l ~/.m2/repository/obiee
total 8
drwxr-xr-x 3 oracle dba 4096 Feb 22 21:56 obi-build
drwxr-xr-x 3 oracle dba 4096 Feb 22 21:56 obi-deploy
```

Later on, we'll expore the differences between the **build** and the **deploy** distributions. For now... just know that **build** contains a subset of what's in **deploy**.

# Build Groups
Checkmate for OBI uses the concept of a **build group**: a collection of tasks associated with a particular dependency, which in our case, is a dependency on a published artifact: either a distribution file or a BAR file. Up until now, all the tasks demonstrated exist without being tied to a dependency: they are the core Checkmate for OBI tasks that revolve around working with content checked into a Git repository.

A build group allows us to declare a dependency on a prior version of an artifact, and then get a bunch of new, dynamically generated tasks that belong to that build group. Checkmate contains three build groups by default:
* **feature:** used primarily to test new feature branches prior to their being merged into a mainline of code, usually the **develop** or **master** branch.
* **release:** used to test new releases prior to being deployed to downstream environments, or prior to being merged into release branches such as **master** or **release.x.x.x**.
* **promote:** used for promoting content to downstream environments, and do smoke-testing of those environments.

Because these build groups are already built-in, we don't have to do much to enable them; all we have to do is declare a dependency on a prior artifact version, and the tasks in this build group will magically appear. Since we now have the `1.0.0` distribution published to our Maven Local repository, we can use that distribution as our dependency for these build groups. We could also define custom build groups using the `obi.buildGroups {}` DSL structure if for some reason the combination of **feature**, **release** and **promote** was not sufficient for our workflow.

The first thing we have to tell Checkmate is where to go looking for our artifacts. We're still using Maven Local for this, so we see that in our `repositories{}` closure:

```gradle
repositories {
  mavenLocal()
}
```

We'll make the following changes to our `build.gradle` file, which should be possible by simply uncommenting the `feature 'obiee:obi-build:+'` line:

```gradle
dependencies {
  obiee 'com.redpillanalytics:checkmate-obi:+'
  feature 'obiee:obi-build:+'
}
```

The DSL might be a bit confusing, because we are using Gradle's built-in dependency resolution functionality to resolve our OBI artifacts. Basically, we use a Gradle [configuration](https://docs.gradle.org/current/dsl/org.gradle.api.artifacts.Configuration.html) to declare a dependency on a particular artifact... in this case, the **build** distribution file at a particular version. Notice we have a '`+`' for the version number: this signifies that we want to download whatever the latest version of the artifat is. We could have easily written `feature 'obiee:obi-build:1.0.0'` if we wanted to be exact. We can look at all the new tasks now enabled by the `feature` build group:

```bash
./gradlew tasks | grep feature
featureCatalogCompare - Build incremental file 'catalog/feature-diff.txt' and 'catalog/feature-undiff.txt' using the 'feature' configuration.
featureCompare - Execute 'featureCatalogCompare' and 'featureMetadataCompare'.
featureMetadataCompare - Build incremental files 'repository/feature-patch.xml', 'repository/feature-unpatch.xml' and 'repository/feature-compare.csv' using the 'feature' configuration.
featureExtractBuild - Extract OBIEE dependencies for 'feature'.
featureApplyVersionJson - Set the 'project_version' repository variable to version '1.0.0'.
featureCatalogImport - Import the presentation catalog from SCM into the online presentation catalog using 'catalog/feature'.
featureGenerateVersionJson - Generate JSON patch of version '1.0.0' for 'project_version' repository variable.
featureImport - Execute all configured import tasks for buildGroup 'feature'.
featureMetadataImport - Import 'repository/feature.rpd' into the online metadata repository.
featureCatalogPatch - Apply 'catalog/feature-diff.txt' to the online presentation catalog.
featureCatalogUnpatch - Apply 'catalog/feature-diff.txt' to the online presentation catalog.
featureMetadataPatch - Apply 'repository/feature-patch.xml' to the metadata repository in offline mode.
featureMetadataUnpatch - Apply 'repository/feature-unpatch.xml' to the metadata repository in offline mode
featurePatch - Execute all configured patch tasks for buildGroup 'feature'.
featureUnpatch - Execute all configured unpatch tasks for buildGroup 'feature'.
featureCatalogMerge - Use OBIEE merging instead of SCM merging for presentation catalog.
featureMerge - Execute 'featureMetadataMerge' and 'featureCatalogMerge'.
featureCompareWorkflow - Extract Baseline test library and results from the 'feature' artifact and run Compare tests.
featureImportWorkflow - Import 'feature' metadata and catalog artifacts while managing connection pools.
featureTestWorkflow - Execute ':resultsWorkflow', ':featureCompareWorfklow' and ':publish'.
```

You should see a bunch of new tasks enabled that begin with *feature* and *release*. These tasks will perform whatever Checkmate requires, but will use the content inside the artifact to faciliate the tasks. In some cases... the build group tasks will use both the content in the distribution file as well as content checked into the Git repository. An example is `featureCompare`, which will generate incremental patch files for both the repository and the catalog by comparing the content in the distribution file with whatever is in source control. We execute this task, and also publish again so we can create a *deploy* distribution, which contains all the patch files, as well as the original repository and catalog artifacts. Expect to see some *UP-TO-DATE* checks as Checkmate for OBI skips tasks that don't need to be rerun:

```bash
./gradlew obi:releaseCompare obi:publish --console=plain
> Task :obi:releaseExtractBuild
> Task :obi:catalogBuild UP-TO-DATE
> Task :obi:releaseCatalogCompare
> Task :obi:metadataBuild UP-TO-DATE
> Task :obi:releaseMetadataCompare
> Task :obi:releaseCompare
> Task :obi:assemble UP-TO-DATE
> Task :obi:check UP-TO-DATE
> Task :obi:build UP-TO-DATE
> Task :obi:buildZip UP-TO-DATE
> Task :obi:generatePomFileForBuildPublication
> Task :obi:publishBuildPublicationToMavenLocalRepository
> Task :obi:deployZip
> Task :obi:generatePomFileForDeployPublication
> Task :obi:publishDeployPublicationToMavenLocalRepository
> Task :obi:publish

BUILD SUCCESSFUL in 1m 42s
11 actionable tasks: 8 executed, 3 up-to-date
```

Notice that `:buildZip` was *UP-TO-DATE* while `:deployZip` was not. This is because our incremental patch files get included in the **deploy** distribution file, so it had to be recreated. Now, we can take a look at the enhanced content in our build directory, of which I'm including a subset of here:

```bash
ls -l obi/build/*
obi/build/catalog:
total 4240
drwxr-xr-x 4 oracle dba     128 Feb 22 23:16 current
-rw-r----- 1 oracle dba 2163675 Feb 22 23:16 current.catalog
drwx------ 3 oracle dba      96 Feb 22 23:18 init
drwxr-xr-x 4 oracle dba     128 Feb 22 23:17 release
-rw-r----- 1 oracle dba    1693 Feb 22 23:18 release-diff.txt
-rw-r----- 1 oracle dba    1693 Feb 22 23:18 release-undiff.txt
-rw-r----- 1 oracle dba 2163675 Feb 22 23:17 release.catalog

obi/build/distributions:
total 15236
-rw-r--r-- 1 oracle dba 4888828 Feb 22 23:16 obi-build-1.0.0.zip
-rw-r--r-- 1 oracle dba 9809721 Feb 22 23:19 obi-deploy-1.0.0.zip

obi/build/misc:
total 0
drwxr-xr-x 2 oracle dba 64 Feb 22 23:16 xml-variables

obi/build/publications:
total 0
drwxr-xr-x 3 oracle dba 96 Feb 22 23:16 build
drwxr-xr-x 3 oracle dba 96 Feb 22 23:17 deploy

obi/build/repository:
total 108
-rwxr----- 1 oracle dba 29624 Feb 22 23:16 current.rpd
-rw------- 1 oracle dba     0 Feb 22 23:18 release-compare.csv
-rw------- 1 oracle dba    48 Feb 22 23:18 release-merge-decision.csv
-rwxr----- 1 oracle dba 29640 Feb 22 23:18 release-merge.rpd
-rwxr----- 1 oracle dba   126 Feb 22 23:18 release-patch.xml
-rwxr----- 1 oracle dba   126 Feb 22 23:18 release-unpatch.xml
-rwxr----- 1 oracle dba 29624 Feb 22 23:18 release.rpd
```

We generated all the incremental patch files, including the rollback patches, but the content of those patch files is basically empty, because there is currently no difference in what was published to version `1.0.0` and what is currently in source control; but you get the idea.

Now, let's enable the **promote** build group, which is what we use for deploying content to downstream environments. So we just need to uncomment the `promote 'obiee:obi-deploy:+'` line from our `build.gradle` file:

```gradle
dependencies {
  obiee 'com.redpillanalytics:checkmate-obi:+'
  feature 'obiee:obi-build:+'
  release 'obiee:obi-build:1.0.0'
  promote 'obiee:obi-deploy:+'
}
```

We'll execute the `promotePatch` task, which uses the metadata and catalog incremental patches from the **deploy** distribution file, and applies them to the online OBIEE instance:

```bash
./gradlew obi:promotePatch --console=plain
> Task :obi:promoteExtractDeploy
> Task :obi:promoteMetadataPatch
> Task :obi:promoteCatalogPatch
> Task :obi:promotePatch

BUILD SUCCESSFUL in 55s
3 actionable tasks: 3 executed
```

You'll notice that `promotePatch` is a container task for executing `promoteMetadataPatch` and `promoteCatalogPatch`, either of which can be run individually.

Promoting to downstream environments using incremental patches is a great solution for truly continuous environments that deploy often. But as the time between releases increases, so does the size and complexity of the incremental patches, and the possibility that the automated application of these patches might fail. In these cases, we can simply use the `promoteImportWorkflow` task:

```bash
./gradlew obi:promoteImportWorkflow --console=plain
> Task :obi:connPoolsExport
> Task :obi:promoteExtractDeploy
> Task :obi:promoteMetadataImport
> Task :obi:promoteCatalogImport
> Task :obi:connPoolsImport
> Task :obi:metadataReload
> Task :obi:promoteImport
> Task :obi:promoteImportWorkflow

BUILD SUCCESSFUL in 36s
6 actionable tasks: 6 executed
```
Notice that this *workflow* task manages connection pools for us. Before our new metadata content is imported into the instance, we save the connection pool information as a JSON file, and then reapply it once the import process is complete. You can see the connection pool files in the build directory:

```bash
cat obi/build/repository/*.json

{
    "Title":"List Connection Pools",
    "Conn-Pool-Info":[
        {
            "uid":"80ca62c5-0bd5-0000-714b-e31d00000000",
            "connPool":"SampleApp_Lite_Xml",
            "parentName":"\"Sample App Lite Data\"",
            "user":"<Replace Me>",
            "password":"6C07839E325582FD226EB6A757C7E309 7C2E35213306F12832914CBE7A9DD95561D771DED06484112B1FC6F27B6D0D58",
            "dataSource":"<Replace Me>",
            "appServerName":"<Replace Me>"
        }
    ],
    "Variables-In-Conn-Pool":[
    ]
}
```

The connection pool metadata included in SampleApp isn't very compelling as it uses XML data files, but you can see that the encrypted password is included, and although no variables are included in this connection pool, if they did, our JSON file would capture those as well.

# Testing
Checkmate has a pluggable testing framework that supports either the standard Checkmate testing library built in to our Gradle plugin, or the Baseline Validation Tool (BVT) which is offered from Oracle. We prefer our built-in testing framework, so we'll configure that, first by telling Checkmate that the library is available in the Gradle Plugin Maven repo, which is specified with the `maven { url "https://plugins.gradle.org/m2/" }` line:

```gradle
repositories {
  mavenLocal()
  maven { url "https://plugins.gradle.org/m2/" }
}
```
Then we tell Checkmate that the `obiee` configuration requires the Checkmate testing library, which is specified with the `com.redpillanalytics:checkmate-obi:+` dependency:

```gradle
dependencies {
  obiee 'com.redpillanalytics:checkmate-obi:+'
  feature 'obiee:obi-build:+'
  release 'obiee:obi-build:1.0.0'
  promote 'obiee:obi-deploy:+'
}
```
In Checkmate, a test is simply a web catalog analysis that we've written to test some aspect of our analytics system. In the analysis, we can join together a series of tables and ensure that they join correctly, or make calculations across multiple hierarchies and ensure that the query results make sense. This analysis will be processed using two different types of tests in Checkmate:
* **Results Tests:** extract the *logical SQL* from the analysis and execute it against the BI Server. We store the logical SQL, the full results from the analysis, and the hash value from the results in the build directory, and also collect it in our distribution file. This gives us access to the test results whenever we pull the artifact. You can think of results tests as similar to *unit tests* in other development paradigms.
* **Compare Tests:** compare the results tests from our current branch with the results tests extracted from our distribution file to see if either the logical SQL or the execution results change. This comparision can be done using the hash value calculated during the results test (faster), or a full textual comparison of the analysis results test output. You can think of compare tests as similar to *regression tests* in other development paradigms.

To fully execute each test, we need to capture the results of that analysis *before and after* our most recent changes, and then compare those results. If results match, the test is successful; otherwise, it's not. We typically run these results in response to Git commits, and more specifically, either *pull requests* or *branch merges*. Additionally, Checkmate generates JUnit XML files for every unit and regression test it executes. Each JUnit result file expresses the overall test result (*success, failed, skipped*) as well as the execution time, and the standard output and error of the process. Most continuous integration servers and development IDEs are able to parse JUnit files and report back on the success and failure of tests either individually or aggregated across complex testing workflows.

Checkmate for OBI has the concept of a **test group**, which is configured using the `obi.testGroups {}` DSL structure. The default test group called **regression** is already built in, and by default recursively tests any analyses in the `/Shared/Regression` folder. So out of the box, if we have any analyses that we want to run before and after our committed changes, we just add those analyses to `/Shared/Regression` and the before and after logical SQL and results will be compared during each build.

If we wanted to modify settings in our `regression` test group, we can see an example of that below. We could use the same DSL to create whole new test groups as well:

**NOTE: this is just a sample, and does not map to any real folders or users in the environment. It is included only for reference.**

```gradle
obi.testGroups {
  regression {
    libraryFolder = '/shared/sales-analytics:/shared/sales-reports'
    compareText = false
    showOutput = false
    impersonateUser = 'sales-consumer'
  }
}
```

Here are a few of the configurations we can define in `obi.testGroups{}`:
* **libraryFolder:** The folder(s) in the presentation catalog that we want to test. This can be a colon (:) separated list of catalog folders, and Checkmate recursively includes every analysis in those folders.
* **compareText:** By default, Checkmate uses hash values of logical SQL and query results to facilitate faster comparisons, which improves performance when logical SQL is very complicated, or the analysis returns a lot of records in the result set. Setting this to `true` enables the full text search of the results. We are using the default value of `false`.
* **showOutput:** Output more information about the execution of each regression test. Feel free to set this to `true` to see the logical SQL being executed, any error stack generated, etc. We are using the default value of `false`.
* **impersonateUser:** Specify a user that you want to impersonate for this test group. This allows us to configure multiple test groups that run the same regression tests but with different security profiles. This requires that the `adminUser` specified above be granted the `impersonate` privilege, which is not granted by default.

Lets import our content from Git into OBIEE and then run results tests and include the results in our distribution file:

```bash
./gradlew obi:resultsWorkflow publish --console=plain
> Task :obi:connPoolsExport
> Task :obi:metadataBuild UP-TO-DATE
> Task :obi:metadataImport
> Task :obi:catalogBuild
> Task :obi:catalogImport
> Task :obi:connPoolsImport
> Task :obi:import
> Task :obi:metadataReload
> Task :obi:importWorkflow
> Task :obi:regressionResultsLibrary
> Task :obi:extractTestSuites UP-TO-DATE

> Task :obi:regressionResultsTest

com.redpillanalytics.obi.regression.suites.ResultsTest

  Test revision execution: /shared/Regression/Hierarchies/Product Hierarchy with Custom Group PASSED (5.1s)
  Test revision execution: /shared/Regression/Level Based/Level Based Measures : Full Products Revenue PASSED (3.7s)
  Test revision execution: /shared/Regression/Level Based/Level Based Measures : Full Qtr Revenue PASSED (3.6s)
  Test revision execution: /shared/Regression/Trellis/Trellis Charts 3 PASSED (3.9s)
  Test revision execution: /shared/Regression/Trellis/Trellis5 PASSED (3.7s)

SUCCESS: Executed 5 tests in 24.7s


> Task :obi:resultsTest
> Task :obi:resultsWorkflow
> Task :obi:assemble UP-TO-DATE
> Task :obi:check UP-TO-DATE
> Task :obi:build
> Task :obi:buildZip
> Task :obi:generatePomFileForBuildPublication
> Task :obi:publishBuildPublicationToMavenLocalRepository
> Task :obi:deployZip
> Task :obi:generatePomFileForDeployPublication
> Task :obi:publishDeployPublicationToMavenLocalRepository
> Task :obi:publish

BUILD SUCCESSFUL in 2m 25s
16 actionable tasks: 14 executed, 2 up-to-date
```
We now have the results of our tests saved in the distribution file. We can compare our results to the prior run by simply pulling that distribution file, running the current test results, and then comparing them. We use the *feature* version of our task, because that tells checkmate we are interested in the test results associated with the `feature` buildgroup, which is `1.0.0`:

```bash
./gradlew obi:featureTestWorkflow --console=plain
> Task :obi:connPoolsExport
> Task :obi:featureExtractBuild
> Task :obi:metadataBuild UP-TO-DATE
> Task :obi:metadataImport
> Task :obi:catalogBuild UP-TO-DATE
> Task :obi:catalogImport
> Task :obi:connPoolsImport
> Task :obi:metadataReload
> Task :obi:regressionResultsLibrary
> Task :obi:extractTestSuites UP-TO-DATE

> Task :obi:regressionResultsTest

com.redpillanalytics.obi.regression.suites.ResultsTest

  Test revision execution: /shared/Regression/Hierarchies/Product Hierarchy with Custom Group PASSED (10s)
  Test revision execution: /shared/Regression/Level Based/Level Based Measures : Full Products Revenue PASSED (6s)
  Test revision execution: /shared/Regression/Level Based/Level Based Measures : Full Qtr Revenue PASSED (5.7s)
  Test revision execution: /shared/Regression/Trellis/Trellis Charts 3 PASSED (6.9s)
  Test revision execution: /shared/Regression/Trellis/Trellis5 PASSED (6.6s)

SUCCESS: Executed 5 tests in 40.3s


> Task :obi:resultsTest

> Task :obi:regressionCompareTest

com.redpillanalytics.obi.regression.suites.CompareTest

  Test logical SQL comparison: /shared/Regression/Level Based/Level Based Measures : Full Products Revenue PASSED
  Test logical SQL comparison: /shared/Regression/Level Based/Level Based Measures : Full Qtr Revenue PASSED
  Test logical SQL comparison: /shared/Regression/Hierarchies/Product Hierarchy with Custom Group PASSED
  Test logical SQL comparison: /shared/Regression/Trellis/Trellis Charts 3 PASSED
  Test logical SQL comparison: /shared/Regression/Trellis/Trellis5 PASSED
  Test hash value comparison of results: /shared/Regression/Level Based/Level Based Measures : Full Products Revenue PASSED
  Test hash value comparison of results: /shared/Regression/Level Based/Level Based Measures : Full Qtr Revenue PASSED
  Test hash value comparison of results: /shared/Regression/Hierarchies/Product Hierarchy with Custom Group PASSED
  Test hash value comparison of results: /shared/Regression/Trellis/Trellis Charts 3 PASSED
  Test hash value comparison of results: /shared/Regression/Trellis/Trellis5 PASSED
  Test text comparison of results file: #path SKIPPED

SUCCESS: Executed 11 tests in 5s (1 skipped)


> Task :obi:compareTest
> Task :obi:featureCompareWorkflow
> Task :obi:assemble UP-TO-DATE
> Task :obi:check UP-TO-DATE
> Task :obi:build UP-TO-DATE
> Task :obi:buildZip
> Task :obi:generatePomFileForBuildPublication
> Task :obi:publishBuildPublicationToMavenLocalRepository
> Task :obi:deployZip
> Task :obi:generatePomFileForDeployPublication
> Task :obi:publishDeployPublicationToMavenLocalRepository
> Task :obi:publish
> Task :obi:import
> Task :obi:importWorkflow
> Task :obi:resultsWorkflow
> Task :obi:featureTestWorkflow

BUILD SUCCESSFUL in 2m 33s
18 actionable tasks: 15 executed, 3 up-to-date
```

# OBIEE 12c BAR Files
The first release of Checkmate for OBI pre-dated OBIEE 12c, and therefore, pre-dated the new BAR file functionality introduced 12c. We understood the need for a singular artifact that expressed the content for an OBIEE environment, and built it years before Oracle did. There's an interesting decision to be made when configuring Checkmate workflows: use the distribution file, or the BAR file?

There are pros and cons for each approache. The distribution file provides the added benefit of including incremental patch files and test result outcomes, neither of which can exist in the BAR file. The BAR import and export utilities are also painfully slow. However, the BAR file contains a lot of content not available using traditional OBIEE utilities and APIs: datasets, authorization details, search, actions, etc. The choice is not cut and dry.

In recent releases, Checkmate has consolidated the Git repository structure so that it's consistent whether we are using traditional utilties and APIs used in distribution file workflows, and those used with the BAR file. Checkmate adopted the BAR directory structure as the inspiration for how we store our content in Git, and we've modified our legacy distribution tools to understand that directory structure and use it for interacting content checked into Git. The content in this Quickstart was exported using `barExport`, so we can use either the distribution tools or the BAR tools for interacting with it.

So first, let's build a BAR file. We need to change our `contentPolicy` to `bar` so we can use all the high level container tasks:

```gradle
obi {
  middlewareHome = '/opt/oracle/product/12.2.1.4.0'
  domainHome = '/opt/oracle/config/domains/bi'
  compatibility = '12.2.1.4'
  adminUser = 'weblogic'
  adminPassword = 'Admin123'
  repositoryPassword = 'Admin123'
  contentPolicy = 'bar'
}
```

When using OBIEE's BAR utilities, the only way to create the BAR file is by exporting it from an instance of OBIEE. This was an unfortunate choice because it reinforces the pattern common in aging legacy tools that the *gold copy* of our content and code exist *in environments* instead of in source control systems like Git. And checking whole BAR files into source control is silly: it would be unusable with even the simplies branching strategy.

We've written Checkmate to enhance the BAR process by dismantling the BAR file on the way into source control, and reassembling on the way out. The BAR file is really just a zip file with a manifest, so of course we unzip before copying it to source control. But we also convert the binary metadata file into MDS-XML for storage in Git, and rebuild the binary during the BAR assembly process.

So let's build an publish a BAR file from source control, similar to what we did with distribution files:

```bash
./gradlew obi:publish --console=plain
> Task :obi:barSync
> Task :obi:barBuild
> Task :obi:barZip
> Task :obi:generatePomFileForBarPublication
> Task :obi:publishBarPublicationToMavenLocalRepository
> Task :obi:publish

BUILD SUCCESSFUL in 37s
5 actionable tasks: 5 executed
```
We can take a look at our build directory and see a subset of the generated content. Although a BAR file is different from a distribution file, the final artifact exists in the same `distributions` directory:

```bash
ls -ltr obi/build/*
obi/build/bar:
total 0
drwxr-xr-x 9 oracle dba 288 Mar  5 08:13 current

obi/build/misc:
total 0
drwxr-xr-x 2 oracle dba 64 Mar  5 08:13 xml-variables

obi/build/distributions:
total 3108
-rw-r--r-- 1 oracle dba 2490752 Mar  5 08:13 obi-bar-1.0.0.zip

obi/build/publications:
total 0
drwxr-xr-x 3 oracle dba 96 Mar  5 08:13 bar
```

Since BAR files are really just ZIP files, we continue to use that extension instead of the `.bar` extension, renaming the artifacts if necessary prior to calling any of the BAR utilities. Let's add a BAR artifact dependency on the `promote` build group, so we can use a BAR file for downstream deployments:

```gradle
dependencies {
  obiee 'com.redpillanalytics:checkmate-obi:+'
  feature 'obiee:obi-build:+'
  release 'obiee:obi-build:1.0.0'
  promote 'obiee:obi-deploy:+'
  promote 'obiee:obi-bar:+'
}
```

Now, let's promote our BAR file to a downstream environment, but let's exclude all *action* content, all *search* content, and all *authorization* content:

```bash
./gradlew obi:promoteBarImport --noaction --nosearch --noauthorization --console=plain
> Task :obi:promoteBarImport
> Task :obi:promoteBarImport

BUILD SUCCESSFUL in 1m 46s
1 actionable task: 1 executed
```
