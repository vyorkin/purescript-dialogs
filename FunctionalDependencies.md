**ron wilson**:  
What exactly does the `|` do in type class definitions?
I looked here but it doesn't seem to discuss that: https://github.com/purescript/documentation/blob/master/language/Type-Classes.md
An example of what I mean:

```purescript
class Newtype t a | t -> a where
  wrap :: a -> t
  unwrap :: t -> a
```

**garyb**:  
Those are functional dependencies. Basically it's a way of describing relationships between type variables in a multi-parameter type class.
So here we're saying that the type `t` determines the type `a` in an instance.  
That also restricts the instances you can define, since you can't have `Newtype X Int` and `Newtype X Boolean` now (since `X` determines the second type parameter - so it can't be both of those things.

**cvlad**:  
Another way to think about these are, say you have a type class `TC` with two type parameters, `a` and `b`.  
If you have functions that only take `a` or `b`, then you wouldn't have enough information to know which `TC` to pick.  
For example,

```
class TC a b where
  getA :: a
```

However, if you say `class TC a b | a -> b`, it means that there's a single `b` for each `a`, so you will always be able to find the unique `getA`.

***
