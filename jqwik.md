# jqwik

[jqwik](https://jqwik.net/) is a [property based testing library](https://en.wikipedia.org/wiki/Software_testing#Property_testing) inspired from [quickcheck](https://en.wikipedia.org/wiki/QuickCheck).

Here is minimal example from its homepage (with explaining comments by myself):

```java
import java.util.*;
import java.util.stream.*;

import net.jqwik.api.*;

```java
import java.util.*;
import java.util.stream.*;

import net.jqwik.api.*;

class FizzBuzzTests {
    
    // the test of a property: note that the integer i in the method signature is provided
    // by the method divisibleBy3
    @Property
    boolean every_third_element_starts_with_Fizz(@ForAll("divisibleBy3") int i) {
        return fizzBuzz().get(i - 1).startsWith("Fizz");
    }

    // Creates random integers divisible by 3 between 1 and 100
    @Provide
    Arbitrary<Integer> divisibleBy3() {
        return Arbitraries.integers().between(1, 100).filter(i -> i % 3 == 0);
    }

    // the method to be tested, usually of course a method from another classe
    private List<String> fizzBuzz() {
        return IntStream.range(1, 100).mapToObj((int i) -> {
            boolean divBy3 = i % 3 == 0;
            boolean divBy5 = i % 5 == 0;

            return divBy3 && divBy5 ? "FizzBuzz"
                : divBy3 ? "Fizz"
                : divBy5 ? "Buzz"
                : String.valueOf(i);
        }).collect(Collectors.toList());
    }
}
```

Note the important terminology:

- Properties (methods annotated with `@Property`) are the methods tested in the code (either via asserts are returning booleans). Their methods require arguments annotated with `@ForAll` for the passed constellations to be tested

- Providers are methods annotated with `@Provide` and return an object of type `Arbitrary<T>`. To use them for an argument in a property (method), annotate the argument with `@ForAll("stringNameOfMyProvider")`

- Suppliers have the same function as providers. They are *classes* implementing the interface `ArbitrarySupplier<T>` and can be referenced in a property via `@ForAll(supplier = MySupplier.class)`.
    Note that the interface `ArbitrarySupplier<T>` has default methods for `supplyFor(TypeUsage targetType)` and `get()`, for which at least one must be implemented (else, an exception is thrown).
    Above example with a Supplier:

    ```java
    class FizzBuzzTests {
        // the test of a property: note that the integer i in the method signature is provided
        // by the method divisibleBy3
        @Property
        boolean every_third_element_starts_with_Fizz(@ForAll(supplier = DivisibleBy3Supplier.class) int i) {
            return fizzBuzz().get(i - 1).startsWith("Fizz");
        }

        // Creates random integers divisible by 3 between 1 and 100
        class DivisibleBy3Supplier implements ArbitrarySupplier<Integer>  {
            @Override
            public Arbitrary<Integer> get() {
                return Arbitraries.integers().between(1, 100).filter(i -> i % 3 == 0);
            }
        }

        // the method to be tested, usually of course a method from another classe
        private List<String> fizzBuzz() {
            // ...
        }
    }
    ```

## Documentation

To document the generated tests cases that were run, you can annotate the tests with `@Report(Reporting.GENERATED)` like

```java
@Property
@Report(Reporting.GENERATED)
void myPropertyTest(@ForAll(supplier = MySupplier.class) MyClass random){
    // ...
}
```

## Structuring

JUnit5 tests can be structured within a class with the help of inner classes. These must be annotated with `@Nested`.
Likewise for jqwik, one can annotated inner classes with `@Group` to structure the tests further.

Tests can be simply disabled via the annotation `@Disabled("for whatever reason")` like in JUnit5.

## Determinism

Note that jqwik logs the seed used for properties (i.e. method annotated with `@Property`) which can be used to rerun all tests for this property.
Furthermore, jqwik saves the runs in its own database `.jqwik-database` and uses the last seed if a property failed on a run (can be configured).
Please ensure to add `.jqwik-database` to your `.gitignore` file.
