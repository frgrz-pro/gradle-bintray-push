gradle-bintray-push
===============

This fork is just some small modifications to Chris Banes original gradle-mvn-push script.
The differences here are:
- Release upload goes to bintray.com (jcenter)
- Snapshot upload goes to oss.jfrog.org (oss-snapshot-local)
- Added option to modify VERSION_NAME from environment variables (useful for CI)
- Added option to use username, password from environment variables (useful for CI)
- Fixes the JavaDoc generation (https://github.com/chrisbanes/gradle-mvn-push/pull/51)
- Added local maven support (https://github.com/chrisbanes/gradle-mvn-push/pull/41)
- Added POM_ARTIFACT_URL optional property. This is makes to easier to have an artifact with one artifactId but the name on jcenter something else.

To use those environment variables on CI just export them

```
export NEXUS_USERNAME=<your bintray username>
export NEXUS_PASSWORD=<your bintray api key>
export VERSION_NAME_EXTRAS=-RC1
```

Below is the original readme with adjustments for those params
============

See this blog post for more context on this 'library': [http://chris.banes.me/2013/08/27/pushing-aars-to-maven-central/](http://chris.banes.me/2013/08/27/pushing-aars-to-maven-central/).


## Usage

### 1. Have a working Gradle build
This is upto you.

### 2. Update your home gradle.properties

This will include the username and password to upload to the Maven server and so that they are kept local on your machine. The location defaults to `USER_HOME/.gradle/gradle.properties`.

It may also include your signing key id, password, and secret key ring file (for signed uploads).  Signing is only necessary if you're putting release builds of your project on maven central.

```properties
NEXUS_USERNAME=<your bintray username>
NEXUS_PASSWORD=<your bintray api key>

signing.keyId=ABCDEF12
signing.password=n1c3try
signing.secretKeyRingFile=~/.gnupg/secring.gpg
```

#### 2.1 Alternative, use environment variables

```
export NEXUS_USERNAME=<your bintray username>
export NEXUS_PASSWORD=<your bintray api key>
```

### 3. Create project root gradle.properties
You may already have this file, in which case just edit the original. This file should contain the POM values which are common to all of your sub-project (if you have any). For instance, here's [ActionBar-PullToRefresh's](https://github.com/chrisbanes/ActionBar-PullToRefresh):

```properties
VERSION_NAME=0.9.2-SNAPSHOT
VERSION_CODE=92
GROUP=com.github.chrisbanes.actionbarpulltorefresh

POM_DESCRIPTION=A modern implementation of the pull-to-refresh for Android
POM_URL=https://github.com/chrisbanes/ActionBar-PullToRefresh
POM_SCM_URL=https://github.com/chrisbanes/ActionBar-PullToRefresh
POM_SCM_CONNECTION=scm:git@github.com:chrisbanes/ActionBar-PullToRefresh.git
POM_SCM_DEV_CONNECTION=scm:git@github.com:chrisbanes/ActionBar-PullToRefresh.git
POM_LICENCE_NAME=The Apache Software License, Version 2.0
POM_LICENCE_URL=http://www.apache.org/licenses/LICENSE-2.0.txt
POM_LICENCE_DIST=repo
POM_DEVELOPER_ID=chrisbanes
POM_DEVELOPER_NAME=Chris Banes
```

The `VERSION_NAME` value is important. If it contains the keyword `SNAPSHOT` then the build will upload to the snapshot server, if not then to the release server.


### 4. Modify the version name from environment variables

If there's an environment variables called `VERSION_NAME_EXTRAS`, its value will get appended at the end of VERSION_NAME.
This can be very powerful when running from CI. For example, to have one SNAPSHOT per branch, you could

```
export VERSION_NAME_EXTRAS=-master-SNAPSHOT
```
in this case it will be uploaded to the snapshot server and indicates it's from the master branch.

### 5. Create gradle.properties in each sub-project
The values in this file are specific to the sub-project (and override those in the root `gradle.properties`). In this example, this is just the name, artifactId and packaging type:

```properties
POM_NAME=ActionBar-PullToRefresh Library
POM_ARTIFACT_ID=library
POM_ARTIFACT_URL=actionbar-library
POM_PACKAGING=aar
```

### 5. Call the script from each sub-modules build.gradle

Add the following at the end of each `build.gradle` that you wish to upload:

```groovy
apply from: 'https://raw.githubusercontent.com/sensorberg-dev/gradle-bintray-push/master/gradle-bintray-push.gradle'
```

### 6. Build

You can now build and push to jfrog/jcenter

```bash
$ gradle clean build uploadArchives
```

Build and install on local Maven
```bash
$ gradle clean build installArchives
```
### 7. Inter-module dependency
If you modules have dependencies on each other (e.g. compile project(':other_module')), then you should do one of the following for proper POM generation

- option A: top level build.gradle
```groovy
allprojects {
    version = VERSION_NAME
    group = GROUP
}
```
- option B: top level gradle.properties
```
version=1.0.0
group=<name here>
```
more info: https://stackoverflow.com/questions/45078381/gradle-library-with-multiple-modules


### Other Properties

`RELEASE_REPOSITORY_URL` and `SNAPSHOT_REPOSITORY_URL` got removed from the original script

## License

    Copyright 2013 Chris Banes

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
