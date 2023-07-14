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
- the [java flight recorder](https://docs.oracle.com/javacomponents/jmc-5-4/jfr-runtime-guide/about.htm#JFRUH171)

## Heap Dumps

To [create a heap dump](https://www.baeldung.com/java-heap-dump-capture) the easiest way is to use `jcmd`:

```bash
# use jps to get PID
jcmd myPID GC.heap_dump /path/to/where/i/store/heap.hprof
```

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
# altogether example
~/projects/FlameGraph/stackcollapse-jstack.pl stack.txt | iconv -f iso-8859-1 -t utf-8 > stack.folded && ~/projects/FlameGraph/flamegraph.pl --color=java stack.folded > stack.svg
```

## Programming

### Immutablility

A class has to satisfy five conditions to be immutable, here an extract from item 17 of Joshua Bloch's [Effective Java](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/):

> 1. **Don't provide methods that modify the object's state** (known as mutators).
> 2. **Ensure that the class can't be extended**. This prevents careless or malicious
> subclasses from compromising the immutable behavior of the class by
> behaving as if the object's state has changed. Preventing subclassing is
> generally accomplished by making the class final, but there is an alternative
> that we'll discuss later.
> 3. **Make all fields final**. This clearly expresses your intent in a manner that is en-
> forced by the system. Also, it is necessary to ensure correct behavior if a refer-
> ence to a newly created instance is passed from one thread to another without
> synchronization, as spelled out in the memory model [JLS, 17.5; Goetz06, 16].
> 4. **Make all fields private**. This prevents clients from obtaining access to
> mutable objects referred to by fields and modifying these objects directly.
> While it is technically permissible for immutable classes to have public final
> fields containing primitive values or references to immutable objects, it is not
> recommended because it precludes changing the internal representation in a
> later release (Items 15 and 16).
> 5. **Ensure exclusive access to any mutable components**. If your class has any
> fields that refer to mutable objects, ensure that clients of the class cannot obtain
> references to these objects. Never initialize such a field to a client-provided
> object reference or return the field from an accessor. Make defensive copies
> (Item 50) in constructors, accessors, and readObject methods (Item 88).

Some advantages of immutable objects are (see same source):

- Immutable objects are simple.
- Immutable objects are inherently thread-safe; they require no synchronization.
- Immutable objects can be shared freely.
- Immutable objects can share their internals (like negating `BigInteger`: The new and old object share their magnitude array, but differ in sign).
- Immutable objects make great building blocks for other objects, e.g. keys in maps or in sets
- Immutable objects provide failure atomicity for free

Disadvantages:

- The major disadvantage of immutable classes is that they require a separate object for each distinct value, especially when objects are big. Ideas to cirumvent this:
  - If costly operations are predictable, implement them with a package-private companion class that does several steps with a mutable object and finally returns an immutable object (see `BigInteger` with modular exponentitation)
  - If not predicatable, provide a public companion class like `StringBuilder`

## Trivia

- to set the Java Home Path for at least Eclipse projects like the IDE or Memory Analyzer, use the `vm` argument in the `ini` file:

    ```bash
    -vm
    C:\Tools\jdk-17.0.2\bin\javaw.exe
    ```

- main args like `gradle bootRun --args="--spring.profiles.active=manager,dev"` (see args) are handled as one argument `--spring.profiles.active=manager,dev` and not splitted artificially
