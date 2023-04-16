# Gradle

Enable caching in `GRADLE_USER_HOME`, e.g. `~/.gradle/gradle.properties`

```gradle
org.gradle.caching=true
```

## Linting

[Nebula](https://github.com/nebula-plugins/gradle-lint-plugin) is recommended by [gradle](https://docs.gradle.org/current/userguide/performance.html).

## Build precedence

According to the [gradle docs](https://docs.gradle.org/current/userguide/build_environment.html#sec:gradle_configuration_properties), gradle unintuitively looks in the following order:

1. command line, as set using -D.

2. gradle.properties in [GRADLE_USER_HOME](https://docs.gradle.org/current/userguide/directory_layout.html#dir:gradle_user_home) directory, e.g. `~/.gradle`

3. gradle.properties in the project’s directory, then its parent project’s directory up to the build’s root directory, e.g. `my-project/gradle.properties`.s

4. gradle.properties in Gradle installation directory, e.g. `/c/Tools/gradle/gradle-7.4/bin/gradle`. `9 mins 29.287 secs.`
