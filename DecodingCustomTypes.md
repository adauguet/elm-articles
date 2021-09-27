# Decoding JSON into custom types

How should we decode JSON into custom Elm types?

## Directions

Here is a custom type we will be working with:
```elm
type CardinalPoint
    = North
    | West
    | East
    | South
```
Let's say we have a JSON structure containing some cardinal point at some point:
```json
{ ...
  "cardinal_point" : "west",
  ...
}
```

There is a particular function in the `elm/json` package designed for that:
```elm
andThen : (a -> Decoder b) -> Decoder a -> Decoder b
```
`andThen` takes a `Decoder a` and gives you a chance to decide what to do with the decoded value if it succeeds. Should you accept it? Or make your decoder fail? It is up to you!

In our case, the cardinal point is a string in the JSON, so we could start by:
```elm
import Json.Decode as D exposing (Decoder)

decoder : Decoder CardinalPoint
decoder =
    D.field "cardinal_point" D.string
        |> D.andThen ( {- and then what ? -} )
```

Then we need to write a function with the following signature:
```elm
a -> Decoder b
```
Or more precisely in our example:
```elm
String -> Decoder CardinalPoint
```

Here is our chance to examine the decoded string, and success or fail depending on its value. Too functions comes handy:
1. [`Json.Decode.succeed : a -> Decoder a`](https://package.elm-lang.org/packages/elm/json/latest/Json-Decode#succeed): produce a certain value.
2. [`Json.Decode.fail : String -> Decoder a`](https://package.elm-lang.org/packages/elm/json/latest/Json-Decode#fail): make the decoder fail.

Enough chatting, here it goes:
```elm
import Json.Decode as D exposing (Decoder)

decoder : Decoder CardinalPoint
decoder =
    D.field "cardinal_point" D.string
        |> D.andThen
            (\string ->
                case string of
                    "north" ->
                        D.succeed North

                    "west" ->
                        D.succeed West

                    "east" ->
                        D.succeed East

                    "south" ->
                        D.succeed South

                    invalidString ->
                        D.fail ("could not decode invalid cardinal point: " ++ invalidString)
            )
```

Note: you could also extract the anonymous function in a separate function.

Note: if parsing `cardinal_point` to `String` fails, the decoder fails before having `andThen` called.

## Conclusion

`andThen` is particularly helpful when decoding custom types. It allows you to implement a custom decoding strategy.

Happy Elm coding!