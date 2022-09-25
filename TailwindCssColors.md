# Import Tailwind CSS colors in Elm

Tailwind CSS is a great CSS framework. It includes a great color palette that I like to use in my projects. What I typically do is just copy/paste the hex color codes I am interested in into my Elm code.

But colors are defined in hex codes, and I am using `elm-ui` which only supports rgb values. Also, what if I could import all of those colors in my projects as a single reusable file?

## Tailwind CSS colors data

Here is what we get when copy pasting the html table from Tailwind's [website](https://tailwindcss.com/docs/customizing-colors):

```
Slate
50
#f8fafc
100
#f1f5f9
200
#e2e8f0
300
#cbd5e1
400
#94a3b8
500
#64748b
600
#475569
700
#334155
800
#1e293b
900
#0f172a
Gray
50
#f9fafb
100
#f3f4f6
200
#e5e7eb
300
#d1d5db
400
#9ca3af
500
#6b7280
600
#4b5563
700
#374151
800
#1f2937
900
#111827
...
```

Yes. We could totally spend some time working this data with multicursors or some magical macro tricks from your favorite editor. Totally agree. But please bear with me.

## `elm-parser`

Each color appears as a block starting by the color name and followed by its ten shades. Each shade starts by its index and ends with an hex color code.

To parse this data, we are going to use the `elm-parser` library, which is specifically designed for this kind of task.

I you worked with `elm/json` before, it works in a similar way. All we need is to describe the shape of our data so that it can be parsed correctly.

Let's start by parsing a single color shade.
```
50
#f8fafc
```

We define a `Shade` record to hold the data, and its corresponding parser.
```elm
import Parser

type alias Shade =
    { index : Int
    , color : String
    }

parser : Parser Shade
parser =
    Parser.succeed
        |= Parser.int
        |. Parser.spaces
        |= (Parser.chompWhile (\c -> c /= '\n') |> getChompedString)
```

This gives us something like:
```elm
{ index = 50, color = "#f8fafc" }
```

But it is not good enough. What would be great is to have red, green and blue components instead.

## `elm-codegen`