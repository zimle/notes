# Ant

## Globbing with ant in execs

Globbing is not that simple. The usual shell expansion is `**/*.{js,xml}` is not recognized in `ant`. The only way to make it work in `ant` seems to be the following (see last `arg`):

```xml
    <!-- get latest git commit date and revision number of current branch where production code has changed -->
    <target name="getGitRevision">
        <exec executable="git" outputproperty="git.revision">
            <arg value="log" />
            <!-- only latest commit -->
            <arg value="-1" />
            <!-- define date format we will use -->
            <arg value="--date=format:%Y-%m-%d_%H_%M_%S" />
            <!-- only return the short hashNumber_date, e.g. 2023-01-04_13:47:39_15f6fb0 -->
            <arg value="--pretty=format:%cd_%h" />
            <!-- Only consider commits where production code has changed. Beware of defined basedir.
                If we change docs, a build will be started but not overwrite the previous one avoiding build junk,
                see Jenkinsfile. Globbing like *.{js,xml,properties} does not work with ant -->
            <arg value="--" />
            <arg value="**/*.js" />
            <arg value="**/*.xml" />
            <arg value="**/*.properties" />
        </exec>
    </target>
```
