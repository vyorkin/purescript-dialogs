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

Some trivial examples of functional dependencies from the [Fun with functional dependencies](http://www.cse.chalmers.se/%7Ehallgren/Papers/hallgren.pdf) paper:

* example 1: http://try.purescript.org/?gist=b93cd253c65bea68e703bf85d214ab49
* example 2: http://try.purescript.org/?gist=a36a1519a9ca45cb8394282e87b7918c
* example 3: http://try.purescript.org/?gist=4dd60313ac753c4c9623e1ae5b712b35
* example 4: http://try.purescript.org/?gist=e9b85b4ba021704321d8fa8fbd351587

***
