# Checkmate for OBI 12.2.1.2 Quickstart
This quickstart provides basic understanding of the core features of Checkmate for OBI 12.2.1.2. The project folder includes sample OBIEE content from [SampleAppLite](http://docs.oracle.com/middleware/12212/biee/BIESG/GUID-E439E473-DD4D-48FE-9BF1-7AED4ADD73B6.htm#BIESG9340) already checked into the [`src/main`](src/main) directory.

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

The `plugins` block applies any desired plugins from the [Gradle Plugin Portal](https://plugins.gradle.org) using the unique ID associated with that plugin. In this case we are applying the Gradle-provided `maven-publish` plugin, as well as the `com.redpillanalytics.checkmate.obi` plugin, which is Checkmate for OBI.