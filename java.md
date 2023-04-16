# Java

To see the runtime jar version of an object `bla`, run `bla.getClass().getPackage().getImplementationVersion()`.

## Junit Performance

From <https://stackoverflow.com/questions/11826680/profile-junit-in-eclipse-indigo-using-visualvm>

You can use JVisualVM (`%JAVA_HOME%/bin/jvisualvm.exe`), but to use it with JUnit, you need to change the run configuration slightly.

1. Run the JUnit tests once, to create the run configuration
2. Edit the Run Configuration (`Run->Run Configurations...`)
3. In the `Test` tab check the box 'Keep JUnit running after test when debugging'
4. Rerun the test (with Debug). This will run the tests, but importantly, leave the JVM around, so that you can attach to it using JVisualVM.

If you wish to profile a specific section of the test, then setting a breakpoint before and after will enable you to start / stop profiling.

## Performance

Tools

- `jps` like the unix `ps` lists all java process ids currently running
- `jstack` prints java stacktraces

### From terminal

One-liner  to accumulate stack informations for profiling form [stack overflow](https://stackoverflow.com/questions/27228972/how-to-get-java-profiling-dump-for-creating-flame-graphs-on-the-mac)

```bash
while true; do jstack pidOfJavaProcess >> stack.txt; sleep 0.0013412; done
# with automatic determination of pid
pid=$(jps | rg Bootstrap | cut -f1 -d " ") && while true; do jstack "$pid" >> ~/perf/iwl_stack.txt; sleep 0.013412; done
```

Abort with `CTRL+C` when the interesting part of your application has finished and use [Brendan Gregg's flame graph tools](https://github.com/brendangregg/FlameGraph) to create a nice picture:

```bash
# convert for flame graph input
./stackcollapse-jstack.pl my_jstack > my_jstack.folded
# maybe encoding problems
iconv -f iso-8859-1 -t utf-8 my_jstack.folded > my_jstack_utf8.folded
# convert to flamegraph
./flamegraph.pl --color=java my_jstack.folded > my_jstack.svg
# alltogether example
~/projects/FlameGraph/stackcollapse-jstack.pl stack.txt | iconv -f iso-8859-1 -t utf-8 > stack.folded && ~/projects/FlameGraph/flamegraph.pl --color=java stack.folded > stack.svg
```
