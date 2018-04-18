**drathier**:
where do I put the `forall` and what difference does it make?
thought I could only put them at the start of a signature, but I just stumbled into one that used a forall in the last type

**cvlad**:
They need to be parenthesized if they appear somewhere else, and sometimes they're equivalent to putting them at the beginning. For example, I think `mkTuple :: forall a b. a -> b -> Tuple a b` is equivalent to `mkTuple :: forall a. a -> (forall b. b -> Tuple a b)`.
Not sure of a simple example where they're not similar. If you're familiar with `Yoneda`, that's one example.

**cvlad**:
But let's say you have a function like `f :: forall a. (Int -> a) -> Int -> a`; then `f int2a a = int2a a` is the only possible implementation because the `a`s must match. However, if we change that to `f :: forall a. (forall b. b -> a) -> Int -> a` then we can implement it like `f a2b a = a2b a` of `f a2b a = a2b (a2b a)` etc.
That's because in the context of the second `f`, `a2b` is polymorphic in `b`, so it can take both `Int`s and `a`s.
Whereas in the first, it's not polymorphic at all since `a` is fixed by `f`s return type.

**natefaubion**:
I think the best intuition for `forall` is that they are just arguments for types (which the compiler can fill in). Therefore a function with the type `identity :: forall a. a -> a` actually takes _2_ arguments, it’s just that one of them is a type which specializes the function. So `identity` is a function that when given a Type argument, return a function specialized to the provided type. When you place the forall in different positions you are requiring some contract wrt polymorphism. A function like `rank2Identity :: forall a. (forall b. b -> b) -> a -> a` has a rank-2 forall, which means that it demands you give it a function that is itself polymorphic. (edited)

**dbaynard**:
@drathier To add to what @cvlad has said (and yes, simple examples don't come to mind for me either), try this.

If the forall is used as in `mkTuple` you as the person might call `mkTuple (string :: String) (int :: Int)` and will end up with a value of type `Tuple String Int`. Or perhaps you call `mkTuple (a :: A) (b :: B)` for a `Tuple A B` where `A` and `B` are concrete types.

The types you supply determine the output type, and whoever wrote `mkTuple` wrote it in a way that it should work no matter what types you supply.

By contrast, how about `reify :: forall a . a -> (forall b . b -> Tuple a b) -> a`? Pick an a — maybe `String` — and partially apply it for `reify "a string" :: (forall b . b -> Tuple String b) -> String`. This is a function that takes as its first parameter a function that takes any `b` and makes a tuple. Well, your `mkTuple` function can do that! `mkTuple "another string" :: forall b . b -> Tuple String b`.

But how about `b`? Well, the `forall` in the type of the next argument of `reify "a string" :: (forall b . b -> Tuple String b) -> String` means you need to supply a function that is itself valid for any input type. So `forall b . b -> Tuple String b` is OK but `Int -> Tuple String Int` is not. You no longer get to determine the type of the variable `b`.

If you wish to try it out, try to guess the result. An implementation of `reify` is below.

```
import Data.Tuple as T
reify = (flip (\f -> T.fst <<< f) :: forall a . a -> (forall b . b -> T.Tuple a b) -> a)

```

This is where @cvlad's second point comes in — this is (I believe) one of two implementations of reify (try to find the other one!).

Another consequence is that once you have supplied this function, the resulting value no is longer parameterized by the type `b`. The `b` has been hidden. This is often the main reason for using this sort of formulation — perhaps the b represents some resource which shouldn't be allowed to leak.

One common example (hence the name I chose) is `reifySymbol` from Data.Symbol.

```reifySymbol :: forall r. String -> (forall sym. IsSymbol sym => SProxy sym -> r) -> r```

So… you pick the type `r`. How about `Int`? And then supply a string:

```reifySymbol "this is a string" :: (forall sym. IsSymbol sym => SProxy sym -> Int) -> Int```

And you now have a function which itself takes a function `f` of a type that represents any string and returns an integer, and applies it to your string. As this function `f` has the `IsSymbol` constraint it can do more useful things than the unconstrained versions, without leaking the `SProxy`.

***
