---
layout: post
title: Modules
---

Modules
=======

Loading modules
---------------

![modules](img/modules.png)

An Elm module is a collection of related functions and types.
An Elm program is a collection of modules where the main
module loads up the other modules and then uses the functions defined in
them to do something. Having code split up into several modules has
quite a lot of advantages. If a module is generic enough, the functions
it exports can be used in a multitude of different programs. If your own
code is separated into self-contained modules which don't rely on each
other too much (we also say they are loosely coupled), you can reuse
them later on. It makes the whole deal of writing code more manageable
by having it split into several parts, each of which has some sort of
purpose.

The Elm core library is split into modules, each of them
contains functions and types that are somehow related and serve some
common purpose. There's a module for manipulating lists, a module for
concurrent programming, a module for dealing with dates, etc.
All the functions and types that we've dealt with so far
were from modules which are imported by default. In this
chapter, we're going to examine a few useful modules and the functions
that they have. But first, we're going to see how to import modules.

The syntax for importing modules in an Elm script is `import <module
name>`, which will make the entire module accessible under the `module name`
namespace (e.g. if we import the `List` module, we can access everything exposed
by the module as long as you put `List.` before it... like `List.map`, `List.filter`,
etc.). These are called *qualified* imports. The functions you're importing are
qualified by the module name, like `Dict.map` or `String.toList`
You can also import specific functions from a module like this:
`import List exposing (map, filter)` or `import Dict exposing (..)`
(which exposes everything in the `Dict` module). These are *unqualified* imports.
This can make your code more concise, but since many modules export functions
with identical names (e.g. `map` and `filter`), you may need to take
care not to import multiple identically named functions into the same
module (but don't worry too much, if you do the compiler will let you know),
Importing must be done before defining any functions, so imports are
usually done at the top of the file. One script can, of course, import
several modules. Just put each import statement into a separate line.
Let's import the `List` module, which has a bunch of useful functions
for working with lists and use a function that it exports to create a
function that tells us how many odd elements a list has. We'll also import
the `Tuple` module to work with the results produced by `List.partition`.
`List` and `Tuple` are imported qualified by default, so the following is
intended just to give you an idea of how importing works. You wouldn't
necessarily need to do the following in real life.

```elm
import List exposing (length, partition)
import Tuple

numOdds : List Int -> Int
numOdds = 
    let
        odd n = n % 2 == 1
    in
    length << Tuple.first << partition odd
```

When you do `import Tuple`, all the functions that `Tuple` exports
become available under the `Tuple` namespace, meaning that you can call them
from wherever in the script. And when you do
`import List exposing (length, partition)`, you pull `length` and `partition`
into the module unqalified.
`partition` is a function defined in `List` that
takes a predicate and a list and produces a tuple of two lists. The first list
being those elements for which the predicate evaluated to `True`, and the
second list being those elements for which the predicate evaluated to
`False`. Composing `length`, `Tuple.first`, and `partition`
we produce a function that's the equivalent of `\xs -> length (Tuple.first (partition (\n -> n % 2 == 1) xs))`

We can also import modules using the `as` keyword to provide a shorter
name. For example,

```elm
import Json.Decode as Decode

Decode.string "JSON string"
```

This makes it so that if we want to reference `Json.Decode`'s `string`
function, we don't have to type out `Json.Decode.string` every time.

Use [this handy
reference](http://package.elm-lang.org/packages/elm-lang/core/5.1.1) to
see which modules are in the standard library. A great way to pick up
new Elm knowledge is to just click through the standard library
reference and explore the modules and their functions. You can also view
the Elm source code for each module. Reading the source code of some
modules is a really good way to learn Elm and get a solid feel for
it.


List
---------

The `List` module is all about lists, obviously. It provides some
very useful functions for dealing with them. We've already met some of
its functions (like `map` and `filter`) because the `List` module is
imported by default. Let's take a look at some of the functions that we haven't
met before.

`intersperse` takes an element and a list and then puts that element in
between each pair of elements in the list. Here's a demonstration:

```elm
> List.intersperse "on" ["turtles","turtles","turtles"]
["turtles","on","turtles","on","turtles"] : List String
> intersperse 0 [1,2,3,4,5,6]
[1,0,2,0,3,0,4,0,5,0,6] : List number
```

![shopping lists](img/legolists.png)

`concat` flattens a list of lists into just a list of elements.

```elm
> List.concat [["foo", "bar"],["baz","qux"]]
["foo","bar","baz","qux"] : List String
> List.concat [[3,4,5],[2,3,4],[2,1,1]]
[3,4,5,2,3,4,2,1,1] : List number
```

It will just remove one level of nesting. So if you want to completely
flatten `[[[2,3],[3,4,5],[2]],[[2,3],[3,4]]]`, which is a list of lists of
lists, you have to concatenate it twice.

Doing `concatMap` is the same as first mapping a function to a list and
then concatenating the list with concat.

```elm
> List.concatMap (List.repeat 4) [1,2,3]
[1,1,1,1,2,2,2,2,3,3,3,3] : List number
```

`any` and `all` take a predicate and then check if any or all the elements
in a list satisfy the predicate, respectively.

```elm
> List.any ((==) 4) [2,3,5,6,1,4]
True : Bool
> List.all ((>) 4) [6,9,10]
True : Bool
> List.all (\n -> n % 2 == 1) [1,2,3,4,5]
False : Bool
> List.any (\n -> n % 2 == 1) [1,2,3,4,5]
True : Bool
```

`sort` simply sorts a list. The type of the elements in the list has to be
part of the `comparable` typeclass, because if the elements of a list can't be
compared in some way, then the list can't be sorted.

```elm
> List.sort [8,5,3,2,1,6,4,2]
[1,2,2,3,4,5,6,8]
> List.sort ["foo", "bar", "baz", "qux"]
["bar","baz","foo","qux"] : List String
```

`member` checks if an element is inside a list.

`partition` takes a list and a predicate and returns a pair of lists. The
first list in the result contains all the elements that satisfy the
predicate, the second contains all the ones that don't.

```elm
> List.partition (\n -> n % 2 == 1) [1,2,3,4,5,6]
([1,3,5],[2,4,6]) : ( List Int, List Int )
> List.partition (flip (>) 3) [1,3,5,6,3,2,1,0,3,7]
([5,6,7],[1,3,3,2,1,0,3]) : ( List number, List number )
```

What `length`, `take`, `drop`, `repeat` have in common is
that they take an Int as one of their parameters (or return an Int).
For instance, length has a type signature of `length : List a -> Int`. If we
try to get the average of a list of numbers by doing `let xs = List.range 1 6 in
List.sum xs / List.length xs`, we get a type error, because you can't use `/` with an
Int.

The `sort` function also has a more general equivalent.
`sortWith` take a function that determines if one element is greater,
smaller or equal to the other. The type signature of `sortWith` is `sortWith :
(a -> a -> Order) -> List a -> List a`. If you remember from before, the
Order type can have a value of LT, EQ or GT. `sort` is the equivalent
of `List.sortWith compare`, because `compare` just takes two elements whose type is
in the `comparable` typeclass and returns their ordering relationship.

Lists can be compared, but when they are, they are compared
lexicographically. What if we have a list of lists and we want to sort
it not based on the inner lists' contents but on their lengths? Well, as
you've probably guessed, we'll use the `sortWith` function. First we'll
define the `on` function, like this, to act as a helper: 

```elm
on : (b -> b -> c) -> (a -> b) -> a -> a -> c
on f g = \x y -> f (g x) (g y)
```

So doing `on (==) (> 0)` returns an equality function that looks like `\x y -> (x > 0) == (y > 0)`. `on` is useful with the *By/With* functions because with it, we can do:

```elm
> xs = [[5,4,5,4,4],[1,2,3],[3,5,4,3],[],[2],[2,2]]
[[5,4,5,4,4],[1,2,3],[3,5,4,3],[],[2],[2,2]] : List (List number)
> List.sortWith (on compare List.length) xs
[[],[2],[2,2],[1,2,3],[3,5,4,3],[5,4,5,4,4]]
```

Awesome! `on compare List.length`... man, that reads almost like real
English! If you're not sure how exactly the `on` works here, `on compare List.length`
is the equivalent of `\x y -> compare (List.length x) (List.length y)`.
When you're dealing with *By/With* functions that take an equality
function, you might do something like `on (==) something` and when you're dealing
with *By/With* functions that take an ordering function, you might do
`on compare something`.

Char
---------

![lego char](img/legochar.png)

The `Char` module does what its name suggests. It exports functions
that deal with characters.

`Char` exports a bunch of predicates over characters. That is,
functions that take a character and tell us whether some assumption
about it is true or false. Here's what they are:

`isUpper` checks whether a character is an upper case ASCII letter.

`isLower` checks whether a character is a lower case ASCII letter.

`isDigit` checks whether a character is an ASCII digit [0-9].

`isOctDigit` checks whether a character is an ASCII octal digit [0-7].

`isHexDigit` checks whether a character is an ASCII hexadecimal digit [0-9a-fA-F].

All these predicates have a type signature of `Char -> Bool`. Most of the
time you'll use this to filter out strings or something like that. For
instance, let's say we're making a program that takes a username and the
username can only be comprised of lowercase characters. We can use
the `List` function `all` and the `String` function `toList` in combination
with the `Char` predicates to determine if the username is alright.

```elm
> List.all Char.isLower <| String.toList "bobby"
True : Bool
> List.all Char.isLower <| String.toList "Bobbyå"
False : False
```

Kewl. In case you don't remember, `all` takes a predicate and a list and
returns `True` only if that predicate holds for every element in the list.

`toUpper` converts a character to upper-case. Spaces, numbers, and the
like remain unchanged.

`toLower` converts a character to lower-case.

`toCode` converts a character to a `KeyCode` (a type alias for `Int`).

```elm
> List.map Char.toCode <| String.toList "34538"
[51,52,53,51,56] : List Char.KeyCode
> List.map Char.toCode <| String.toList "FF85AB"
[70,70,56,53,65,66] : List Char.KeyCode
```

`fromCode` is the inverse function of `toCode`. It takes a `KeyCode`
and converts it to a character.

```elm
> Char.fromCode 70
'F' : Char
> Char.toCode 51
'3'
```

The Caesar cipher is a primitive method of encoding messages by shifting
each character in them by a fixed number of positions in the alphabet.
We can easily create a sort of Caesar cipher of our own, only we won't
constrict ourselves to the alphabet.

```elm
encode : Int -> String -> String
encode shift msg =
    let
        ords = List.map Char.toCode <| String.toList msg
        shifted = List.map ((+) shift) ords
    in
    String.fromList <| List.map Char.fromCode shifted
```

Here, we first convert the string to a list of `Char`, then map
it to a list of `KeyCode`'s (which, again, are just a type alias to `Int`).
Then we add the
shift amount to each number before converting the list of numbers back
to characters, then back to a string. If you're a composition cowboy,
you could write the body of this function as
`String.fromList <| List.map (Char.fromCode << ((+) shift) << Char.toCode)
<| String.toList msg`.
Let's try encoding a few messages.

```elm
> encode 3 "Heeeeey"
"Khhhhh|" : String
> encode 4 "Heeeeey"
"Liiiii}" : String
> encode 1 "abcd"
"bcde" : String
> encode 5 "Marry Christmas! Ho ho ho!"
"Rfww~%Hmwnxyrfx&%Mt%mt%mt&" : String
```

That's encoded alright. Decoding a message is basically just shifting it
back by the number of places it was shifted by in the first place.

```elm
decode : Int -> String -> String
decode shift msg = encode (negate shift) msg
```

```elm
> encode 3 "Im a little teapot"
"Lp#d#olwwoh#whdsrw"
> decode 3 "Lp#d#olwwoh#whdsrw"
"Im a little teapot"
> decode 5 << encode 5 <| "This is a sentence"
"This is a sentence"
```

Dict
--------

Association lists (also called dictionaries) are lists that are used to
store key-value pairs where ordering doesn't matter. For instance, we
might use an association list to store phone numbers, where phone
numbers would be the values and people's names would be the keys. We
don't care in which order they're stored, we just want to get the right
phone number for the right person.

The most obvious way to represent association lists in Elm would be
by having a list of pairs. The first component in the pair would be the
key, the second component the value. Here's an example of an association
list with phone numbers:

```elm
phoneBook =
    [("betty","555-2938")
    ,("bonnie","452-2928")
    ,("patsy","493-2928")
    ,("lucille","205-2928")
    ,("wendy","939-8282")
    ,("penny","853-2492")
    ]
```

Despite this seemingly odd indentation, this is just a list of pairs of
strings. The most common task when dealing with association lists is
looking up some value by key. Let's make a function that looks up some
value given a key.

```elm
findKey : k -> List (k,v) -> Maybe v
findKey key xs = Maybe.map Tuple.second << List.head << List.filter (\(k,v) -> key == k) <| xs
```

Pretty simple. The function that takes a key and a list, filters the
list so that only matching keys remain, gets the first key-value that
matches and returns the value. But what happens if the key we're looking
for isn't in the association list? Hmm. Well, our old friend `Maybe` is
here to help. If we don't find the key, we'll return a `Nothing`.
If we find it, we'll return `Just something`, where `something` is the value
corresponding to that key.

This is a textbook recursive function that operates on a list. Edge
case, splitting a list into a head and a tail, recursive calls, they're
all there. This is the classic fold pattern, so let's see how this would
be implemented as a fold.

```elm
findKey : k -> List (k,v) -> Maybe v
findKey key = List.foldr (\(k,v) acc -> if key == k then Just v else acc) Nothing
```

*Note:* It's usually better to use folds for this standard list
recursion pattern instead of explicitly writing the recursion because
they're easier to read and identify. Everyone knows it's a fold when
they see the `foldr` call, but it takes some more thinking to read
explicit recursion.

```elm
> findKey "penny" phoneBook
Just "853-2492" : Maybe.Maybe String
> findKey "betty" phoneBook
Just "555-2938" : Maybe.Maybe String
> findKey "wilma" phoneBook
Nothing : Maybe.Maybe String
```

![legomap](img/legomap.png)

Works like a charm! If we have the girl's phone number, we `Just` get the
number, otherwise we get `Nothing`.

If we want to find the corresponding value to a key, we have to traverse all the
elements of the list until we find it. The `Dict` module offers
association lists that are much faster (because they're internally
implemented with trees) and also it provides a lot of utility functions.
From now on, we'll say we're working with dictionaries instead of association
lists.

Let's go ahead and see what `Dict` has in store for us! Here's the
basic rundown of its functions.

The `fromList` function takes an association list (in the form of a list)
and returns a dictionary with the same associations.

```elm
> Dict.fromList [("betty","555-2938"),("bonnie","452-2928"),("lucille","205-2928")]
Dict.fromList [("betty","555-2938"),("bonnie","452-2928"),("lucille","205-2928")] : Dict.Dict String String
> Dict.fromList [(1,2),(3,4),(3,2),(5,5)]
Dict.fromList [(1,2),(3,2),(5,5)] : Dict.Dict number number1
```

If there are duplicate keys in the original association list, the
duplicates are just discarded. This is the type signature of `fromList`

```elm
Dict.fromList : List (comparable, v) -> Dict.Dict comparable v
```

It says that it takes a list of pairs of type `comparable` and `v` and returns a
dictionary that maps from keys of type `comparable` to type `v`.
Notice here that the keys have to be comparable. That's an essential
constraint in the `Dict` module. It
needs the keys to be comparable so it can arrange them in a tree.

You should always use `Dict` for key-value associations unless you
have keys that aren't part of the `comparable` typeclass.

Let's back up for a moment. Did notice anything unusual with our
dictionary's type signature? `Dict.Dict number number1`. We've seen
`number` before, so what is `number1`? Well, `number` in Elm is a
typeclass consisting of either `Int` or `Float`. So what the type
system is telling us is that we have a dictionary with keys of
type `number` and values also of type `number`, but we should not
assume that these types are equivalent! It could turn out that our
keys are `Int`'s, but our values are `Float`'s (or vice versa), so
the type system is helping us out here by keeping these two types
distinct. Alright, let's continue.

`empty` represents an empty dictionary. It takes no arguments, it just returns an
empty dictionary.

```elm
> Dict.empty
Dict.fromList [] : Dict.Dict k v
```

`insert` takes a key, a value and a dictionary and returns a new dictionary
that's just like the old one, only with the key and value inserted.

```elm
> Dict.empty
Dict.fromList [] : Dict.Dict k v
> Dict.insert 3 100 Dict.empty
Dict.fromList [(3,100)] : Dict.Dict number number1
> Dict.insert 5 600 (Dict.insert 4 200 ( Dict.insert 3 100  Dict.empty))
fromList [(3,100),(4,200),(5,600)] : Dict.Dict number number1
> Dict.insert 5 600 << Dict.insert 4 200 << Dict.insert 3 100 <| Dict.empty
fromList [(3,100),(4,200),(5,600)] : Dict.Dict number number1
```

We can implement our own `fromList` by using the empty dictionary, `insert` and a
`fold`. Watch:

```elm
fromList : List (comparable,v) -> Dict.Dict comparable v
fromList = List.foldr (\(k, v) acc -> Dict.insert k v acc) Dict.empty
```

It's a pretty straightforward fold. We start of with an empty dictionary and we
fold it up from the right, inserting the key value pairs into the
accumulator as we go along.

`isEmpty` checks if a dictionary is empty.

```elm
> Dict.isEmpty Dict.empty
True : Bool
> Dict.isEmpty <| Dict.fromList [(2,3),(5,5)]
False : Bool
```

`size` reports the size of a dictionary.

```elm
> Dict.size Dict.empty
0 : Int
> Dict.size <| Dict.fromList [(2,4),(3,3),(4,2),(5,4),(6,4)]
5 : Int
```

`singleton` takes a key and a value and creates a dictionary that has exactly one
mapping.

```elm
> Dict.singleton 3 9
fromList [(3,9)] : Dict.Dict number number1
> Dict.insert 5 9 <| Dict.singleton 3 9
fromList [(3,9),(5,9)] : Dict.Dict number number1
```

`get` returns `Just something` if it finds `something` for the key and `Nothing`
if it doesn't.

`member` is a predicate that takes a key and a dictionary and reports whether the key
is in the dictionary or not.

```elm
> Dict.member 3 <| Dict.fromList [(3,6),(4,3),(6,9)]
True : Bool
> Dict.member 3 <| Dict.fromList [(2,5),(4,5)]
False : Bool
```

`map` and `filter` work much like their list equivalents, although the functions
you pass to them will need to have a slightly different signature to account
for both the key and the value which they will be passed.

```elm
> Dict.map (\k v -> (k, v * 100)) <| Dict.fromList [(1,1),(2,4),(3,9)]
fromList [(1,100),(2,400),(3,900)] : Dict.Dict number number1
> Dict.filter (\_ v -> Char.isUpper v) <| Dict.fromList [(1,'a'),(2,'A'),(3,'b'),(4,'B')]
fromList [(2,'A'),(4,'B')] : Dict.Dict number Char
```

`toList` is the inverse of `fromList`.

```elm
> Dict.toList << Dict.insert 9 2 <| Dict.singleton 4 3
[(4,3),(9,2)] : List (number, number)
```

`keys` and `values` return lists of keys and values respectively. `keys` is the
equivalent of `List.map Tuple.first << Dict.toList` and `values` is the equivalent of
`List.map Tuple.second << Dict.toList`


These were just a few functions from `Dict`. You can see a complete
list of functions in the
[documentation](http://package.elm-lang.org/packages/elm-lang/core/5.1.1/Dict).

Set
--------

![legosets](img/legosets.png)

The `Set` module offers us, well, sets. Like sets from mathematics.
Sets are kind of like a cross between lists and dictionaries. All the elements
in a set are unique. And because they're internally implemented with
trees (much like dictionaries in `Dict`), they're ordered. Checking for
membership, inserting, deleting, etc. is much faster than doing the same
thing with lists. The most common operation when dealing with sets are
inserting into a set, checking for membership and converting a set to a
list.

Let's say we have two pieces of text. We want to find out which
characters were used in both of them.

```elm
text1 = "I just had an anime dream. Anime... Reality... Are they so different?"
text2 = "The old man left his garbage can out and now his trash is all over my lawn!"
```

The fromList function works much like you would expect. It takes a list
and converts it into a set.

```elm
> set1 = Set.fromList <| String.toList text1
Set.fromList [' ','.','?','A','I','R','a','d','e','f','h','i','j','l','m','n','o','r','s','t','u','y']
    : Set.Set Char
> set2 = Set.fromList <| String.toList text2
Set.fromList [' ','!','T','a','b','c','d','e','f','g','h','i','l','m','n','o','r','s','t','u','v','w','y']
    : Set.Set Char
```

As you can see, the items are ordered and each element is unique. Now
let's use the `intersect` function to see which elements they both
share.

```elm
> Set.intersect set1 set2
Set.fromList [' ','a','d','e','f','h','i','l','m','n','o','r','s','t','u','y']
    : Set.Set Char
```

We can use the `diff` function to see which letters are in the first
set but aren't in the second one and vice versa.

```elm
> Set.diff set1 set2
Set.fromList ['.','?','A','I','R','j'] : Set.Set Char
> Set.diff set2 set1
Set.fromList ['!','T','b','c','g','v','w'] : Set.Set Char
```

Or we can see all the unique letters used in both sentences by using
`union`.

```elm
> Set.union set1 set2
Set.fromList [' ','!','.','?','A','I','R','T','a','b','c','d','e','f','g','h','i','j','l','m','n','o','r','s','t','u','v','w','y']
    : Set.Set Char
```

The `isEmpty`, `size`, `member`, `empty`, `singleton`, `insert` and `remove` functions
all work like you'd expect them to.

```elm
> Set.isEmpty Set.empty
True : Bool
> Set.isEmpty <| Set.fromList [3,4,5,5,4,3]
False : Bool
> Set.size <| Set.fromList [3,4,5,3,4,5]
3 : Int
> Set.singleton 9
Set.fromList [9] : Set.Set number
> Set.insert 4 <| Set.fromList [9,3,8,1]
Set.fromList [1,3,4,8,9] : Set.Set number
> Set.insert 8 <| Set.fromList <| List.range 5 10
Set.fromList [5,6,7,8,9,10] : Set.Set Int
> Set.remove 4 <| Set.fromList [3,4,5,4,3,4,5]
Set.fromList [3,5] : Set.Set number
```

We can also map over sets and filter them.

```elm
> odd n = n % 2 == 1
<function> : Int -> Bool
> Set.filter odd <| Set.fromList [3,4,5,6,7,2,3,4]
Set.fromList [3,5,7] : Set.Set Int
> Set.map ((+) 1) <| Set.fromList [3,4,5,6,7,2,3,4]
Set.fromList [3,4,5,6,7,8] : Set.Set number
```

Sets are often used to weed a list of duplicates from a list by first
making it into a set with `fromList` and then converting it back to a list
with `toList`.

```elm
> setDedup xs = Set.toList <| Set.fromList xs
<function> : List comparable -> List comparable
> String.fromList <| setDedup <| String.toList "HEY WHATS CRACKALACKIN"
" ACEHIKLNRSTWY" : String
```

Keep in mind, though, that the order of the original list likely will
not be preserved.

Making our own modules
----------------------

![making modules](img/making_modules.png)

We've looked at some cool modules so far, but how do we make our own
module? Almost every programming language enables you to split your code
up into several files and Elm is no different. When making programs,
it's good practice to take functions and types that work towards a
similar purpose and put them in a module. That way, you can easily reuse
those functions in other programs by just importing your module.

Let's see how we can make our own modules by making a little module that
provides some functions for calculating the volume and area of a few
geometrical objects. We'll start by creating a file called Geometry.elm.

We say that a module *exports* functions. What that means is that when I
import a module, I can use the functions that it exports. It can define
functions that its functions call internally, but we can only see and
use the ones that it exports.

At the beginning of a module, we specify the module name. If we have a
file called Geometry.elm, then we should name our module Geometry. Then,
we specify the functions that it exports and after that, we can start
writing the functions. So we'll start with this.

```elm
module Geometry
    exposing
        ( sphereVolume
        , sphereArea
        , cubeVolume
        , cubeArea
        , cuboidArea
        , cuboidVolume
        )
```

As you can see, we'll be doing areas and volumes for spheres, cubes and
cuboids. Let's go ahead and define our functions then:

```elm
module Geometry
    exposing
        ( cubeArea
        , cubeVolume
        , cuboidArea
        , cuboidVolume
        , sphereArea
        , sphereVolume
        )


sphereVolume : Float -> Float
sphereVolume radius =
    (4.0 / 3.0) * pi * (radius ^ 3)


sphereArea : Float -> Float
sphereArea radius =
    4 * pi * (radius ^ 2)


cubeVolume : Float -> Float
cubeVolume side =
    cuboidVolume side side side


cubeArea : Float -> Float
cubeArea side =
    cuboidArea side side side


cuboidVolume : Float -> Float -> Float -> Float
cuboidVolume a b c =
    rectangleArea a b * c


cuboidArea : Float -> Float -> Float -> Float
cuboidArea a b c =
    rectangleArea a b * 2 + rectangleArea a c * 2 + rectangleArea c b * 2


rectangleArea : Float -> Float -> Float
rectangleArea a b =
    a * b
```

Pretty standard geometry right here. There are a few things to take note
of though. Because a cube is only a special case of a cuboid, we defined
its area and volume by treating it as a cuboid whose sides are all of
the same length. We also defined a helper function called `rectangleArea`,
which calculates a rectangle's area based on the lenghts of its sides.
It's rather trivial because it's just multiplication. Notice that we
used it in our functions in the module (namely `cuboidArea` and
`cuboidVolume`) but we didn't export it! Because we want our module to
just present functions for dealing with three dimensional objects, we
used `rectangleArea` but we didn't export it.

When making a module, we usually export only those functions that act as
a sort of interface to our module so that the implementation is hidden.
If someone is using our `Geometry` module, they don't have to concern
themselves with functions that we don't export. We can decide to change
those functions completely or delete them in a newer version (we could
delete `rectangleArea` and just use `*` instead) and no one will mind
because we weren't exporting them in the first place.

To use our module, we just do:

```elm
import Geometry
```

Geometry.elm has to be in the same folder that the program that's
importing it is in, though.

Modules can also be given a hierarchical structures. Each module can
have a number of sub-modules and they can have sub-modules of their own.
Let's section these functions off so that `Geometry` is a module that has
three sub-modules, one for each type of object.

First, we'll make a folder called Geometry. Mind the capital G. In it,
we'll place three files: Sphere.elm, Cuboid.elm, and Cube.elm. Here's what
the files will contain:

Sphere.elm

```elm
module Geometry.Sphere
    exposing
        ( area
        , volume
        )


volume : Float -> Float
volume radius =
    (4.0 / 3.0) * pi * (radius ^ 3)


area : Float -> Float
area radius =
    4 * pi * (radius ^ 2)
```

Cuboid.elm

```elm
module Geometry.Cuboid
    exposing
        ( area
        , volume
        )


volume : Float -> Float -> Float -> Float
volume a b c =
    rectangleArea a b * c


area : Float -> Float -> Float -> Float
area a b c =
    rectangleArea a b * 2 + rectangleArea a c * 2 + rectangleArea c b * 2


rectangleArea : Float -> Float -> Float
rectangleArea a b =
    a * b
```

Cube.elm

```elm
module Geometry.Cube
    exposing
        ( area
        , volume
        )

import Geometry.Cuboid as Cuboid


volume : Float -> Float
volume side =
    Cuboid.volume side side side


area : Float -> Float
area side =
    Cuboid.area side side side
```

Alright! So first is `Geometry.Sphere`. Notice how we placed it in a
folder called Geometry and then defined the module name as
`Geometry.Sphere`. We did the same for the cuboid. Also notice how in all
three sub-modules, we defined functions with the same names. We can do
this because they're separate modules.

So now if we're in a file that's on the same level as the Geometry
folder, we can do, say:

```elm
import Geometry.Sphere
```

And then we can call `area` and `volume` and they'll give us the area and
volume for a sphere.

The next time you find yourself writing a file that's really big and has
a lot of functions, try to see which functions serve some common purpose
and then see if you can put them in their own module. You'll be able to
just import your module the next time you're writing a program that
requires some of the same functionality.
