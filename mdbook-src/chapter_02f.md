# Escaping characters

Now that `Html` has its own source file and module, and creating
html code can be done only via the functions we exported,
we can also handle user input that may contain characters we
that may conflict with our meta language html,
such as `<` and `>` which are used for creating HTML tags.

We can convert these characters into different strings that HTML can handle.

See [https://stackoverflow.com/questions/7381974/which-characters-need-to-be-escaped-in-html](https://stackoverflow.com/questions/7381974/which-characters-need-to-be-escaped-in-html)
for a list of characters we need to escape.

Let's create a new function called `escape`:

```hs
escape :: String -> String
escape =
  let
    escapeChar c =
      case c of
        '<' -> "&lt;"
        '>' -> "&gt;"
        '&' -> "&amp;"
        '"' -> "&quot;"
        '\'' -> "&#39;"
        _ -> [c]
  in
    concat . map escapeChar
```

In `escape` we see a few new things:

1. let expressions - we can define local names using this syntax:

```hs
let
  <name> = <expression>
in
  <expression>
```

This will make <name> available as a variable in the second <expression>.

2. Pattern matching with multiple patterns - we match on different
   characters and convert them to a string. Note that `_` is a "catch
   all" pattern that will always succeed.

3. Two new functions: `map` and `concat`, we'll talk about these more in depth

## Linked lists briefly

Linked lists are a very common data structure in Haskell, so common that
they have their own special syntax:

1. The type for lists are denoted with brackets and inside them is the type of the element. For example:
   - `[Int]` - a list of integers
   - `[Char]` - a list of characters
   - `[String]` - a list of strings
   - `[[String]]` - a list of a list of strings
2. An expression representing a empty list is written like this: `[]`
3. Prepending an element to a list is done with the operator `:` (pronounced cons) which is right-associative (like `->`).
   For example: `1 : []`, or `1 : 2 : 3 : []`.
4. The above lists can also be written like this: `[1]` and `[1, 2, 3]`.

Also, Strings are linked lists of chararacters - String is defined as:
`type String = [Char]`, so we can use them the same way we use lists.

---

Do note, however, that linked lists, despite their convenience, are often
not the right tool for the job. They are not particularily space efficient
and are slow for appending, random access and more. That also makes `String`
a lot less efficient than it could be. And I generally recommend using a
different string type, `Text`, instead, which is available in an external package.
We will talk about lists, `Text`, and other data structures in the future!

---

We can implement our own operations on lists by using pattern matching and recursion.
And we'll touch on this subject later when talking about ADTs.

For now, we will use the various functions found in the [Data.List](https://hackage.haskell.org/package/base-4.15.0.0/docs/Data-List.html) module. Specifically, [map](https://hackage.haskell.org/package/base-4.15.0.0/docs/Data-List.html#v:map) and [concat](https://hackage.haskell.org/package/base-4.15.0.0/docs/Data-List.html#v:concat).

`map` applying a function to each of the elements in a list. Its type signature is:

```hs
map :: (a -> b) -> [a] -> [b]
```

For example:

```hs
map not [False, True, False] == [True, False, True]
```

Or as can be seen in our `escape` function, this can help us escape each character:

```hs
map escapeChar ['<','h','1','>'] == ["&lt;","h","1","&gt;"]
```

However, note that the `escapeChar` has the type `Char -> String`,
so the result type of `map escapeChar ['<','h','1','>']` is `[String]`,
and what we really want is a `String` and not `[String]`.

This is where `concat` enters the picture. `concat` has the type

```hs
concat :: [[a]] -> [a]
```

It flattens a list of list of something into a list of something.
In our case in will flatten `[String]` into `String`, remember that this works
because `String` is a **type alias** for `[Char]`, so we actually have
`[[Char]] -> [Char]`.

---

## Escaping

The user of our library can currently only supply strings in a few places:

1. Page title
2. Paragraphs
3. Headers

We can apply our escape function at these places before doing anything else with it.
That way all html constructions are safe.

Try adding the escaping function in those places.

<details>
  <summary>Solution</summary>

```hs
html_ :: HtmlTitle -> HtmlBodyContent -> Html
html_ title content =
  Html
    ( el "html"
      ( el "head" (el "title" (escape title))
        <> el "body" (getBodyContentString content)
      )
    )

p_ :: String -> HtmlBodyContent
p_ = HtmlBodyContent . el "p" . escape

h1_ :: String -> HtmlBodyContent
h1_ = HtmlBodyContent . el "h1" . escape
```

</details>

---

<details>
  <summary><b>Our revised Html.hs</b></summary>

```hs
-- Html.hs

module Html
  ( Html
  , HtmlTitle
  , HtmlBodyContent
  , html_
  , p_
  , h1_
  , append_
  , render
  )
where

-- * Types

newtype Html
  = Html String

newtype HtmlBodyContent
  = HtmlBodyContent String

type HtmlTitle
  = String

-- * EDSL

html_ :: HtmlTitle -> HtmlBodyContent -> Html
html_ title content =
  Html
    ( el "html"
      ( el "head" (el "title" (escape title))
        <> el "body" (getBodyContentString content)
      )
    )

p_ :: String -> HtmlBodyContent
p_ = HtmlBodyContent . el "p" . escape

h1_ :: String -> HtmlBodyContent
h1_ = HtmlBodyContent . el "h1" . escape

append_ :: HtmlBodyContent -> HtmlBodyContent -> HtmlBodyContent
append_ c1 c2 =
  HtmlBodyContent (getBodyContentString c1 <> getBodyContentString c2)

-- * Render

render :: Html -> String
render html =
  case html of
    Html str -> str

-- * Utilities

el :: String -> String -> String
el tag content =
  "<" <> tag <> ">" <> content <> "</" <> tag <> ">"

getBodyContentString :: HtmlBodyContent -> String
getBodyContentString content =
  case content of
    HtmlBodyContent str -> str

escape :: String -> String
escape =
  let
    escapeChar c =
      case c of
        '<' -> "&lt;"
        '>' -> "&gt;"
        '&' -> "&amp;"
        '"' -> "&quot;"
        '\'' -> "&#39;"
        _ -> [c]
  in
    concat . map escapeChar
```

</details>

Trying constructing an invalid html in `hello.hs` to see if this works or not!

Now we can use our tiny html library safely. But what if the user
wants to use our library with something we didn't think about, for
example adding unordered lists? We are completely blocking them from
extending our library. We'll talk about this next.