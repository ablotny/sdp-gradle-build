# SDP gradle build
================
This repo centralizes our gradle build script files that are used for java projects in combination with a cloudfoundry deployment. 

### Contents 
This gradle build will provide you with a basic project structure that was based on the *Continuous Delivery Build Pipleline* suggested by [Peter Niederwieser](https://twitter.com/pniederw) (see [link](http://www.slideshare.net/SpringCentral/cd-pipeline-gradlejenkins)). Additionally a Deployment to a [Cloudfoundry](http://www.gopivotal.com/platform-as-a-service/pivotal-cf) installation is provided.

Features:

* Basic Java Build
  * with build-in support (source sets, dependencies) of
    * unit tests
    * integration tests
    * system integration
  * SonarQube connector
  * CloudFoundry connector
  * Intellij support
* the CF deployment may also be used for non-Java-projects

Requirements that your project must fulfill to work with this gradle build

1. Add cloudfoundry gradle plugin to buildscript dependencies
2. Provide project variables for versioning, paths to cloudfoundry configuration file and the artefact that should be deployed
3. Provide credentials for cloudfoundry, nexus and sonar
4. Provide a cloudfoundry configuration file in groovy syntax
5. Include sdp.gradle in your project build.gradle

# Usage
## 1. Add cloudfoundry gradle plugin to buildscript dependencies (build.gradle)

    buildscript {
        repositories {
            maven { url "<REPLACE_WITH_SDP_REPO_URL>" }
        }
        dependencies {
            classpath 'org.cloudfoundry:cf-gradle-plugin:1.0.3.BUILD-SNAPSHOT'
        }
    }


## 2. Provide project variables (build.gradle)
The following variables must be set in the projects build.gradle

* `majorVersion`
* `minorVersion`
* `sourceBuildNumber`
* `buildConfigFilePath`
* `artefactType`
* `fileForCloundfoundryDeployment`
* `description`

### Example

    project.ext {
        majorVersion = 0
        minorVersion = 4
        sourceBuildNumber = project.hasProperty("sourceBuildNumber") ? project.getProperty("sourceBuildNumber") : "NO-BUILD-NR"

        buildConfigFilePath = "$rootDir/gradle/config/buildConfig.groovy"
        artefactType = "war"
        filePathForCloundfoundryDeployment = "$buildDir/libs/${project.name}-${majorVersion}.${minorVersion}.${sourceBuildNumber}.war"

        description = "A sample app"
    }


## 3. Provide credentials (gradle.properties)

Provide credentials for cloudfoundry, nexus and sonar by setting the following properties in your gradle.properties

* `cf.username`
* `cf.password`
* `nexus.username`
* `nexus.password`
* `sonar.username`
* `sonar.password`

### Example

    # cloudfoundy credentials
    cf.username=example.user@asideas.de
    cf.password=secretPasswd

    # nexus credentials
    nexus.username=exampleUser
    nexus.password=secretPasswd

    # sonar credentials
    sonar.username=sonar
    sonar.password=sonar



## 4. Provide a cloudfoundry configuration file (buildConfig.groovy)

### Example

    binaryRepository {
        baseUrl = '<REPLACE_WITH_SDP_NEXUS_URL>'
        releaseUrl = "$baseUrl/content/repositories/sdp-releases"
    }

    cloudfoundry {
        target = "<REPLACE_WITH_CF_API_URL>"
        organization = "Playground"
        domain = "<REPLACE_WITH_CF_DOMAIN>"

        services {
            'upsi-logstash-drain' {
                label = 'user-provided'
                bind = true
            }
        }
    }

    environments {

        def baseURL = "example-project"

        development {
            cloudfoundry {
                application = "$baseURL-development"
                host = "$baseURL-development"
                space = 'development'
            }
        }

        staging {
            cloudfoundry {
                application = "$baseURL-staging"
                host = "$baseURL-staging"
                space = 'staging'
            }
        }

        production {
            cloudfoundry {
                application = "$baseURL-production"
                host = baseURL
                space = 'production'
            }
        }
    }


## 5. Include sdp.gradle (build.gradle)

Apply the sdp.gradle file in the build script of your application with:

    apply from: "https://raw.githubusercontent.com/as-ideas/sdp-gradle-build/master/sdp.gradle"
    
## 6. Project Structure    
    (root)
    |- build.gradle
    |- gradlew
    |- gradlew.bat
    |- settings.gradle
    |- gradle
    |    |- config
    |    |   |- buildConfig.groovy
    |    |- wrapper  
    |- src
        |- main
        |    |- java
        |    |- resources
        |    |- webapp   
        |- test
        |    |- java
        |    |- resources
        |- integrationTest
        |    |- java
        |    |- resources        
        |- systemIntegrationTest
        |    |- java
        |    |- resources        

## 7. Connect with Jenkins Build Pipeline

You can trigger the creation of a new build pipeline by sending a build-request for the Auto_Build-job on the sdp-jenkins.
The request must contain the following two parameters as well as the jenkins credentials.

* `APP_NAME` - is used in the created jenkins-jobs as part of their job name
* `GIT_URL` - provides the location where your project is stored in the git-repository

Example:

    curl -i -vv -X POST http://<REPLACE_WITH_JENKINS_USER>:<REPLACE_WITH_JENKINS_PASSWD>@<REPLACE_WITH_JENKINS_URL>/job/Auto_Build/buildWithParameters?APP_NAME=<REPLACE_WITH_APPNAME>&GIT_URL=<REPLACE_WITH_APP_GIT_URL>