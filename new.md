# Haskell notes

## Structures

### Type

From the [wiki](https://wiki.haskell.org/Type): In Haskell, `types` are how you describe the data your program will work with.

```haskell
-- generell naming
data TypeName = ValueConstructors
-- examples
data Maybe a = Just a | Nothing
data Tree a = Branch (Tree a) (Tree a) | Leaf a
-- without arguments because it is an enum
data Bool = True | False
-- nested value constructors
data Shape = Circle Float Float Float | Rectangle Float Float Float Float
```

Note that value constructors are functions returning the specified type

```
ghci>:t True
True :: Bool
ghci>:t Circle
Circle :: Float -> Float -> Float -> Shape
```

### Type synonyms / newtype

Type synonyms just rename existing types to make the code more readable, e.g.

```haskell
type String = [Char]
```

In as similar fashion, `newtype` wraps an existing type into a record fashion

```haskell
newtype CharList = CharList { getCharList :: [Char] }

-- useful for easying pattern matching
newtype Pair b a = Pair { getPair :: (a,b) }
instance Functor (Pair c) where
    fmap f (Pair (x,y)) = Pair (f x, y)
```

allowing conversion between the old and new type (here: getCharList returns the type `[Char]`, whereas the constructor returns a `CharList`). Note that the compile will treat `newtype` as the old type internally, removing the overhead for wrapping/unwrapping if an algebraic data type would be defined.

### Recursive data types

Data Types can be recursive, e.g. a list implementation:

```haskell
-- Cons stands for the constructor name
data List a = Empty | Cons { listHead :: a, listTail :: List a} deriving (Show, Read, Eq, Ord)
-- example inititalization
a = 5 `Cons` Empty
-- or other way
b = Cons 5 Empty
-- we could also use another String starting with capital letter like ... "What" as in lists
data List a = Empty | What { listHead :: a, listTail :: List a} deriving (Show, Read, Eq, Ord)
a = 5 `What` Empty
-- To use sepcial symbols like ":?:", we have to
infixr 5 :?:
data List a = Empty | a :?: (List a) deriving (Show, Read, Eq, Ord)
a = 5 :?: Empty
```

### Algebraic Data Types

From the [wiki](https://wiki.haskell.org/Algebraic_data_type):

>This is a [type](https://wiki.haskell.org/Type) where we specify the shape of each of the elements. Wikipedia has a thorough discussion. "Algebraic" refers to the property that an Algebraic Data Type is created by "algebraic" operations. The "algebra" here is "sums" and "products":
>
>- "sum" is alternation (A | B, meaning A or B but not both)
>- "product" is combination (A B, meaning A and B together)

It uses the key word `data`

```haskell
data Pair = P Int Double
data Pair = I Int | D Double deriving (Show)
```

### Named Fields / Record Syntax

From <https://devtut.github.io/haskell/record-syntax.html#basic-syntax>: `records` are an extension of sum algebraic data type that allow fields to be named.

They provide automatically built in functionalities:

```haskell
-- defining data type
    data Car = Car {company :: String, model :: String, year :: Int} deriving (Show)

-- instantiate via named fields
c = Car {company="Ford", model="Mustang", year=1967}

-- automatically get functions:
ghci>:t company
company :: Car -> String

-- pattern matching
isMustang :: Car -> Bool
isMustang Car {model="Mustang"} = True
isMustang c = False

ghci>isMustang c
True

-- creating new objects (note that c is defined above)
ghci>c' = c {model="Bla"}
ghci>c'
Car {company = "Ford", model = "Bla", year = 1967}
ghci>isMustang c'
False
-- Haskells object are immutable, so we get a new object!
ghci>c
Car {company = "Ford", model = "Mustang", year = 1967}
```

Additional `pattern matching` rules:

```haskell
data Person = Person { age :: Int, name :: String }

-- normal
lowerCaseName :: Person -> String
lowerCaseName (Person { name = x }) = map toLower x

-- Pattern Matching with NamedFieldPuns
lowerCaseName :: Person -> String
lowerCaseName (Person { name }) = map toLower name

-- Pattern Matching with RecordWildcards
lowerCaseName :: Person -> String
lowerCaseName (Person { .. }) = map toLower name
```

### Type class

- Resource: <https://serokell.io/blog/haskell-typeclasses>

From ["Learn you a Haskell"](http://learnyouahaskell.com/types-and-typeclasses#typeclasses-101):

>A `type class` is a sort of interface that defines some behavior. If a type is a part of a typeclass, that means that it supports and implements the behavior the typeclass describes.

Don't interchange it with usual classed from Object Oriented Languages.

Built-in examples:

```haskell
Eq
Ord
Read
Show
Enum
Num
Integral
Floating
```

Here is how `Eq` is implemented in the Prelude:

```haskell
class Eq a where
-- mandatory: function headers
    (==) :: a -> a -> Bool
    (/=) :: a -> a -> Bool
-- optinal: function bodies
    x == y = not (x /= y)
    x /= y = not (x == y)
```

Here,

- `Eq` is called the `type class`
- `a` is a `type class variable` (or more generally a `type constructor`, see functor typeclass). It is just a placeholder (lowercase) needed for the definitions following.

Note that by defining the function bodies, it suffices to implement (==) or (/=) to implement all of `Eq` (and in this case prevents a faulty implementation of /=).

`Num` is also a `type class`. Here is an example of a function that needs `Num` because arbitrary types do not implement the `+` function:

```haskell
-- type a is specified to be instance of Num so it is ensured
-- that a + 1 makes sense
plusOne :: Num a => a -> a
plusOne a = a + 1
```

An example for a type that implements a type class (or is instance of), we define

```haskell
data TrafficLight = Red | Yellow | Green
-- here same as derive (Eq)
instance Eq TrafficLight where
    Red == Red = True
    Green == Green = True
    Yellow == Yellow = True
    _ == _ = False
```

### The Functor typeclass

The Functor typeclass `functor` is defined as follows

```haskell
-- careful: Functor laws are expected to hold, but not guaranteed by Haskell -> programmers duty to check
class Functor f where
    fmap :: (a -> b) -> f a -> f b
```

Notable here is that f is not a `type variable`, but a `type constructor` taking a type parameter. For example

```haskell
instance Functor Maybe where
    fmap f (Just x) = Just (f x)
    fmap f Nothing = Nothing
```

It's like applying a map to something boxed. However, we assume that only one boxed parameter is changed:

```haskell
data Either a b = Left a | Right b
instance Functor (Either a) where
    fmap f (Right x) = Right (f x)
    fmap f (Left x) = Left x
```

Every type that is instance of `functor` must be of kind `*->*` , that is, map one type to another. For example,

```haskell
-- A concrete type
ghci>:k Int
Int :: *
-- Mapping a concrete type to a concrete type
--  a -> Maybe a
ghci>:k Maybe
Maybe :: * -> *
-- Mapping two concrete types to a concrete type
-- Either a b -> Left a or Right b
ghci>:k Either
Either :: * -> * -> *
-- One type parameter set
ghci>:k Either Int
Either Int :: * -> *
```

### Monoid

Just a type class where the properties of a `monoid` in the mathmatical sense are listed

```haskell
-- takes concrete type as opposed to type class in functor
-- careful: Haskell does not enforce associativity, programmers duty
class Monoid m where
    mempty :: m
    mappend :: m -> m -> m
    mconcat :: [m] -> m
    mconcat = foldr mappend mempty -- at times more performant implementation needed
-- examples
instance Monoid (*) where
    mempty = 1
    mappend = (*)
instance Monoid [] where
    mempty = []
    mappend = (++)
```

### Applicative Functor

A functor allows the application of a function to a lifted value. An `applicative functor` also allows the application of a lifted function to a lifted value:

```haskell
# definition
class (Functor f) => Applicative f where
    pure :: a -> f a
    (<*>) :: f (a -> b) -> f a -> f b

# example
instance Applicative Maybe where
    pure = Just
    Nothing <*> _ = Nothing
    (Just f) <*> something = fmap f something

ghci> pure (+3) <*> Just 9
Just 12
ghci> Just (++"hahah") <*> Nothing
Nothing

# exported in Control.Applicative: Shorthand for for first lifting function
# then applying applicative functor
(<$>)::(Functorf)=>(a->b)->fa->fb
f<$>x=fmapfx
# example for <$>
1.  ghci>(++)<$>Just"johntra"<*>Just"volta"
2.  Just"johntravolta"
```

### Monad

A `monad` is a functor and an `applicative functor` that allows applying a "section" to a lifted value via the `bind` operator `>>=`

```haskell
# definition

class Monad m where
    return :: a -> m a
    # bind operator
    (>>=) :: m a -> (a -> m b) -> m b

    (>>) :: m a -> m b -> m b
    x >> y = x >>= \_ -> y

    fail :: String -> m a
    fail msg = error msg

# examples

instance Monad Maybe where
    return x = Just x
    Nothing >>= f = Nothing
    Just x >>= f  = f x
    fail _ = Nothing

instance Monad [] where
    return x = [x]
    xs >>= f = concat (map f xs)
    fail _ = []
```

Note that the `do notation` faciliates the use of `monads` when using nested bind operations:

```haskell
type Pole = (Int, Int)

routine :: Maybe Pole
routine = do
    start <- return (0,0)
    first <- landLeft 2 start
    Nothing # same as _ <- Nothing
    second <- landRight 2 first
    landLeft 1 second

justH :: Maybe Char
justH = do
    (x:xs) <- Just "hello"
    return x # alternatively use Just x (without return)
```

### State Monad

The `state monad` resides in the package `Control.Monad.State` and is defined as

```haskell
newtypeStatesa=State{runState::s->(a,s)}
```

It is useful for stateful objects like stacks, random generators that are not allowed to be manipulated by Haskell due to its pureness. When calling a function that also manipulates the state (like pop), one has to return the result *and* the current state (a new object). This quickly becomes tedious, so one wraps it into State Monads:

```haskell
instanceMonad(States)where
returnx=State$\s->(x,s)
(Stateh)>>=f=State$\s->let(a,newState)=hs
    (Stateg)=fa
ingnewState
```

### MonadPlus

A MonadPlus simply a monad that can also act as an monoid:

```haskell
# definition

class Monad m => MonadPlus m where
    # mzero is synonymous to mempty from the Monoid type class
    mzero :: m a
    # mplus corresponds to mappend from the Monoid type class
    mplus :: m a -> m a -> m a

# example

instance MonadPlus [] where
    mzero = []
    mplus = (++)
```

## Space leaks

Space leaks are code snippets that cause haskell to run into an oom like error, using much more memory than it should be. See [SO answer](https://stackoverflow.com/questions/46007746/whats-a-space-leak) for the following excellent explanation:

>As noted in @Rasko's answer, a space leak refers to a situation where a program or specific computation uses more (usually much more) memory than is necessary for the computation and/or expected by the programmer.
>
>Haskell programs tend to be particularly susceptible to space leaks, mostly because of the lazy evaluation model (sometimes complicated by the way IO interacts with this model) and the highly abstract nature of the language which can make it difficult for a programmer to determine exactly how a particular computation is likely to be performed.
>
>It helps to consider a specific example. This Haskell program:
>
>```haskell
>main = print $ sum [1..1000000000]
>```
>
>is an idiomatic way to sum the first billion integers. Compiled with `-O2`, it runs in a few seconds in constant memory (a few megabytes, basically the runtime overhead).
>
>Now, any programmer would expect a program to sum the first billion integers should run without chewing up memory, but it's actually a little surprising that this Haskell version is well behaved. After all, read literally, it constructs a list of a billion integers before summing them up, so it ought to require at least a few gigabytes (just for storage for the billion integers, not to mention the overhead of a Haskell linked list).
>
>However, lazy evaluation ensures that the list is only generated *as it's needed* and -- equally importantly -- optimizations performed by the compiler ensure that as list elements are added to the accumulating sum, the program recognizes they are no longer needed and allows them to be garbage collected instead of keeping them around until the end of the computation. So, at any point during the computation, only a sliding "window" into the middle of the list needs to be kept in memory -- earlier elements have been discarded, and later elements are yet to be lazily computed. (In fact, the optimizations go further than this: no list is even constructed, but this is far from obvious to the programmer.)
>
>Soooo... Haskell programmers get used to the idea that tossing around giant (or even infinite) data structures will "just work" with computations automatically using only the memory they need.
>
>But, a minor change to the program, like also printing the length of the list as proof of all the hard work we are doing:
>
>```haskell
>main = let vals = [1..1000000000]
>       in print (sum vals, length vals)
>```
>
>suddenly causes space usage to explode to dozens of gigabytes (or in the case of my laptop, to about 13Gigs before it starts swapping hopelessly and I kill it).
>
>This is a space leak. Calculating the sum and length of this list are obviously things that can be done in constant space using a "sliding window" view into the list, but the above program uses much more memory than needed. The reason, it turns out, is that once the list has been given a name `vals` that's used in two places, the compiler no longer allows the "used" elements to be immediately discarded. If the `sum vals` is evaluated first, the list is lazily generated and summed, but the entire, giant list is then kept around until `length vals` can be evaluated.
>
>As a more practical example, you might write a simple program to count words and characters in a file:
>
>```haskell
>main = do txt <- getContents
>          print (length txt, length (words txt))
>```
>
>This works fine on small test files up to a couple megabytes, but it's noticeably sluggish on 10meg file, and if you try to run it on a 100meg file, it'll slowly but surely start gobbling up all available memory. Again, the problem is that -- even though the file contents are read lazily into `txt` -- because `txt` is used twice, the entire contents are read into memory as a Haskell `String` type (a memory-inefficient representation of large blocks of text) when, say, `length txt` is evaluated, and none of that memory can be freed until `length (words txt)` has also been computed.
>
>Note that:
>
>```haskell
>main = do txt <- getContents
>          print $ length txt
>```
>
>and:
>
>```haskell
>main = do txt <- getContents
>          print $ length (words txt)
>```
>
>both run quickly in constant space even on big files.
>
>As a side note, fixing the above space leak normally involves rewriting the computation so the characters and words are counted with one pass through the contents, so the compiler can determine that the contents of the file that have already been processed do not need to be kept around in memory until the end of the computation. One possible solution is:
>
>```haskell
>{-# LANGUAGE BangPatterns #-}
>
>import Data.List
>import Data.Char
>
>charsWords :: String -> (Int, Int)
>charsWords str = let (_, chrs, wrds) = foldl' step (False, 0, 0) str
>                 in (chrs, wrds)
>  where step (inWord, cs, ws) c =
>          let !cs' = succ cs
>              !ws' = if not inWord && inWord' then succ ws else ws
>              !inWord' = not (isSpace c)
>          in (inWord', cs', ws')
>
>main = do txt <- getContents
>          print $ charsWords txt
>```
>
>The complexity of this solution (use of bang (`!`) patterns and an explicit fold instead of `length` and `words`) illustrates how tough space leaks can be, especially for new Haskell programmers. And it's not at all obvious that using `foldl'` instead of `foldl` makes no difference (but using `foldr` or `foldr'` would be a disaster!), that the bangs before `cs'` and `ws'` are critical to avoid a space leak, but that the bang before `inWord'` isn't (though it slightly improves performance), etc.

Also see [Avoiding space leaks at all costs](https://kodimensional.dev/space-leak)

## Testing

### Property based testing

- [quickcheck](https://hackage.haskell.org/package/QuickCheck) (the mother of pbt, generates random tests)
- [smallcheck](https://github.com/Bodigrim/smallcheck) (tries to capture all finitely many cases up to a certain depth)
- [quickstrom](https://github.com/quickstrom/quickstrom) test the dom of the browser
