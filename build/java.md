- [External variables used in templates](#external-variables-used-in-templates)
- [Variables with default values that can be overridden](#variables-with-default-values-that-can-be-overridden)
- [GitLab environment variables used in templates](#gitlab-environment-variables-used-in-templates)
- [Gradle Usage examples in .gitlab-ci.yml](#gradle-usage-examples-in-gitlab-ciyml)
- [Maven Примеры использования в .gitlab-ci.yml](#maven-примеры-использования-в-gitlab-ciyml)

## External variables used in templates
```yaml
.build_and_test_gradle:
JVM_IMAGE           # Specify your image with gradle, default value is gradle:jdk11-alpine
GRADLE
GRADLE_PARAMS
TESTCONTAINERS_HUB_IMAGE_NAME_PREFIX # You need to specify it if you use testcontainers

.build_image_jib:
JVM_IMAGE           # Specify your image with gradle, default value is gradle:jdk11-alpine
GRADLE
GRADLE_PARAMS
REGISTRY_PATH        # used in jib.gradle to build image path
IMAGE_NAME          # used in jib.gradle to build image path
IMAGE_TAG           # used in jib.gradle to get image tag
.build_and_test_maven:
JVM_IMAGE             # Specify your image with gradle, default value is docker.io/library/maven:3.8.4-openjdk-11-slim
MAVEN_OPTS            # Specify parameters for the JVM running maven
MAVEN_ADDITIONAL_OPTS # Specify any additional parameters for maven runtime
MAVEN_SETTINGS_PATH   # Specify path to your settings.xml, default value is ./cicd/settings.xml
TESTCONTAINERS_HUB_IMAGE_NAME_PREFIX # You need to specify it if you use testcontainers
REGISTRY_USER     # Hide this in Settings -> CI/CD -> Variables
REGISTRY_PASSWORD # Hide this in Settings -> CI/CD -> Variables

.build_with_maven_and_publish_jar:
JVM_IMAGE             # Specify your image with gradle, default value is docker.io/library/maven:3.8.4-openjdk-11-slim
MAVEN_OPTS            # Specify parameters for the JVM running maven
MAVEN_ADDITIONAL_OPTS # Specify any additional parameters for maven runtime
MAVEN_SETTINGS_PATH   # Specify path to your settings.xml, default value is ./cicd/settings.xml
TESTCONTAINERS_HUB_IMAGE_NAME_PREFIX # You need to specify it if you use testcontainers
DESTINATION_REPOSITORY_ID # Specify destination repository id (name)
REGISTRY_USER     # Hide this in Settings -> CI/CD -> Variables
REGISTRY_PASSWORD # Hide this in Settings -> CI/CD -> Variables
```

## Variables with default values that can be overridden
```yaml
.build_and_test_gradle:
JVM_IMAGE: "gradle:jdk11-alpine"
PIPELINE_CACHE_KEY: $CI_COMMIT_REF_NAME-$CI_PIPELINE_ID
REGISTRY_HOST: #default points to hub.docker.com
DOCKER_HOST: tcp://dind:2375
DOCKER_TLS_CERTDIR: ""
DOCKER_DRIVER: overlay2
TESTCONTAINERS_RYUK_DISABLED: "true"
TESTCONTAINERS_CHECKS_DISABLE: "true"
GRADLE_OPTS: "-Dorg.gradle.daemon=false"
GRADLE_USER_HOME: ".gradle/"
JUNIT_PATH: "./build/test-results/**/*.xml"

.build_image_jib:
JVM_IMAGE: "gradle:jdk11-alpine"
REGISTRY_HOST: #default points to hub.docker.com. Used in jib.gradle to build image path
REGISTRY_PATH: $REGISTRY  #considering you use 'define-registry stage'
PIPELINE_CACHE_KEY: $CI_COMMIT_REF_NAME-$CI_PIPELINE_ID
GRADLE_OPTS: "-Dorg.gradle.daemon=false"
GRADLE_USER_HOME: ".gradle/"
GRADLE_PARAMS: "-Dgradle.wrapperUser=$REGISTRY_USER -Dgradle.wrapperPassword=$REGISTRY_PASSWORD -Pregistry_user=$REGISTRY_USER -Pregistry_password=$REGISTRY_PASSWORD"

.build_and_test_maven:
JVM_IMAGE: "docker.io/library/maven:3.8.4-openjdk-11-slim"
DOCKER_HOST: tcp://dind:2375
DOCKER_TLS_CERTDIR: ""
DOCKER_DRIVER: overlay2
TESTCONTAINERS_RYUK_DISABLED: "true"
TESTCONTAINERS_CHECKS_DISABLE: "true"
MAVEN_OPTS: "-Dmaven.repo.local=./repository"
MAVEN_SETTINGS_PATH: "./cicd/settings.xml"
JUNIT_PATH: "./build/test-results/**/*.xml"

.build_with_maven_and_publish_jar:
JVM_IMAGE: "docker.io/library/maven:3.8.4-openjdk-11-slim"
REGISTRY_HOST: #default points to hub.docker.com
DOCKER_HOST: tcp://dind:2375
DOCKER_TLS_CERTDIR: ""
DOCKER_DRIVER: overlay2
TESTCONTAINERS_RYUK_DISABLED: "true"
TESTCONTAINERS_CHECKS_DISABLE: "true"
MAVEN_OPTS: "-Dmaven.repo.local=./repository"
MAVEN_SETTINGS_PATH: "./cicd/settings.xml"
JUNIT_PATH: "./build/test-results/**/*.xml"
DESTINATION_REPOSITORY_URL: "$REGISTRY_HOST/$DESTINATION_REPOSITORY_ID"
```

## GitLab environment variables used in templates
```yaml
.build_and_test_gradle:
$CI_COMMIT_REF_NAME
$CI_PIPELINE_ID

.build_image_jib:
$CI_COMMIT_REF_NAME
$CI_PIPELINE_ID
```

## Gradle Usage examples in .gitlab-ci.yml

Add the `ji b.grade` file with the image build settings to the project folder:
```
ext.imagePath = {
    def registry_host = System.getenv('REGISTRY_HOST')
    def registry_path = System.getenv('REGISTRY_PATH')
    def image_name = System.getenv('IMAGE_NAME')
    return [registry_host,registry_path,image_name].join('/')
}

ext.imageTag = {
    return System.getenv('IMAGE_TAG')
}

jib {
    from {
        image = 'eclipse-temurin:17.0.6_10-jre-alpine'
        auth {
            username "$registryUser"
            password "$registryPassword"
        }
    }
    to {
        image = "${imagePath()}:${imageTag()}"
        auth {
            username "$registryUser"
            password "$registryPassword"
        }
    }
    container {
        user = 'www-data'
        jvmFlags = ['-Xms512m','-Xmx1024m']
        ports = ['8080']
        environment = [TZ: "Europe/Moscow",LANG: "en_US.UTF-8"]
        creationTime = "USE_CURRENT_TIMESTAMP"
    }
    new File(projectDir, "deploy.env").text = "image_path=${imagePath()}\nimage_tag=${imageTag()}\n"
}
```

Then add it to `build.gradle` plugin `Jb` and the contents of the `jib.gradle` file:
```
...

plugins {
    ...
    id 'com.google.cloud.tools.jib' version '3.3.1'
}

...

apply from: 'jib.gradle'

```

More info about `Jib`: https://github.com/GoogleContainerTools/jib/tree/master

After that, you can connect jobs to build and publish the image using `Jib` in the pipeline:

```yaml
########################################
###      Include template job        ###
###   .build_and_test_gradle         ###
########################################

include:
  - project: 'path-to-gitlab-repo/the-way-of-cicd-open-source-edition'
    ref: 'Major.minor.patch' # Please fill in with the desired tag
    file:
      - 'build/build-package.yml'

########################################
###   Create job in your pipeline    ###
###     extending included job       ###
########################################
build-via-gradle:
  stage: build
  extends:
    - .build_and_test_gradle
  variables:
    JVM_IMAGE: "eclipse-temurin:17.0.6_10-jdk-alpine"
    GRADLE: "./gradlew --stacktrace"
    GRADLE_PARAMS: "-Dgradle.wrapperUser=$REGISTRY_USER -Dgradle.wrapperPassword=$REGISTRY_PASSWORD -Pregistry_user=$REGISTRY_USER -Pregistry_password=$REGISTRY_PASSWORD"
    TESTCONTAINERS_HUB_IMAGE_NAME_PREFIX: "" #define if you use test containers
  before_script:
    - chmod +x gradlew

build-image:
  stage: publish-image
  extends:
    - .build_image_jib
  variables:
    REGISTRY_HOST: #default points to hub.docker.com
    IMAGE_NAME: "$CI_PROJECT_NAME"
    IMAGE_TAG: "${VERSION}"
    GRADLE: "./gradlew --stacktrace"
    GRADLE_PARAMS: "-Pregistry_user=$REGISTRY_USER -Pregistry_password=$REGISTRY_PASSWORD"
  before_script:
    - chmod +x gradlew
```

## Maven Примеры использования в .gitlab-ci.yml

```yaml
########################################
###      Include template job        ###
###     .build_and_test_maven        ###
########################################
include:
  - project: 'path-to-gitlab-repo/the-way-of-cicd-open-source-edition'
    ref: 'Major.minor.patch' # Please fill in with the desired tag
    file:
      - 'build/build-package.yml'

########################################
###   Create job in your pipeline    ###
###     extending included job       ###
########################################
build-via-maven:
  stage: build
  extends:
    - .build_and_test_maven
  variables:
    JVM_IMAGE: "docker.io/library/maven:3.8.4-openjdk-11-slim"
    MAVEN_ADDITIONAL_OPTS: "-Psome-profile"

build-via-maven-and-publish:
  stage: publish-image
  extends:
    - .build_with_maven_and_publish_jar
  variables:
    JVM_IMAGE: "docker.io/library/maven:3.8.4-openjdk-11-slim"
    MAVEN_ADDITIONAL_OPTS: "-X"
```