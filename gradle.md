# Gradle

Most basic issues can be achieved by using the help functions:

```bash
# get overview of help functions
./gradlew help
# get help on cli functionality
./gradlew --help
# get overview of available tasks
./gradlew tasks
# get help on specific task
gradlew help --task <task>
```

Often, gradle tasks depend on each other and it is not visible from the help functions what exactly a task does. To see this, use the `--dry-run` flag:

```bash
# see what tasks are executed with a specific gradle task, e.g. check
./gradlew check --dry-run
:bootBuildInfo SKIPPED
:processResources SKIPPED
:compileJava SKIPPED
:generateGitProperties SKIPPED
:classes SKIPPED
:checkstyleMain SKIPPED
:checkstyleNohttp SKIPPED
:compileTestJava SKIPPED
:processTestResources SKIPPED
:testClasses SKIPPED
:checkstyleTest SKIPPED
:test SKIPPED
:integrationTest SKIPPED
:modernizer SKIPPED
:pmdMain SKIPPED
:pmdTest SKIPPED
:spotlessInternalRegisterDependencies SKIPPED
:spotlessJava SKIPPED
:spotlessJavaCheck SKIPPED
:spotlessMisc SKIPPED
:spotlessMiscCheck SKIPPED
:spotlessYaml SKIPPED
:spotlessYamlCheck SKIPPED
:spotlessCheck SKIPPED
:check SKIPPED
```

## Start / stop application

```bash
# E.g. start Spring Boot application
./gradlew bootRun
# stop application
./gradlew -stop
```

## Tests

If running tests under different environment setups, but with the same test sources, use the flag `--rerun-tasks` to circumvent the caching.

```bash
# run all tests except integration tests
gradle test
# do not forget to set the JDK if your Java Home is different ;-)
gradle test -Dorg.gradle.java.home="C:\\Tools\\jdk-17.0.2"
# run specific test class
gradle test --tests MyTestClass
# run specific test method
gradle test --tests MyTestClass.myTestMethod
# run all tests including integration tests
gradle integrationTest
# really run all tests including integration tests, even if source did not change
# (useful, as integration tests rely on ports, containers, etc. of host system which might change)
gradle integrationTest --rerun-tasks
# runs all tests and some additional checks like checkstyle
gradle check
```

## Linting

[Nebula](https://github.com/nebula-plugins/gradle-lint-plugin) is recommended by [gradle](https://docs.gradle.org/current/userguide/performance.html).

## Settings

Enable caching in `GRADLE_USER_HOME`, e.g. `~/.gradle/gradle.properties`

```gradle
org.gradle.caching=true
```

### Build precedence

According to the [gradle docs](https://docs.gradle.org/current/userguide/build_environment.html#sec:gradle_configuration_properties), gradle unintuitively looks in the following order:

1. command line, as set using -D.

2. gradle.properties in [GRADLE_USER_HOME](https://docs.gradle.org/current/userguide/directory_layout.html#dir:gradle_user_home) directory, e.g. `~/.gradle`

3. gradle.properties in the project's directory, then its parent project's directory up to the build's root directory, e.g. `my-project/gradle.properties`.s

4. gradle.properties in Gradle installation directory, e.g. `/c/Tools/gradle/gradle-7.4/bin/gradle`. `9 mins 29.287 secs.`

## Trivia

- to specify the java version (e.g. when `JAVA_HOME` is older than Java version in project), one can specify it via the flag `-Dorg.gradle.java.home=myJavaPath`, e.g. `./gradlew build -Dorg.gradle.java.home=myJavaPath`.

- use the flag `--refresh-dependencies` to circumvent using cached packages
