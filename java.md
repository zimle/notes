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

## Tooling within jdk

The jdk (at least more modern ones) provide some [nice toolings](https://docs.oracle.com/en/java/javase/17/docs/specs/man/index.html).
Here is an (to be extended) list:

- `javap`: Java disassembler providing information about classes and interfaces:

    ```bash
    # print put public accessors
    javap MyClass1.class MyClass$MyInnerClass.class
    # also print private members (note there is also -protected, -package, -verbose etc.)
    javap -p MyClass1.class MyClass$MyInnerClass.class
    ```

## Performance

Tools

- `jps` like the unix `ps` lists all java process ids currently running
- `jstack` prints java stacktraces
- the [java flight recorder](https://docs.oracle.com/javacomponents/jmc-5-4/jfr-runtime-guide/about.htm#JFRUH171)
  - JDK below 11: start the JVM with args `-XX:+UnlockCommercialFeatures -XX:+FlightRecorder` (order is important!)
  - start recording:
    - either use directly `java mission control` (executable `jmc`) in your `jdk` bin folder
    - or use `jcmd` executable in your `jdk` bin folder via `jcmd 1234 JFR.start duration=100s filename=flight.jfr` (example).
      See the [official documentation](https://docs.oracle.com/javacomponents/jmc-5-4/jfr-runtime-guide/comline.htm#JFRUH190) for available flags.
    - be sure to stop the recording via `jcmd 1234 JFR.stop` at some time if no duration given.
      If using `java mission control`, right click on the flight recording and dump it to see information
  - start viewing via `java mission control` (executable `jmc`)

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

## Effective Java (Josh Bloch)

### Item 17: Minimize Mutability

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

### Item 18: Favor composition over inheritance

Inheritance violates encapsulation: Implementation of superclass may change from release to release, and subclass must involve in tandem.

Inheritance can be safely used:

- within the same package
- on classes which are specifically designed and documented for extension

Inheritance is appropriate:

- when a genuine subtype relationship exists between subclass and superclass (though fragile when not within the same package)

Use composition (just pass the class you need into the constructor and forward all methods needed, in other words wrap it).

### Item 28: Prefer Lists to Arrays

Arrays differ from lists in two fundamental ways:

- Arrays are *covariant* (`Sub[]` is subclass of `Super[]`) whereas lists are *invariant* (`List<T>` and `List<U>` are unrelated for `T != U`):

    ```java
    // Fails at runtime
    Object[] objectArray = new Long[1];
    objectArray[0] = "I don't fit in"; // throws ArrayStoreException

    // Won't compile
    List<Object> ol = new ArrayList<Long>(); // Incompatible types
    ol.add("I don't fit in");
    ```

- Arrays are *reified*, meaning that arrays know and enforce there types at runtime.
    Generics, by contrast, are erasured at compile time and none of the following will compile:

    ```java
        new List<E>[]
        List<String>[]
        new E[]
    ```

    This is good as we could otherwise create bugs easily:

    ```java
    List<String>[] stringLists = new List<String>[1]; // won't compile, but assume
    List<Integer> intList = List.of(42);
    Object[] objects = stringLists; // ok because arrays are covariant
    objects[0] = intList; // runtype type of stringLists is List[] and of intList is List due to type erasure, so perfectly fine
    String s = stringLists[0].get(0); // compiler assures we get a string, but it is an Integer!
    ```

  - Hint: the only parametrized types that are *reifiable* are unbounded wildcard types such as `List<?>` or `Map<?,?>`

In summary, arrays and lists do not mix well. As type safety on compile time is much more precious, prefer lists over arrays if not *proven* to be a performance leak.

### Item 31: Use bounded wildcards to increase API-flexibility

Long story short: As the title suggests, use bounded wildcards (`<? extends T>` resp. `<? super T>` to increase API flexibility).

As a simple example:

```java
static void FruitSalad makeFruitSaladNoWildCard(List<Fruit> fruits) { // does not use wildcard
    return fruits.stream().map(Fruit::slice).reduce(new FruitSalad(), (salad, slices) -> salad.add(slices));
}

static void FruitSalad makeFruitSaladWithWildCard(List<? extendsFruit> fruits) { // uses wildcard
    return fruits.stream().map(Fruit::slice).reduce(new FruitSalad(), (salad, slices) -> salad.add(slices));
}

List<Banana> bananas = buyBananas();
makeFruitSaladNoWildCard(bananas); // won't compile as generics are invariant and not covariant as arrays, see Item 28
makeFruitSaladWithWildCard(bananas); // will compile
```

A more advanced scenario is comparing the following signature:

```java
// first guess, but not as flexible as one would wish
static <T extends Comparable<T>> T max(List<T> list)
// most flexible version
static <T extends Comparable<? super T>> T max(List<? extends T> list)
```

So what is the difference?
Recall the following interface definitions as well as the compareTo method of the `Integer` class:

```java
public interface ScheduledFuture<V> extends Delayed, Future<V> {}
public interface Delayed extends Comparable<Delayed> {
   long getDelay(TimeUnit var1);
}
// from Integer class
@Override
public int compareTo(Integer anotherInteger) {
    return compare(this.value, anotherInteger.value);
}
```

With the first version, we could obviously apply the first `max` function to the interface `Delayed`, as `Delayed extends Comparable<Delayed>` holds and we would put in a List of `Delayed`.
But for `ScheduledFuture`, this is not true. We have to do two things:

1. Our method produces a `T` object, so it should be allowed for all classes that extend `T`, i.e. we need to allow `List<? extends T>` as the method argument. Thus, we can put in a list of `ScheduledFuture`.
1. The max function will need to compare the objects in the list. Therefore, our `?` type has to be comparable. This is guaranteed if `T` is a supertype of `?`.

Example from [Generics FAQ](http://www.angelikalanger.com/GenericsFAQ/FAQSections/TypeArguments.html#FAQ103):

```java
public class Collections {
  public static <T> void copy( List<? super T> dest, List<? extends T> src) {  // bounded wildcard parameterized types
      for (int i=0; i < src.size(); i++)
        dest.set(i, src.get(i));
  }
}
```

The upshot is the `PECS` ("Producer Extends, Consumer Super") rule (taken from <https://stackoverflow.com/questions/4343202/difference-between-super-t-and-extends-t-in-java>):

- "Producer Extends" - If you need a List to produce T values (you want to read Ts from the list), you need to declare it with ? extends T, e.g. List<? extends Integer>. But you cannot add to this list.

- "Consumer Super" - If you need a List to consume T values (you want to write Ts into the list), you need to declare it with ? super T, e.g. List<? super Integer>. But there are no guarantees what type of object you may read from this list.

- If you need to both read from and write to a list, you need to declare it exactly with no wildcards, e.g. List<Integer>.

## Value-based Classes

From the [oracle docs](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/doc-files/ValueBased.html):

>Some classes, such as `java.lang.Integer` and `java.time.LocalDate`, are *value-based*. A value-based class has the following properties:
>
> - the class declares only final instance fields (though these may contain references to mutable objects);
> - the class's implementations of equals, hashCode, and toString compute their results solely from the values of the class's instance fields (and the members of the objects they reference), not from the instance's identity;
> - the class's methods treat instances as freely substitutable when equal, meaning that interchanging any two instances x and y that are equal according to equals() produces no visible change in the behavior of the class's methods;
> - the class performs no synchronization using an instance's monitor;
> - the class does not declare (or has deprecated any) accessible constructors;
> - the class does not provide any instance creation mechanism that promises a unique identity on each method call-in particular, any factory method's contract must allow for the possibility that if two independently-produced instances are equal according to equals(), they may also be equal according to ==;
>    -the class is final, and extends either Object or a hierarchy of abstract classes that declare no instance fields or instance initializers and whose constructors are empty.
>
>When two instances of a value-based class are equal (according to `equals`), a program should not attempt to distinguish between their identities, whether directly via reference equality or indirectly via an appeal to synchronization, identity hashing, serialization, or any other identity-sensitive mechanism.
>
>Synchronization on instances of value-based classes is strongly discouraged, because the programmer cannot guarantee exclusive ownership of the associated monitor.
>
>Identity-related behavior of value-based classes may change in a future release. For example, synchronization may fail.

## Trivia

- to set the Java Home Path for at least Eclipse projects like the IDE or Memory Analyzer, use the `vm` argument in the `ini` file:

    ```bash
    -vm
    C:\Tools\jdk-17.0.2\bin\javaw.exe
    ```

- main args like `gradle bootRun --args="--spring.profiles.active=manager,dev"` (see args) are handled as one argument `--spring.profiles.active=manager,dev` and not splitted artificially
