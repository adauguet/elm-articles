# Decoding JSON into Elm Opaque types

JSON decoders are one of the most frequent subject in Elm Slack's `#beginners` channel. Here is one of the question I saw recently.

## The `Cred` type alias

Let's say we want to parse the following JSON in a record.
```json
{ "username" : "...",
  "token" : "..."
}
```

We would write a record and assign it to a `type alias Cred`:
```elm
type alias Cred =
    { username : String
    , token : String
    }
```

Using `elm/json` package, we write the corresponding JSON decoder:
```elm
import Json.Decode as D exposing (Decoder)

decoder : Decoder Cred
decoder =
    D.map2 Cred
        (D.field "username" D.string)
        (D.field "token" D.string)
```
Pretty straightforward, right?

When we write the `type alias Cred`, we automatically get the following constructor for free, which is used in the current decoder:
```elm
Cred : String -> String -> Cred
Cred username token =
    { username = username
    , token : token
    }
```

Note: we could also use `NoRedInk/elm-json-decode-pipelines` to write our decoder this way:
```elm
import Json.Decode as D exposing (Decoder)
import Json.Decode.Pipeline as DP

decoder : Decoder Cred
decoder =
    D.succeed Cred
        |> DP.required "username" D.string
        |> DP.required "token" D.string
```
This would result in the exact same behavior.

## Opaque type

Now we want to add an extra layer of security by hiding the internal implementation details of `Cred` from the outside of the module. To do that we use an opaque type. Our `Cred` type now looks like this:

```elm
type Cred
    = Cred
        { username : String
        , token : String
        }
```
Note that we replace the `type alias` by a `type` with a unique `Cred` variant. Consequence: **the free constructor we previously had is no longer available**.

Also note the difference between the type `Cred` and our type constructor `Cred`. Elm supports them to have the same name, but it can be easier to name them differently. We will just rename the type constructor to `CredConstructor` for the rest of the example.
```elm
type Cred
    = CredConstructor
        { username : String
        , token : String
        }
```

By exposing only the type and not the constructor, it is now impossible to create a `Cred` manually. You have to use our dedicated functions, like our decoder.
```elm
module Cred exposing (Cred)
```

## Opaque type decoder

So how should we adapt our decoder?
```elm
import Json.Decode as D exposing (Decoder)

decoder : Decoder Cred
decoder =
    -- what to write here?
        (D.field "username" D.string)
        (D.field "token" D.string)
```

Let's look at the signature of `Json.Decode.map2`:
```elm
map2 : (a -> b -> value) -> (Decoder a) -> (Decoder b) -> (Decoder value)
```

In our case, `a` and `b` are `String`, `value` is `Cred`. In our case, the signature actually is:
```elm
map2 : (String -> String -> Cred) -> (Decoder String) -> (Decoder String) -> (Decoder Cred)
```

So all we need now is to implement the following function:
```elm
makeCred : String -> String -> Cred
```

There is no difficulty here:
```elm
makeCred : String -> String -> Cred
makeCred username token =
    CredConstructor
        { username = username
        , token = token
        }
```
Note that we use the `CredConstructor` unique type constructor, which signature is:
```elm
CredConstructor : { username : String, token : String } -> Cred
```

We can now write our decoder:
```elm
import Json.Decode as D exposing (Decoder)

decoder : Decoder Cred
decoder =
    D.map2 makeCred
        (D.field "username" D.string)
        (D.field "token" D.string)
```
Or group everything in a single function:
```elm
import Json.Decode as D exposing (Decoder)

decoder : Decoder Cred
decoder =
    D.map2
        (\username token ->
            CredConstructor
                { username = username
                , token = token
                }
        )
        (D.field "username" D.string)
        (D.field "token" D.string)
```

Which could also be written with `NoRedInk/elm-json-decode-pipelines`:
```elm
import Json.Decode as D exposing (Decoder)
import Json.Decode.Pipeline as DP

decoder : Decoder Cred
decoder =
    D.succeed
        (\username token ->
            CredConstructor
                { username = username
                , token = token
                }
        )
        |> DP.required "username" D.string
        |> DP.required "token" D.string
```

## Refactoring

There are two problems with the current solution:
1. Things can quickly get messy with a bigger record.
2. We have to write the record constructor ourselves ... which can be avoided.

To fix those problems, we can extract the record in its own `type alias`:
```elm
type Cred = Cred CredData

type alias CredData =
    { username : String
    , token : String
    }
```

This way, we get our free constructor back! And we can immediatly use it when decoding:

Using `elm/json`:
```elm
dataDecoder : Decoder CredData
dataDecoder =
    D.map2 CredData
        (D.field "username" D.string)
        (D.field "token" D.string)
```

Using `NoRedInk/elm-json-decode-pipelines`:
```elm
dataDecoder : Decoder CredData
dataDecoder =
    D.succeed CredData
        |> DP.required "username" D.string
        |> DP.required "token" D.string
```

But wait, we are not decoding the `Cred` type yet! In fact it really looks like we are back to square one.

Yet we are close to the solution. This is were `Json.Decode.map` comes handy:
```elm
decoder : Decoder Cred
decoder =
    D.map Cred dataDecoder
```

Same thing in a single function:
```elm
decoder : Decoder Cred
decoder =
    D.map2 CredData
        (D.field "username" D.string)
        (D.field "token" D.string)
        |> D.map Cred
```

Using `NoRedInk/elm-json-decode-pipelines`:
```elm
decoder : Decoder Cred
decoder =
    D.succeed CredData
        |> DP.required "username" D.string
        |> DP.required "token" D.string
        |> D.map Cred
```

Extracting `CredData` allows us to write a simple record decoder and wrapping it with `Json.Decoder.map` to decode the full opaque type. As you can see the final solution is quite simple.

## Conclusion

With this example I hope you now have a better understanding of decoding JSON into opaque types in Elm.

JSON decoders do not have to be difficult! 😉

Happy Elm coding!

## Credits

Original question from `Stanley`. Thanks to `@sebbes` for re-reading and relevant suggestions.