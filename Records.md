**islon**:
I have two record types:

```purescript
type GetStuff r = {getStuff :: Int -> String | r}
type GetAnother r = {getAnother :: String -> Int | r}
```
I would like to get a new record type that joins them both: `type Context = ???` is it possible?

**monoidmusician**:
this is probably the least-hassle pattern for doing that:

```purescript
-- create a synonym for just the row
type GetStuffRow r = ( getStuff :: Int -> String | r )

-- and one for the record will be helpful too ;)
type GetStuff r = Record (GetStuffRow r)

-- rinse and repeat
type GetAnotherRow r = ( getAnother :: String -> Int | r )
type GetAnother r = Record (GetAnotherRow r)

-- and now chain them together
type GetBothRow r = GetStuffRow (GetAnotherRow r)
type GetBoth r = Record (GetBothRow r)
```
(not saying it’s elegant! :wink:)
[this is](https://github.com/purescript/purescript-typelevel-prelude/commit/27ca2226f6b4ef4037ce37a4b5ac2ca3ceb2adf1) Nate’s way to make it seem a little cleaner (but it isn’t in a library yet)

***
