# Checkmate for OBI Quickstart
This Quickstart demonstrates the basic functionality of the Checkmate for OBI Build Framework, using version 12.2.1.4 of Oracle Business Intelligence. The project folder includes sample OBIEE content from [SampleAppLite](https://docs.oracle.com/middleware/bi12214/biee/BIESG/GUID-7FCD90A3-E005-49BF-902F-30FBF9B41B07.htm#BIESG9340) already checked into the [`src/main`](src/main) directory. The repository also contains a [Docker compose](https://docs.docker.com/compose/overview/) configuration that provides an OBIEE 12.2.1.4 environment for walking through the Quickstart.

Checkmate is built using [Gradle](https://www.gradle.org): a declarative, [DSL](https://en.wikipedia.org/wiki/Domain-specific_language)-based build tool most commonly associated with building JVM-based software and [Android apps](https://developer.android.com/studio/index.html). Specifically, Checkmate is a series of [Gradle Plugins](https://guides.gradle.org/designing-gradle-plugins/). The following OBI functionality exists in the [com.redpillanalytics.checkmate.obi](https://plugins.gradle.org/plugin/com.redpillanalytics.checkmate.obi) plugin: source control integration, content versioning and publishing, automated regression and integration testing, and automated deployments.

A real delivery pipeline will likely have a Continuous Delivery server involved in this process, such as Jenkins, Travis CI, Google CloudBuild, etc. All of these CD servers have Gradle integration which makes Checkmate easy to use. But honestly, all CD servers also provide simple CLI integration for any tool, and this is the usual way we integrate Checkmate, especially when using [Jenkins Pipeline](https://jenkins.io/doc/book/pipeline/) configurations.

# Docker Compose for OBIEE
We have configured all the necessary steps for building an OBIEE 12.2.1.4 environment on Docker for use with this Quickstart. Beware: OBIEE is resource-intensive, and requires a considerable amount of memory and CPU for your Docker daemon. In my Docker configuration on my Mac, I set `CPUs:4` and `Memory:8GB`, but you might be able to make it work with smaller configurations.

First, clone the repository from GitHub using your favoriate Git client, or use the command-line:

```bash
git clone https://github.com/RedPillAnalytics/checkmate-examples.git
cd checkmate-examples
```

We've configured all the Docker Compose functionality to be usable with Gradle. Since we are working with the `obi` subproject folder, all of the Gradle tasks we call are prefixed with `obi:`. Additionally, I'm adding the `-q` option to remove some of the noise output to the screen by Docker Compose. This step starts up a Linux-based OBIEE instance, so it can be time-consuming. Also, the first time this statement is run, the rather-large OBIEE and database Docker images have to be downloaded, which can also take a long time:

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
docker exec -u oracle -w /workspace -ti rpa-checkmate-bi bash
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

And to ensure that our Gradle CLI is functioning inside the Docker container, let's run a basic Checkmate for OBI build. The first time a command is executed, Gradle will pull down any library dependencies used by Checkmate for OBI from the central Maven repository called [Bintray jCenter](https://bintray.com/bintray/jcenter), including the Gradle distribution itself, so we should see of this happening:

```bash
./gradlew obi:build

BUILD SUCCESSFUL in 30s
2 actionable tasks: 2 executed
```

# Basic Configuration
The heart of a Gradle build is the [build script](https://docs.gradle.org/current/userguide/tutorial_using_tasks.html#sec:hello_world), which by default is defined using a `build.gradle` file. This repository subdirectory already contains a functioning [build script](https://github.com/RedPillAnalytics/checkmate-examples/blob/master/obi/build.gradle), with all the necessary configurations already made, with several of the advanced features that we will apply later commented out.

The `plugins` block is the first and most important aspect to the build script: it applies any desired plugins from the [Gradle Plugin Portal](https://plugins.gradle.org) using the unique ID associated with that plugin:

```groovy
plugins {
  id 'com.redpillanalytics.checkmate.obi' version '9.1.15'
  id 'maven-publish'
  id 'com.avast.gradle.docker-compose' version "0.8.14"
}
```

* `com.redpillanalytics.checkmate.obi`: this is the Checkmate for OBI plugin. It enables all the features in this Quickstart.
* `maven-publish`: a core Gradle plugin that enables publishing to Maven repositories. Checkmate for OBI uses `maven-publish` with it's distribution files and for BAR files.
* `com.avast.gradle.docker-compose`: enables running the OBIEE 12.2.1.4 on Docker environment for this Quickstart.

With the Checkmate for OBI plugin applied, our Gradle [project](https://docs.gradle.org/current/userguide/tutorial_using_tasks.html#sec:projects_and_tasks) exists with all the core tasks enabled. We can use the [Gradle Wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html) checked in the root of this repository to see all the tasks associated with the `obi` project subdirectory, specified with the `obi:tasks` command. If we comment everything else out of the `build.gradle` file below the warning comment, the `obi:tasks` command gives us the basic default tasks for the plugin:

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

Checkmate Build tasks
---------------------
barBuild - Build and assemble the components for the BI Archive (BAR) file from SCM.
barSync - Synchronize the BAR build directory from SCM.
catalogBuild - Copy catalog from SCM to 'catalog/current' and generate catalog archive file 'catalog/current.catalog'.
metadataBuild - Build binary repository 'repository/current.rpd' from the MDS-XML repository in SCM.

Checkmate Distribution tasks
----------------------------
barZip - Create a BI Archive (BAR) file from the assembled component.
buildZip - Create a ZIP distribution archive for downstream deployments containing metadata and catalog artifacts.
deployZip - Create a ZIP distribution archive for downstream deployments containing metadata and catalog artifacts and incremental patches.

Checkmate Export tasks
----------------------
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

Checkmate Import tasks
----------------------
applyVersionJson - Set the 'project_version' repository variable to version 'unspecified'.
barImport - Import BI Archive (BAR) File into the 'ssi' Service Instance.
barImportSAL - Import SampleAppLite.bar BI Archive (BAR) File into the 'ssi' Service Instance.
barReset - Reset the 'ssi' Service Instance, equivalent to using an empty BI Archive (BAR) File.
catalogImport - Import the presentation catalog from SCM into the online presentation catalog using 'catalog/current'.
connPoolsImport - Import server connection pool information in JSON format from 'repository/conn-pools.json' to the target OBIEE server.
generateVersionJson - Generate JSON patch of version 'unspecified' for 'project_version' repository variable.
import - Execute all configured import tasks.
metadataImport - Import 'repository/current.rpd' into the online metadata repository.
variablesImport - Import server variable information in JSON format from 'repository/variables.json' to the target OBIEE server.

Checkmate SCM tasks
-------------------
catalogSCM - Synchronize 'catalog/current' with SCM and then commit.
metadataSCM - Synchronize 'repository/current.rpd' with SCM and then commit.
scmCheckout - Checkout a branch in the local SCM repository.
scmCommit - Issue a commit to the local SCM repository. Customize with 'scmMessage', 'scmCommitter', 'scmEmail', and 'scmCommitPath' build parameters.
scmPush - Push to the origin for the local SCM repository. Requires 'sourceBase' build parameter pointing to Git repo root directory, if running task outside of a Git repository.

Checkmate Services tasks
------------------------
metadataReload - Execute the 'Reload Files and Metadata' web service.

Checkmate Testing tasks
-----------------------
baselineTest - Execute all Baseline regression tests for the entire project.
compareTest - Execute all Compare regression tests for the entire project.
extractTestSuites - Extract the compiled test suites and copy them to the 'build/classes' directory.
revisionTest - Execute all Revision regression tests for the entire project.

Checkmate Workflow tasks
------------------------
baselineWorkflow - Import 'current' metadata and catalog artifacts, manage connection pools, and execute the Baseline Test Library.
importWorkflow - Import 'current' metadata and catalog artifacts while managing connection pools.
revisionWorkflow - Import 'current' metadata and catalog artifacts, manage connection pools, and execute the Revision Test Library.

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

Publishing tasks
----------------
generateMetadataFileForBuildPublication - Generates the Gradle metadata file for publication 'build'.
generateMetadataFileForDeployPublication - Generates the Gradle metadata file for publication 'deploy'.
generatePomFileForBuildPublication - Generates the Maven POM file for publication 'build'.
generatePomFileForDeployPublication - Generates the Maven POM file for publication 'deploy'.
publish - Publishes all publications produced by this project.
publishBuildPublicationToMavenLocal - Publishes Maven publication 'build' to the local Maven repository.
publishDeployPublicationToMavenLocal - Publishes Maven publication 'deploy' to the local Maven repository.
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

BUILD SUCCESSFUL in 2s
```

Checkmate for OBI enables the following Task Groups:
* **Checkmate Distribution**: Managing the creation and deletion of different types of OBIEE distribution files.
* **Checkmate Export**: Tasks that facilitate exporting content from an OBIEE instance into the build directory and eventually into source control.
* **Checkmate Import**: Tasks that facilitate importing content into an OBIEE instance from source control, or from artifacts downloaded from Maven.
* **Checkmate SCM**: Tasks for integrating 'checkout' and 'commit' workflows with [Git](https://git-scm.com). These tasks are for very complex workflows, and are generally unnecessary when using standard CI/CD processes.
* **Checkmate Services**: The `metadataReload` task, which executes *Reload Files and Metadata* in OBIEE.
* **Checkmate Testing**: Tasks for Regression Testing OBIEE.

The Maven Publish plugin enables the following Task Groups:
* **Publishing**: Publishing content to Maven repositories.

Gradle enables certain default Task Groups as well:
* **Build Setup**: For generating new projects and for generating the Gradle Wrapper.
* **Help**: Basic help tasks.
* **Verification**: Runs any configured checks enabled in the project.

# OBIEE Environment Configuration
Checkmate for OBI needs to know the basics about the OBIEE environment it will execute against. Keep in mind: with the Gradle Wrapper in the source control repository, we can use Checkmate for OBI on any environment where we can check out a Git repository without doing a Checkmate-specific installation. We use Checkmate **build parameters** to configure  properties for the OBIEE environment, as well as other things we'll see later.

Build parameters can be enabled one of four ways, in reverse-prioritized order... meaning the last item in the list overrides the second-to-the-last item, and so forth:
* Specified in the `build.gradle` file
* Specified in a `gradle.properties` file, which is a standard Java properties file existing in the project directory, or in the `$HOME/.gradle` directory.
* Specified with environment variables
* Specified using Gradle project properties, which are passed to the Gradle Wrapper command-line using `-P<property>=<value>`

Checkmate provides this degree of flexibility because many build properties are environment specific, and may need to change from one environment to the next. Additionally, some parameters--such as passwords--are sensitive, and need to be treated as such. For instance, [Jenkins Credentials](https://jenkins.io/doc/book/pipeline/syntax/#environment), which are typically used to store passwords in Continuous Delivery environments, are exposed as environment variables, so this is a very handy way to pass sensitive build parameters to Checkmate for OBI.

For the sake of simplicity and clarity, we'll declare all the build parameters in the `build.gradle` file... even the sensitive ones. Just remember... you would want to use another approach in a real delivery pipeline. We'll use the `obi{}` closure to define these properties:

```gradle
obi {
  middlewareHome = '/opt/oracle/product/12.2.1.4.0'
  domainHome = '/opt/oracle/config/domains/bi'
  compatibility = '12.2.1.4'
  adminUser = 'weblogic'
  adminPassword = 'Admin123'
  repositoryPassword = 'Admin123'
  contentPolicy = 'legacy'
  importWorkflowConns = true
}
```

Notes on a few of the parameters below:
* **domainHome:** Defaults to `<obi.middlewareHome>/user_projects/domains/bi`, but can be configured separately as we've done here.
* **compatibility:** Options are `12.2.1.4, 12.2.1.3, 12.2.1.2, 12.2.1.1, 12.2.1.0, 11.1.1.9 and 11.1.1.7`. There are subtle and not-so-subtle differences in the way Checkmate interacts with OBIEE in the different releases, so this parameter controls that behavior. The 11.x functionality and parameters will likely disappear in the near future.
* **contentPolicy:** This parameter controls the relationship between legacy import/export features, and the **BAR** functionality that exists in newer versions of 12c. Possible values include: `bar, mixed, legacy, legacy-metadata, legacy-catalog`. The main values for this parmater are `legacy` or `bar`, with all others being for mostly depracated use cases. Currently, we have the value set to `legacy`, which means we are building distribution files, instead of BAR files.
* **importWorkflowConns:** When we execute our promotion workflow later on, we do want Checkmate to automatically manage connection pools.

# Building and Publishing
The workflow for building OBIEE content usually occurs in the following steps:
* **Build:** The building of OBIEE deployment artifacts from source control. In source control we have the following checked in: the metadata repository as MDS-XML, and the presentation catalog in filesystem structure. The **build** steps involve building a binary RPD, as well as a catalog archive file of the Shared Folders of the catalog. Depending on the value of `contentPolicy`, we either build a legacy ZIP file, called a Checkmate *distribution* file, or a 12c *BAR* file. Either a distribrution file, or a BAR file, is supported in working with OBIEE 12c environments.
* **Publish:** Publishing distribution files or BAR files to one or more [Maven repositories](https://maven.apache.org/pom.html#Repositories).

Currently, we have the publication repositories and a version number configured but commented out in our `build.gradle`. Let's uncomment these lines to turn on our publication functionality:

```gradle
version = '1.0.0'

publishing {
  repositories {
    // publish to Maven local repository, which is usually ~/.m2
    // This is only for testing purposes
    mavenLocal()
  }
}
```

This will publish our OBI distribution files or BAR files to a [Maven Local](https://docs.gradle.org/current/userguide/publishing_maven.html#publishing_maven:install) repository, which defaults to `$HOME/.m2`. Obviously, this is for testing purposes only. In a real continuous delivery pipeline, content should be published to a real Maven-compatible repository. At the very least, publish to an S3 bucket, which Gradle supports, using the following syntax:

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

We're hard-coding our version number to `1.0.0` in the build.gradle file, although we would normally use a plugin such as the [axion-release-plugin](https://github.com/allegro/axion-release-plugin) to handle release management with semantic versioning and automatic version bumping

Why do we bother publishing our OBIEE distribution or BAR files as artifacts to Maven repositories? So we can pull that artifact later by simply referring to the version number. This allows us to automate deployments to downstream environments, as Checkmate for OBI can simply pull artifacts from Maven prior to deployment. It also allows us to automate regression testing. As you will see later in the Quickstart, we can declare published artifact versions as baselines for testing new builds generated by Checkmate for OBI. This gives us a convenient way to pull down any prior distribution for regression testing or integration testing purposes. We can also generate incremental patch files this way and include them in our distributin files: we simply pull down from Maven whatever distribution was last published to our **Production** server, and use that in the patch-creation process.

We'll run the `obi:build` task again, but this time we'll use the `--console=plain` option to get better screen output. We'll also add the `obi:clean` task to first clean out our build directory to ensure we re-generate all our artifacts:

```bash
./gradlew obi:clean obi:build --console=plain
> Task :obi:clean
> Task :obi:assemble UP-TO-DATE
> Task :obi:catalogBuild
> Task :obi:check UP-TO-DATE
> Task :obi:metadataBuild
> Task :obi:build

BUILD SUCCESSFUL in 32s
3 actionable tasks: 3 executed
```

You'll notice that the `build` task doesn't really do anything on its own: it's really just a container for two other tasks that do all the work: `metadataBuild` and `catalogBuild`. This introduces Gradle's powerful dependencies and ordering features, which uses a [DAG](https://docs.gradle.org/current/userguide/build_lifecycle.html) implementation.

Furthermore... we can run the entire Build, Bundle and Publish workflow by simply running the `publish` task, which has dependencies on building and bundling all the content.

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

BUILD SUCCESSFUL in 29s
8 actionable tasks: 6 executed, 2 up-to-date
```

In the output, you'll notice the *UP-TO-DATE* status that Checkmate is reporting when running the dependent tasks. Checkmate is written to take advantage of the [Gradle Incremental Build](https://docs.gradle.org/current/userguide/more_about_tasks.html#sec:up_to_date_checks) feature. The catalog and metadata build tasks are not executed again, because none of the task input and output files have changed. This keeps Checkmate from re-running tasks that it doesn't have to. Rerunning tasks can always be forced by providing the `--rerun-tasks` command-line option.

Let's take a look at what our Build, Bundle and Publish process generated. If we look at the `obi/build` directory, we can see all the things that Checkmate built, with a subset displayed below:

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

A build group allows us to declare a dependency on a prior artifact, and then get a bunch of new, dynamically generated tasks that belong to that build group. Checkmate contains three build groups by default:
* **feature:** used primarily to regression test new feature branches prior to their being merged into a mainline of code, usually the **develop** or **master** branch.
* **release:** used to regression test new releases prior to being deployed to downstream environments, or prior to being merged into release branches such as **master** or **release.x.x.x**.
* **promote:** used for promoting content to downstream environments.

Because these build groups are already built-in, we don't have to do much to enable them; all we have to do is declare a dependency on a prior artifact version, and the tasks in this build group will magically appear. Since we now have the 1.0.0 distribution published to our Maven Local repository, we can use that distribution as our dependency for these build groups. We could also define custom build groups using the `obi.buildGroups {}` DSL structure if for some reason the combination of **feature**, **release** and **promote** was not sufficient for our workflow.

The first thing we have to tell Checkmate for OBI is where to go looking for our prior distribution files. We're still using Maven Local for this. So let's comment out the following lines:

```gradle
repositories {
  // local maven repository for artifacts, which is usually ~/.m2
  // this is really only for testing purposes
  mavenLocal()
}
```

We'll make the following changes to our [`build.gradle`](build.gradle) file, which should be possible by simply commenting out a few lines:

```gradle
dependencies {
  // Using the Checkmate Testing library which is recommended.
  // obiee "com.redpillanalytics:checkmate-obi:9.1.15"
  // You can also use Baseline Validation Tool
  // The installation ZIP needs to be available in a Maven repository
  // obiee group: 'com.oracle', name: 'oracle-bvt', version: '12.2.1.0.0'

  // Dependencies on previous OBIEE builds
  feature 'obiee:obi-build:+'
  // feature 'obiee:obi-bar:+'
  release 'obiee:obi-build:1.0.0'
  // release 'obiee:obi-bar:1.0.0'
  // promote 'obiee:obi-deploy:+'
  // promote 'obiee:obi-bar:+'
}
```

There's a lot more in the dependencies closure that we'll discuss later. For now, we'll just focus on the two build group dependencies that we want to uncomment out:

```gradle
feature 'obiee:obi-build:+'
release 'obiee:obi-build:1.0.0'
```

The DSL might be a bit confusing, because we are using Gradle's built-in dependency resolution functionality to resolve our OBI artifacts. Basically, we use a Gradle [configuration](https://docs.gradle.org/current/dsl/org.gradle.api.artifacts.Configuration.html) to declare dependencies on artifacts that we want Checkmate to pull down and unzip whenever we use one of the tasks in that build group. We are declaring a particular distribution file... in this case, the **build** distribution, with a particular version. Notice for the **feature** build group, we simply have a plus sign (+): this signifies to Checkmate that we simply want to pull down the most recent distribution file. After you uncomment these two dependencies, pay attention to the new tasks that are enabled:

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

Checkmate Build tasks
---------------------
barBuild - Build and assemble the components for the BI Archive (BAR) file from SCM.
barSync - Synchronize the BAR build directory from SCM.
catalogBuild - Copy catalog from SCM to 'catalog/current' and generate catalog archive file 'catalog/current.catalog'.
featureCatalogCompare - Build incremental file 'catalog/feature-diff.txt' and 'catalog/feature-undiff.txt' using the 'feature' configuration.
featureCompare - Execute 'featureCatalogCompare' and 'featureMetadataCompare'.
featureMetadataCompare - Build incremental files 'repository/feature-patch.xml', 'repository/feature-unpatch.xml' and 'repository/feature-compare.csv' using the 'feature' configuration.
metadataBuild - Build binary repository 'repository/current.rpd' from the MDS-XML repository in SCM.
releaseCatalogCompare - Build incremental file 'catalog/release-diff.txt' and 'catalog/release-undiff.txt' using the 'release' configuration.
releaseCompare - Execute 'releaseCatalogCompare' and 'releaseMetadataCompare'.
releaseMetadataCompare - Build incremental files 'repository/release-patch.xml', 'repository/release-unpatch.xml' and 'repository/release-compare.csv' using the 'release' configuration.

Checkmate Distribution tasks
----------------------------
barZip - Create a BI Archive (BAR) file from the assembled component.
buildZip - Create a ZIP distribution archive for downstream deployments containing metadata and catalog artifacts.
deployZip - Create a ZIP distribution archive for downstream deployments containing metadata and catalog artifacts and incremental patches.
featureExtractBuild - Extract OBIEE dependencies for 'feature'.
releaseExtractBuild - Extract OBIEE dependencies for 'release'.

Checkmate Export tasks
----------------------
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

Checkmate Import tasks
----------------------
applyVersionJson - Set the 'project_version' repository variable to version '1.0.0'.
barImport - Import BI Archive (BAR) File into the 'ssi' Service Instance.
barImportSAL - Import SampleAppLite.bar BI Archive (BAR) File into the 'ssi' Service Instance.
barReset - Reset the 'ssi' Service Instance, equivalent to using an empty BI Archive (BAR) File.
catalogImport - Import the presentation catalog from SCM into the online presentation catalog using 'catalog/current'.
connPoolsImport - Import server connection pool information in JSON format from 'repository/conn-pools.json' to the target OBIEE server.
featureApplyVersionJson - Set the 'project_version' repository variable to version '1.0.0'.
featureCatalogImport - Import the presentation catalog from SCM into the online presentation catalog using 'catalog/feature'.
featureGenerateVersionJson - Generate JSON patch of version '1.0.0' for 'project_version' repository variable.
featureImport - Execute all configured import tasks for buildGroup 'feature'.
featureMetadataImport - Import 'repository/feature.rpd' into the online metadata repository.
generateVersionJson - Generate JSON patch of version '1.0.0' for 'project_version' repository variable.
import - Execute all configured import tasks.
metadataImport - Import 'repository/current.rpd' into the online metadata repository.
releaseApplyVersionJson - Set the 'project_version' repository variable to version '1.0.0'.
releaseCatalogImport - Import the presentation catalog from SCM into the online presentation catalog using 'catalog/release'.
releaseGenerateVersionJson - Generate JSON patch of version '1.0.0' for 'project_version' repository variable.
releaseImport - Execute all configured import tasks for buildGroup 'release'.
releaseMetadataImport - Import 'repository/release.rpd' into the online metadata repository.
variablesImport - Import server variable information in JSON format from 'repository/variables.json' to the target OBIEE server.

Checkmate Patch tasks
---------------------
featureCatalogPatch - Apply 'catalog/feature-diff.txt' to the online presentation catalog.
featureCatalogUnpatch - Apply 'catalog/feature-diff.txt' to the online presentation catalog.
featureMetadataPatch - Apply 'repository/feature-patch.xml' to the metadata repository in offline mode.
featureMetadataUnpatch - Apply 'repository/feature-unpatch.xml' to the metadata repository in offline mode
featurePatch - Execute all configured patch tasks for buildGroup 'feature'.
featureUnpatch - Execute all configured unpatch tasks for buildGroup 'feature'.
releaseCatalogPatch - Apply 'catalog/release-diff.txt' to the online presentation catalog.
releaseCatalogUnpatch - Apply 'catalog/release-diff.txt' to the online presentation catalog.
releaseMetadataPatch - Apply 'repository/release-patch.xml' to the metadata repository in offline mode.
releaseMetadataUnpatch - Apply 'repository/release-unpatch.xml' to the metadata repository in offline mode
releasePatch - Execute all configured patch tasks for buildGroup 'release'.
releaseUnpatch - Execute all configured unpatch tasks for buildGroup 'release'.

Checkmate SCM tasks
-------------------
catalogSCM - Synchronize 'catalog/current' with SCM and then commit.
featureCatalogMerge - Use OBIEE merging instead of SCM merging for presentation catalog.
featureMerge - Execute 'featureMetadataMerge' and 'featureCatalogMerge'.
metadataSCM - Synchronize 'repository/current.rpd' with SCM and then commit.
releaseCatalogMerge - Use OBIEE merging instead of SCM merging for presentation catalog.
releaseMerge - Execute 'releaseMetadataMerge' and 'releaseCatalogMerge'.
scmCheckout - Checkout a branch in the local SCM repository.
scmCommit - Issue a commit to the local SCM repository. Customize with 'scmMessage', 'scmCommitter', 'scmEmail', and 'scmCommitPath' build parameters.
scmPush - Push to the origin for the local SCM repository. Requires 'sourceBase' build parameter pointing to Git repo root directory, if running task outside of a Git repository.

Checkmate Services tasks
------------------------
metadataReload - Execute the 'Reload Files and Metadata' web service.

Checkmate Testing tasks
-----------------------
baselineTest - Execute all Baseline regression tests for the entire project.
compareTest - Execute all Compare regression tests for the entire project.
extractTestSuites - Extract the compiled test suites and copy them to the 'build/classes' directory.
revisionTest - Execute all Revision regression tests for the entire project.

Checkmate Workflow tasks
------------------------
baselineWorkflow - Import 'current' metadata and catalog artifacts, manage connection pools, and execute the Baseline Test Library.
featureBaselineWorkflow - Import 'feature' metadata and catalog artifacts, manage connection pools, and execute the Baseline Test Library.
featureImportWorkflow - Import 'feature' metadata and catalog artifacts while managing connection pools.
featureRevisionPatchWorkflow - Apply 'feature' metadata and catalog patches, manage connection pools, and execute the Revision Test Library.
featureRevisionWorkflow - Import 'feature' metadata and catalog artifacts, manage connection pools, and execute the Revision Test Library.
importWorkflow - Import 'current' metadata and catalog artifacts while managing connection pools.
releaseBaselineWorkflow - Import 'release' metadata and catalog artifacts, manage connection pools, and execute the Baseline Test Library.
releaseImportWorkflow - Import 'release' metadata and catalog artifacts while managing connection pools.
releaseRevisionPatchWorkflow - Apply 'release' metadata and catalog patches, manage connection pools, and execute the Revision Test Library.
releaseRevisionWorkflow - Import 'release' metadata and catalog artifacts, manage connection pools, and execute the Revision Test Library.
revisionWorkflow - Import 'current' metadata and catalog artifacts, manage connection pools, and execute the Revision Test Library.

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

BUILD SUCCESSFUL in 5s
1 actionable task: 1 executed
```

You should see a bunch of new tasks enabled that begin with *feature* and *release*. These tasks will perform whatever Checkmate requires, but will use the content inside the artifact to faciliate the tasks. In some cases... the build group tasks will use both the content in the distribution file as well as content checked into the Git repository. An example is `releaseCompare`, which will generate incremental patch files for both the repository and the catalog by comparing the content in the distribution file with whatever is in source control. Let's also publish again, so we can create a *deploy* distribution, which contains all the patch files, as well as the original repository and catalog artifacts. Expect to see some *UP-TO-DATE* checks as Checkmate for OBI skips tasks that don't need to be rerun:

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

BUILD SUCCESSFUL in 1m 28s
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

We generated all the incremental patch files, including the rollback patches, but the content of those patch files is empty, because there is currently no difference in what was published to version `1.0.0` and what is currently in source control; but you get the idea.

Now, let's enable the **promote** build group distribution file, which is what we use for deploying content to downstream environments. So we just need to uncomment the `promote 'obiee:obi-deploy:+'` line from our `build.gradle` file:

```gradle
dependencies {
  // Using the Checkmate Testing library which is recommended.
  // obiee "com.redpillanalytics:checkmate-obi:9.1.15"
  // You can also use Baseline Validation Tool
  // The installation ZIP needs to be available in a Maven repository
  // obiee group: 'com.oracle', name: 'oracle-bvt', version: '12.2.1.0.0'

  // Dependencies on previous OBIEE builds
  feature 'obiee:obi-build:+'
  // feature 'obiee:obi-bar:+'
  release 'obiee:obi-build:1.0.0'
  // release 'obiee:obi-bar:1.0.0'
  promote 'obiee:obi-deploy:+'
  // promote 'obiee:obi-bar:+'
}
```

This gives us a series of new tasks to work with the **promote** build group. We'll use the `promotePatch` task, which uses the metadata and catalog incremental patches from the **deploy** distribution file, and applies them to the online OBIEE instance:

```bash
./gradlew obi:promotePatch --console=plain
> Task :obi:promoteExtractDeploy
> Task :obi:promoteMetadataPatch
> Task :obi:promoteCatalogPatch
> Task :obi:promotePatch

BUILD SUCCESSFUL in 44s
3 actionable tasks: 3 executed
```

You'll notice that `promotePatch` is a container task for executing `promoteMetadataPatch` and `promoteCatalogPatch`, either of which can be run individually.

Promoting to downstream environments using incremental patches is a great solution for truly continuous environments that deploy often. But as the time between releases increases, so does the size and complexity of the incremental patches, and the possibility that automated application of these patches might fail. In these cases, we can simply use the `promoteImportWorkflow` task:

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

BUILD SUCCESSFUL in 30s
6 actionable tasks: 6 executed
```
Notice that our `promoteImportWorkflow` task manages connection pools for us. Before our new metadata content is imported into the instance, we save the connection pool information as a JSON file, and then re-import it once the import process is complete. You can see the connection pool files in the build directory:

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

The connection pool metadata included in SampleApp isn't the most compelling as it uses XML files, but you can see that the encrypted password is included, and although there are no variables included in this connection pool, if there were, our JSON file would capture those as well.

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

However, we recognize that the BAR file is the future for OBI, so we certainly support it. We can generate a BAR file using the `barExport` task, and can import a BAR file into OBIEE using the `<buildGroup>BarImport` task. Additionally, by simply setting `publishBar = true` in our build script, Checkmate for OBI will automatically generate and publish a BAR file to our Maven repository along with the distribution file.

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
