---
layout: post
title: Zippers
---

Zippers
=======

![hi im chet](img/60sdude.png)

While Elm's purity comes with a whole bunch of benefits, it makes us
tackle some problems differently than we would in impure languages.
Because of referential transparency, one value is as good as another in
Elm if it represents the same thing.

So if we have a tree full of fives (high-fives, maybe?) and we want to
change one of them into a six, we have to have some way of knowing
exactly which five in our tree we want to change. We have to know where
it is in our tree. In impure languages, we could just note where in our
memory the five is located and change that. But in Elm, one five is
as good as another, so we can't discriminate based on where in our
memory they are. We also can't really *change* anything; when we say
that we change a tree, we actually mean that we take a tree and return a
new one that's similar to the original tree, but slightly different.

One thing we can do is to remember a path from the root of the tree to
the element that we want to change. We could say, take this tree, go
left, go right and then left again and change the element that's there.
While this works, it can be inefficient. If we want to later change an
element that's near the element that we previously changed, we have to
walk all the way from the root of the tree to our element again!

In this chapter, we'll see how we can take some data structure and focus
on a part of it in a way that makes changing its elements easy and
walking around it efficient. Nice!

Taking a walk
-------------

Like we've learned in biology class, there are many different kinds of
trees, so let's pick a seed that we will use to plant ours. Here it is:

```elm
type Tree a = Empty | Node a (Tree a) (Tree a)
```

So our tree is either empty or it's a node that has an element and two
sub-trees. Here's a fine example of such a tree, which I give to you,
the reader, for free!

```elm
freeTree : Tree Char
freeTree =
    Node 'P'
        (Node 'O'
            (Node 'L'
                (Node 'N' Empty Empty)
                (Node 'T' Empty Empty)
            )
            (Node 'Y'
                (Node 'S' Empty Empty)
                (Node 'A' Empty Empty)
            )
        )
        (Node 'L'
            (Node 'W'
                (Node 'C' Empty Empty)
                (Node 'R' Empty Empty)
            )
            (Node 'A'
                (Node 'A' Empty Empty)
                (Node 'C' Empty Empty)
            )
        )
```

And here's this tree represented graphically:

![polly says her back hurts](img/pollywantsa.png)

Notice that `'W'` in the tree there? Say we want to change it into a `'P'`. How
would we go about doing that? Well, one way would be to pattern match on
our tree until we find the element that's located by first going right
and then left and changing said element. Here's the code for this:

```elm
changeToP : Tree Char -> Maybe (Tree Char)
changeToP t = case t of
  (Node x l (Node y (Node _ m n) r)) -> Just (Node x l (Node y (Node 'P' m n) r))
  _ -> Nothing
```

One thing to note right now is that our `changeToP` function returns a
`Maybe (Tree Char)`. Because the tree we pass to this function may not
have the path we're matching against, we need to account for the possibility
of this function failing. This will be a common theme in this chapter, and it
will be discussed periodically throughout the chapter as the need arises,
but there will also be a more in-depth discussion at the end. So feel free
to briefly skip to the end if you're confused. Otherwise, just remember,
as we saw earlier, `Maybe` is just a way of encapsulating the possibility
of failure in our data type, and know that this will have some implications
for the design and implementation of the functions throughout this chapter.

So back to the function.
Yuck! Not only is this rather ugly, it's also kind of confusing. What
happens here? Well, we pattern match on our tree and name its root
element `x` (that's becomes the `'P'` in the root) and its left sub-tree `l`.
Instead of giving a name to its right sub-tree, we further pattern match
on it. We continue this pattern matching until we reach the sub-tree
whose root is our `'W'`. Once we've done this, we rebuild the tree, only
the sub-tree that contained the `'W'` at its root now has a `'P'`.

Is there a better way of doing this? How about we make our function take
a tree along with a list of directions. The directions will be either `L`
or `R`, representing left and right respectively, and we'll change the
element that we arrive at if we follow the supplied directions. Here it
is:

```elm
type Direction = L | R
type alias Directions = List Direction

changeToP : Directions -> Tree Char -> Maybe (Tree Char)
changeToP d t = case (d, t) of
  ((L::ds), (Node x l r)) ->
    Maybe.map (\l -> Node x l r) (changeToP ds l)
    
  ((R::ds), (Node x l r)) ->
    Maybe.map (\r -> Node x l r) (changeToP ds r)
    
  ([], (Node _ l r)) ->
    Just (Node 'P' l r)
    
  (_, t) ->
    Nothing
```

If the first element in the our list of directions is `L`, we construct a
new tree that's like the old tree, only its left sub-tree has an element
changed to `'P'`. When we recursively call `changeToP`, we give it only the
tail of the list of directions, because we already took a left. We do
the same thing in the case of an `R`. If the list of directions is empty,
that means that we're at our destination, so we return a tree that's
like the one supplied, only it has `'P'` as its root element. And, if the
list of directions passed to the function describes a path which does not
exist through our tree, we ultimately return `Nothing`.

To avoid printing out the whole tree, let's make a function that takes a
list of directions and tells us what the element at the destination is:

```elm
elemAt : Directions -> Tree a -> Maybe a
elemAt d t = case (d, t) of
    ((L::ds), (Node _ l _)) -> elemAt ds l
    ((R::ds), (Node _ _ r)) -> elemAt ds r
    ([], (Node x _ _)) -> Just x
    _ -> Nothing
```

This function is actually quite similar to changeToP, only instead of
remembering stuff along the way and reconstructing the tree, it ignores
everything except its destination. Here we change the 'W' to a 'P' and
see if the change in our new tree sticks:

```elm
> elemAt [R,L] newTree
Just 'W' : Maybe.Maybe Char
> elemAt [R,L] <| changeToP [R,L] freeTree
Just 'P' : Maybe.Maybe Char
```

Nice, this seems to work. In these functions, the list of directions
acts as a sort of *focus*, because it pinpoints one exact sub-tree from
our tree. A direction list of `[R]` focuses on the sub-tree that's right
of the root, for example. An empty direction list focuses on the main
tree itself.

While this technique may seem cool, it can be rather inefficient,
especially if we want to repeatedly change elements. Say we have a
really huge tree and a long direction list that points to some element
all the way at the bottom of the tree. We use the direction list to take
a walk along the tree and change an element at the bottom. If we want to
change another element that's close to the element that we've just
changed, we have to start from the root of the tree and walk all the way
to the bottom again! What a drag.

In the next section, we'll find a better way of focusing on a sub-tree,
one that allows us to efficiently switch focus to sub-trees that are
nearby.

A trail of breadcrumbs
----------------------

![whoop dee doo](img/bread.png)

Okay, so for focusing on a sub-tree, we want something better than just
a list of directions that we always follow from the root of our tree.
Would it help if we start at the root of the tree and move either left
or right one step at a time and sort of leave breadcrumbs? That is, when
we go left, we remember that we went left and when we go right, we
remember that we went right. Sure, we can try that.

To represent our breadcrumbs, we'll also use a list of `Direction` (which
is either `L` or `R`), only instead of calling it `Directions`, we'll call it
`Breadcrumbs`, because our directions will now be reversed since we're
leaving them as we go down our tree:

```elm
type alias Breadcrumbs = List Direction
```

Here's a function that takes a tree and some breadcrumbs and moves to
the left sub-tree while adding `L` to the head of the list that represents
our breadcrumbs:

```elm
goLeft : (Tree a, Breadcrumbs) -> Maybe (Tree a, Breadcrumbs)
goLeft t = case t of
  (Node _ l _, bs) -> Just (l, L::bs)
  _ -> Nothing
```

We ignore the element at the root and the right sub-tree and just return
the left sub-tree along with the old breadcrumbs with `L` as the head.
Note that an empty tree doesn't have any sub-trees, so when we call `goLeft`
on an empty tree, there isn't anything meanigful to return. As we've seen with
`List.head`, we can return a `Maybe` type to encaptulate this value that may or
may not be returned.

Here's a function to go right:

```elm
goRight : (Tree a, Breadcrumbs) -> Maybe (Tree a, Breadcrumbs)
goRight t = case t of
  (Node _ _ r, bs) -> Just (r, R::bs)
  _ -> Nothing
```

It works the same way. Let's use these functions to take our `freeTree`
and go right and then left:

```elm
> goLeft (goRight (freeTree, []))
Just (Node 'W' (Node 'C' Empty Empty) (Node 'R' Empty Empty),[L,R]) : Maybe (Tree Char, Breadcrumbs)
```

![almostthere](img/almostzipper.png)

Okay, so now we have a tree that has `'W'` in its root and `'C'` in the root
of its left sub-tree and `'R'` in the root of its right sub-tree. The
breadcrumbs are `[L,R]`, because we first went right and then left.

To make walking along our tree clearer, we can use the `|>` function
from the core library, which is defined like so:

```elm
(|>) : a -> (a -> b) -> b
(|>) x f = f x
```

Which allows us to apply functions to values by first writing the value,
then writing a `|>` and then the function. So instead of `goRight
(freeTree, [])`, we can write `(freeTree, []) |> goRight`. Using this, we
can rewrite the above so that it's more apparent that we're first going
right and then left:

```elm
> (freeTree, []) |> goRight |> Maybe.andThen goLeft
Just (Node 'W' (Node 'C' Empty Empty) (Node 'R' Empty Empty),[L,R]) : Maybe (Tree Char, Breadcrumbs)
```

We used `Maybe.andThen`, because we can't simply pass the value of
one application on to the next function. As was mentioned earlier,
we have encapsulated the possibility of failure in our return type,
as `Maybe (Tree a, Breadcrumbs)`. However, this type doesn't align with the
input parameter type. We can't pass `goLeft` a `Maybe (Tree a, Breadcrumbs)`.
We need a way to unpack this `Maybe` to get the value out, while also
considering what should be done in the event that we're dealing with
`Nothing`. And, in a nutshell, that's what `Maybe.andThen` is going to do for us.
We'll talk more about this at the end of the chapter, but for now all you need
to know is that `Maybe.andThen` helps us keep a handle on the constant possibility
of failure at each step, so we can chain together our `goLeft`'s and `goRight`'s
like this, even though any one of them could fail at any moment.

### Going back up

What if we now want to go back up in our tree? From our breadcrumbs we
know that the current tree is the left sub-tree of its parent and that
it is the right sub-tree of its parent, but that's it. They don't tell
us enough about the parent of the current sub-tree for us to be able to
go up in the tree. It would seem that apart from the direction that we
took, a single breadcrumb should also contain all other data that we
need to go back up. In this case, that's the element in the parent tree
along with its right sub-tree.

In general, a single breadcrumb should contain all the data needed to
reconstruct the parent node. So it should have the information from all
the paths that we didn't take and it should also know the direction that
we did take, but it must not contain the sub-tree that we're currently
focusing on. That's because we already have that sub-tree in the first
component of the tuple, so if we also had it in the breadcrumbs, we'd
have duplicate information.

Let's modify our breadcrumbs so that they also contain information about
everything that we previously ignored when moving left and right.
Instead of `Direction`, we'll make a new data type:

```elm
type Crumb a = LeftCrumb a (Tree a) | RightCrumb a (Tree a)
```

Now, instead of just `L`, we have a `LeftCrumb` that also contains the
element in the node that we moved from and the right tree that we didn't
visit. Instead of `R`, we have `RightCrumb`, which contains the element in
the node that we moved from and the left tree that we didn't visit.

These breadcrumbs now contain all the data needed to recreate the tree
that we walked through. So instead of just being normal bread crumbs,
they're now more like floppy disks that we leave as we go along, because
they contain a lot more information than just the direction that we
took.

In essence, every breadcrumb is now like a tree node with a hole in it.
When we move deeper into a tree, the breadcrumb carries all the
information that the node that we moved away from carried *except* the
sub-tree that we chose to focus on. It also has to note where the hole
is. In the case of a `LeftCrumb`, we know that we moved left, so the
sub-tree that's missing is the left one.

Let's also change our `Breadcrumbs` type alias to reflect this:

```elm
type alias Breadcrumbs a = List (Crumb a)
```

Next up, we have to modify the `goLeft` and `goRight` functions to store
information about the paths that we didn't take in our breadcrumbs,
instead of ignoring that information like they did before. Here's
`goLeft`:

```elm
goLeft : (Tree a, Breadcrumbs a) -> Maybe (Tree a, Breadcrumbs a)
goLeft t = case t of
  (Node x l r, bs) -> Just (l, LeftCrumb x r::bs)
  _ -> Nothing
```

You can see that it's very similar to our previous `goLeft`, only instead
of just adding a `L` to the head of our list of breadcrumbs, we add a
`LeftCrumb` to signify that we went left and we equip our `LeftCrumb` with
the element in the node that we moved from (that's the `x`) and the right
sub-tree that we chose not to visit.

`goRight` is similar:

```elm
goRight : (Tree a, Breadcrumbs a) -> Maybe (Tree a, Breadcrumbs a)
goRight t = case t of
  (Node x l r, bs) -> Just (r, RightCrumb x l::bs)
  _ -> Nothing
```

We were previously able to go left and right. What we've gotten now is
the ability to actualy go back up by remembering stuff about the parent
nodes and the paths that we didn't visit. Here's the `goUp` function:

```elm
goUp : (Tree a, Breadcrumbs a) -> Maybe (Tree a, Breadcrumbs a)
goUp t = case t of
  (t, LeftCrumb x r::bs) -> Just (Node x t r, bs)
  (t, RightCrumb x l::bs) -> Just (Node x l t, bs)
  _ -> Nothing
```

![asstronaut](img/asstronaut.png)

We're focusing on the tree `t` and we check what the latest `Crumb` is. If
it's a `LeftCrumb`, then we construct a new tree where our tree `t` is the
left sub-tree and we use the information about the right sub-tree that
we didn't visit and the element to fill out the rest of the `Node`.
Because we moved back so to speak and picked up the last breadcrumb to
recreate with it the parent tree, the new list of breadcrumbs doesn't
contain it.

With a pair of `Tree a` and `Breadcrumbs a`, we have all the information to
rebuild the whole tree and we also have a focus on a sub-tree. This
scheme also enables us to easily move up, left and right. Such a pair
that contains a focused part of a data structure and its surroundings is
called a zipper, because moving our focus up and down the data structure
resembles the operation of a zipper on a regular pair of pants. So it's
cool to make a type alias as such:

```elm
type alias Zipper a = (Tree a, Breadcrumbs a)
```

I'd prefer naming the type alias `Focus` because that makes it clearer
that we're focusing on a part of a data structure, but the term zipper
is more widely used to describe such a setup, so we'll stick with
`Zipper`.

### Manipulating trees under focus

Now that we can move up and down, let's make a function that modifies
the element in the root of the sub-tree that the zipper is focusing on:

```elm
modify : (a -> a) -> Zipper a -> Maybe (Zipper a)
modify f z = case z of
    (Node x l r, bs) -> Just (Node (f x) l r, bs)
    (Empty, bs) -> Nothing
```

If we're focusing on a node, we modify its root element with the
function `f`. If we're focusing on an empty tree, we return `Nothing`.
Now we can start off with a tree, move to anywhere we want and modify an
element, all while keeping focus on that element so that we can easily
move further up or down. An example:

```elm
> newFocus = Maybe.andThen (modify (\_ -> 'P')) (Maybe.andThen goRight (goLeft (freeTree,[])))
```

We go left, then right and then modify the root element by replacing it
with a `'P'`. This reads even better if we use `|>`:

```elm
> newFocus = (freeTree,[]) 
  |> goLeft
  |> Maybe.andThen goRight
  |> Maybe.andThen (modify (\_ -> 'P'))
```

We can then move up if we want and replace an element with a mysterious
`'X'`:

```elm
> Maybe.andThen (modify (\_ -> 'X')) (Maybe.andThen goUp newFocus)
```

Or if we wrote it with `|>`:

```elm
> newFocus2 = newFocus |> Maybe.andThen goUp |> Maybe.andThen (modify (\_ -> 'X'))
```

Moving up is easy because the breadcrumbs that we leave form the part of
the data structure that we're not focusing on, but it's inverted, sort
of like turning a sock inside out. That's why when we want to move up,
we don't have to start from the root and make our way down, but we just
take the top of our inverted tree, thereby uninverting a part of it and
adding it to our focus.

Each node has two sub-trees, even if those sub-trees are empty trees. So
if we're focusing on an empty sub-tree, one thing we can do is to
replace it with a non-empty subtree, thus attaching a tree to a leaf
node. The code for this is simple:

```elm
attach : Tree a -> Zipper a -> Zipper a
attach t z = case z of
  (_, bs) -> (t, bs)
```

We take a tree and a zipper and return a new zipper that has its focus
replaced with the supplied tree. Not only can we extend trees this way
by replacing empty sub-trees with new trees, we can also replace whole
existing sub-trees. Let's attach a tree to the far left of our `freeTree`:

```elm
> farLeft = (freeTree,[]) |> goLeft |> Maybe.andThen goLeft |> Maybe.andThen goLeft |> Maybe.andThen goLeft
> newFocus = farLeft |> Maybe.map (attach (Node 'Z' Empty Empty))
```

`newFocus` is now focused on the tree that we just attached and the rest
of the tree lies inverted in the breadcrumbs. If we were to use `goUp` to
walk all the way to the top of the tree, it would be the same tree as
`freeTree` but with an additional `'Z'` on its far left.

### I'm going straight to the top, oh yeah, up where the air is fresh and clean!

Making a function that walks all the way to the top of the tree,
regardless of what we're focusing on, is really easy. Here it is:

```elm
topMost : Zipper a -> Maybe (Zipper a)
topMost z = case z of
  (t,[]) -> Just (t,[])
  z -> Maybe.andThen topMost (goUp z)
```

If our trail of beefed up breadcrumbs is empty, this means that we're
already at the root of our tree, so we just return the current focus.
Otherwise, we go up to get the focus of the parent node and then
recursively apply `topMost` to that. So now we can walk around our tree,
going left and right and up, applying `modify` and `attach` as we go along
and then when we're done with our modifications, we use `topMost` to focus
on the root of our tree and see the changes that we've done in proper
perspective.

Focusing on lists
-----------------

Zippers can be used with pretty much any data structure, so it's no
surprise that they can be used to focus on sub-lists of lists. After
all, lists are pretty much like trees, only where a node in a tree has
an element (or not) and several sub-trees, a node in a list has an
element and only a single sub-list. When we [implemented our own
lists](08-making-our-own-types-and-typeclasses#recursive-data-structures),
we defined our data type like so:

```elm
type List a = Empty | Cons a (List a)
```

![the best damn thing](img/picard.png)

Contrast this with our definition of our binary tree and it's easy to
see how lists can be viewed as trees where each node has only one
sub-tree.

A list like `[1,2,3]` can be written as `1::2::3::[]`. It consists of the
head of the list, which is `1` and then the list's tail, which is `2::3::[]`.
In turn, `2::3::[]` also has a head, which is `2` and a tail, which is
`3::[]`. With `3::[]`, the `3` is the head and the tail is the empty
list `[]`.

Let's make a zipper for lists. To change the focus on sub-lists of a
list, we move either forward or back (whereas with trees we moved either
up or left or right). The focused part will be a sub-tree and along with
that we'll leave breadcrumbs as we move forward. Now what would a single
breadcrumb for a list consist of? When we were dealing with binary
trees, we said that a breadcrumb has to hold the element in the root of
the parent node along with all the sub-trees that we didn't choose. It
also had to remember if we went left or right. So, it had to have all
the information that a node has except for the sub-tree that we chose to
focus on.

Lists are simpler than trees, so we don't have to remember if we went
left or right, because there's only one way to go deeper into a list.
Because there's only one sub-tree to each node, we don't have to
remember the paths that we didn't take either. It seems that all we have
to remember is the previous element. If we have a list like `[3,4,5]` and
we know that the previous element was `2`, we can go back by just putting
that element at the head of our list, getting `[2,3,4,5]`.

Because a single breadcrumb here is just the element, we don't really
have to put it inside a data type, like we did when we made the `Crumb`
data type for tree zippers:

```elm
type alias ListZipper a = (List a, List a)
```

The first list represents the list that we're focusing on and the second
list is the list of breadcrumbs. Let's make functions that go forward
and back into lists:

```elm
goForward : ListZipper a -> Maybe (ListZipper a)
goForward lz = case lz of
  (x::xs, bs) -> Just (xs, x::bs)
  _ -> Nothing

goBack : ListZipper a -> Maybe (ListZipper a)
goBack lz = case lz of
  (xs, b::bs) -> Just (b::xs, bs)
  _ -> Nothing
```

When we're going forward, we focus on the tail of the current list and
leave the head element as a breadcrumb. When we're moving backwards, we
take the latest breadcrumb and put it at the beginning of the list.

Here are these two functions in action:

```elm
> xs = [1,2,3,4]
> goForward (xs,[])
Just ([2,3,4],[1]) : Maybe.Maybe (ListZipper a)
> goForward ([2,3,4],[1])
Just ([3,4],[2,1]) : Maybe.Maybe (ListZipper a)
> goForward ([3,4],[2,1])
Just ([4],[3,2,1]) : Maybe.Maybe (ListZipper a)
> goBack ([4],[3,2,1])
Just ([3,4],[2,1]) : Maybe.Maybe (ListZipper a)
```

We see that the breadcrumbs in the case of lists are nothing more but a
reversed part of our list. The element that we move away from always
goes into the head of the breadcrumbs, so it's easy to move back by just
taking that element from the head of the breadcrumbs and making it the
head of our focus.

This also makes it easier to see why we call this a zipper, because this
really looks like the slider of a zipper moving up and down.

If you were making a text editor, you could use a list of strings to
represent the lines of text that are currently opened and you could then
use a zipper so that you know which line the cursor is currently focused
on. By using a zipper, it would also make it easier to insert new lines
anywhere in the text or delete existing ones.

A very simple file system
-------------------------

Now that we know how zippers work, let's use trees to represent a very
simple file system and then make a zipper for that file system, which
will allow us to move between folders, just like we usually do when
jumping around our file system.

If we take a simplistic view of the average hierarchical file system, we
see that it's mostly made up of files and folders. Files are units of
data and come with a name, whereas folders are used to organize those
files and can contain files or other folders. So let's say that an item
in a file system is either a file, which comes with a name and some
data, or a folder, which has a name and then a bunch of items that are
either files or folders themselves. Here's a data type for this and some
type aliases so we know what's what:

```elm
type alias Name = String
type alias Data = String
type FSItem = File Name Data | Folder Name (List FSItem)
```

A file comes with two strings, which represent its name and the data it
holds. A folder comes with a string that is its name and a list of
items. If that list is empty, then we have an empty folder.

Here's a folder with some files and sub-folders:

```elm
myDisk : FSItem
myDisk =
    Folder "root"
        [ File "goat_yelling_like_man.wmv" "baaaaaa"
        , File "pope_time.avi" "god bless"
        , Folder "pics"
            [ File "ape_throwing_up.jpg" "bleargh"
            , File "watermelon_smash.gif" "smash!!"
            , File "skull_man(scary).bmp" "Yikes!"
            ]
        , File "dijon_poupon.doc" "best mustard"
        , Folder "programs"
            [ File "fartwizard.exe" "10gotofart"
            , File "owl_bandit.dmg" "mov eax, h00t"
            , File "not_a_virus.exe" "really not a virus"
            , Folder "source code"
                [ File "best_elm_prog.elm" "main = toString (fix error)"
                , File "random.elm" "main = toString 4"
                ]
            ]
        ]
```

That's actually what my disk contains right now.

### A zipper for our file system

![spongedisk](img/spongedisk.png)

Now that we have a file system, all we need is a zipper so we can zip
and zoom around it and add, modify and remove files as well as folders.
Like with binary trees and lists, we're going to be leaving breadcrumbs
that contain info about all the stuff that we chose not to visit. Like
we said, a single breadcrumb should be kind of like a node, only it
should contain everything except the sub-tree that we're currently
focusing on. It should also note where the hole is so that once we move
back up, we can plug our previous focus into the hole.

In this case, a breadcrumb should be like a folder, only it should be
missing the folder that we currently chose. Why not like a file, you
ask? Well, because once we're focusing on a file, we can't move deeper
into the file system, so it doesn't make sense to leave a breadcrumb
that says that we came from a file. A file is sort of like an empty
tree.

If we're focusing on the folder "root" and we then focus on the file
`"dijon_poupon.doc"`, what should the breadcrumb that we leave look like?
Well, it should contain the name of its parent folder along with the
items that come before the file that we're focusing on and the items
that come after it. So all we need is a `Name` and two lists of items. By
keeping separate lists for the items that come before the item that
we're focusing and for the items that come after it, we know exactly
where to place it once we move back up. So this way, we know where the
hole is.

Here's our breadcrumb type for the file system:

```elm
type FSCrumb = FSCrumb Name (List FSItem) (List FSItem)
```

And here's a type alias for our zipper:

```elm
type alias FSZipper = (FSItem, List FSCrumb)
```

Going back up in the hierarchy is very simple. We just take the latest
breadcrumb and assemble a new focus from the current focus and
breadcrumb. Like so:

```elm
fsUp : FSZipper -> Maybe FSZipper
fsUp fsz = case fsz of
  (item, FSCrumb name ls rs::bs) -> Just (Folder name (ls ++ [item] ++ rs), bs)
  _ -> Nothing
```

Because our breadcrumb knew what the parent folder's name was, as well
as the items that came before our focused item in the folder (that's `ls`)
and the ones that came after (that's `rs`), moving up was easy.

How about going deeper into the file system? If we're in the `"root"` and
we want to focus on `"dijon_poupon.doc"`, the breadcrumb that we leave is
going to include the name `"root"` along with the items that precede
`"dijon_poupon.doc"` and the ones that come after it.

Here's a function that, given a name, focuses on a file or folder that's
located in the current focused folder:

```elm
break : (a -> Bool) -> List a -> (List a, List a)
break p ls = case ls of
  [] ->  ([], [])
  (x::xs) ->
    if p x
      then
        ([], ls)
      else
        let
          (ys, zs) = break p xs
        in
        (x::ys,zs)

fsTo : Name -> FSZipper -> Maybe FSZipper
fsTo name fsz = case fsz of
  (Folder folderName items, bs) ->
    let
      (ls, rs) = break (nameIs name) items
    in
      case rs of
        (item::rs) -> Just (item, (FSCrumb folderName ls rs::bs))
        _ -> Nothing
  _ -> Nothing

nameIs : Name -> FSItem -> Bool
nameIs name fsi = case fsi of
  (Folder folderName _) -> name == folderName
  (File fileName _) -> name == fileName
```

fsTo takes a `Name` and a `FSZipper` and returns a new `Maybe FSZipper` that focuses
on the file with the given name. That file has to be in the current
focused folder. This function doesn't search all over the place, it just
looks at the current folder.

![wow cool great](img/cool.png)

First we use a helper function `break` which we implemented here
to break the list of items in a folder into those
that precede the file that we're searching for and those that come after
it. `break` takes a predicate and a list and returns a
pair of lists. The first list in the pair holds items for which the
predicate returns `False`. Then, once the predicate returns `True` for an
item, it places that item and the rest of the list in the second item of
the pair. We also made another auxilliary function called `nameIs` that
takes a name and a file system item and returns `True` if the names match.

So now, `ls` is a list that contains the items that precede the item that
we're searching for, `item` is that very item and `rs` is the list of items
that come after it in its folder. Now that we have this, we just present
the item that we got from `break` as the focus and build a breadcrumb that
has all the data it needs.

Note that if the name we're looking for isn't in the folder, our patterns
won't match, and we'll fall through to the case where we return `Nothing`.
And similarly if our current focus isn't a folder at all but a file. This
is yet another example of how Elm forces us to consider all of the possible
edge cases, and to plan and design for failure, instead of maybe someday
in some far off future trying to shoehorn error-handling in where it
wasn't designed to go.

Anyway, now we can move up and down our file system. Let's start at the root and
walk to the file `"skull_man(scary).bmp"`:

```elm
> newFocus = (myDisk,[]) |> fsTo "pics" |> Maybe.andThen (fsTo "skull_man(scary).bmp")
```

`newFocus` is now a zipper that's focused on the `"skull_man(scary).bmp"`
file. Let's get the first component of the zipper (the focus itself) and
see if that's really true:

```elm
> Maybe.map Tuple.first newFocus
Just (File "skull_man(scary).bmp" "Yikes!") : Maybe.Maybe FSItem
```

Let's move up and then focus on its neighboring file
`"watermelon_smash.gif"`:

```elm
> newFocus2 = newFocus |> Maybe.andThen fsUp |> Maybe.andThen fsTo "watermelon_smash.gif"
> Maybe.map Tuple.first newFocus2
Just (File "watermelon_smash.gif" "smash!!") : Maybe.Maybe FSItem
```

### Manipulating our file system

Now that we know how to navigate our file system, manipulating it is
easy. Here's a function that renames the currently focused file or
folder:

```elm
fsRename : Name -> FSZipper -> FSZipper
fsRename newName fsz = case fsz of
  (Folder name items, bs) -> (Folder newName items, bs)
  (File name dat, bs) -> (File newName dat, bs)
```

Now we can rename our `"pics"` folder to `"cspi"`:

```elm
> newFocus = (myDisk,[]) |> fsTo "pics" |> Maybe.map (fsRename "cspi") |> Maybe.andThen fsUp
```

We descended to the `"pics"` folder, renamed it and then moved back up.

How about a function that makes a new item in the current folder?
Behold:

```elm
fsNewFile : FSItem -> FSZipper -> Maybe FSZipper
fsNewFile item fsz = case fsz of
  (Folder folderName items, bs) -> Just (Folder folderName (item::items), bs)
  _ -> Nothing
```

Easy as pie. Note what happens if we try to add an item but we weren't focusing
on a folder, but were focusing on a file instead. This function returns a
`Maybe` type, to indicate that this operation might fail. A
`Result` might have been more appropriate here, semantically,
but as we're already working with `Maybe`s elsewhere, it will be simpler for the
sake of the example to use `Maybe` here as well.

Let's add a file to our `"pics"` folder and then move back up to the root:

```elm
> newFocus = (myDisk,[]) 
  |> fsTo "pics"
  |> Maybe.andThen (fsNewFile (File "heh.jpg" "lol"))
  |> Maybe.andThen fsUp
```

What's really cool about all this is that when we modify our file
system, it doesn't actually modify it in place but it returns a whole
new file system. That way, we have access to our old file system (in
this case, `myDisk`) as well as the new one (the first component of
`newFocus`). So by using zippers, we get versioning for free, meaning that
we can always refer to older versions of data structures even after
we've changed them, so to speak. This isn't unique to zippers, but is a
property of Elm because its data structures are immutable. With
zippers however, we get the ability to easily and efficiently walk
around our data structures, so the persistence of Elm's data
structures really begins to shine.

Watch your step
---------------

In the examples throughout this chapter, while walking through our
data structures, whether they were binary trees, lists or file systems,
we always considered what would happen if we took a step too far and fell
off the edge. For instance, our `goLeft` function takes a zipper of a binary
tree and moves the focus to its left sub-tree:

```elm
goLeft : Zipper a -> Maybe (Zipper a)
goLeft z = case z of
  (Node x l r, bs) -> Just (l, LeftCrumb x r::bs)
  _ -> Nothing
```

![falling for you](img/bigtree.png)

What happens if you were to try to go left from an empty tree? There's
nowhere to go. Because Elm requires that all possibilities are covered when we
pattern match, in our example functions (such as `goLeft`, `goRight`, etc.)
we were forced to consider the case(s) where these functions could
fail. For example, if someone were to call `goLeft` on an empty tree.
What we choose to do in these cases was entirely up to us.
Taking `goLeft` again as an example, if we really, really wanted the return
type for this function to be a `Zipper`, perhaps we could have chose to
just return an empty zipper when `goLeft` was called on an empty tree.
But an empty zipper would also be a valid return value in other cases, so
handling this edge case like this might cause strange and unexpected results
in our program. What we really needed was a return type that can
represent either success or failure. It's no surprise, then that
we chose to use the `Maybe` monad which adds a context of
possible failure to normal values. We could have also chose to use
the the `Result` type, which is also in Elm's core library.
The `Result` type reprents the possibilities of either a
success or failure, where the failure has it's own associated value
(e.g. an error string), as opposed to the empty `Nothing` we get with `Maybe`.
These are both good choices, but which one you choose for a given situation
will just depend on your requirements. For these examples, we didn't need
to convey any information about the failures, so the simpler `Maybe` suited
our needs.

Our choice of handling these failures with `Maybe` had some other
implications in our program. If our `goLeft` function had a type
signature like `Zipper a -> Zipper a`, we could have
simply used forward function application to naturally express navigation
throughout a tree. E.g.

```elm
> newFocus = (freeTree,[])
  |> goLeft
  |> goRight
  |> goLeft
  |> goLeft
```

However, as we could encounter failure at any one of these steps, we can't
simply blindly apply these functions one after the other. We might have failed
on the previous step. We might have failed ten steps back. And furthermore,
our types don't line up, anyway, so the Elm compiler won't let us apply functions
like this even if we really wanted to. `goLeft` and `goRight` both have type
signatures like `Zipper a -> Maybe (Zipper a)`, so we can't use forward
function application as we did in the example above, because we can't apply
the result of one call as an argument to the next.

All is not lost, however. As you may have noticed, we were able to
express our navigation in a way very similar to the example above, but
with one very minor difference. We used forward function application,
but we passed the results forward, not to the next `goLeft` or `goRight`
in the sequence, but to `Maybe.andThen` applied to the next `goLeft` or
`goRight`. So what's the difference? Well, `Maybe.andThen` is defined like
this:

```elm
andThen : (a -> Maybe b) -> Maybe a -> Maybe b
andThen fn maybe = case maybe of
  Just value -> fn value
  Nothing -> Nothing
```

The implication that will hopefully be clear from the type signature,
and from the implementation itself, is that with `andThen` we can define
a sequence of functions, where the result of the previous function will
be applied to the next function in the sequence, just as we did before,
but any function in the sequence can fail and we don't need to worry
about the following functions in the sequence. If we encounter a failure
at any point (i.e. one of our functions returns `Nothing`), all following
functions in the sequence will short-circuit and return `Nothing` as well,
and overall the expression will evaluate to `Nothing`. Pretty neat. Using the 
example above, this would look like:

```elm
> newFocus = (freeTree,[])
  |> goLeft
  |> Maybe.andThen goRight
  |> Maybe.andThen goLeft
  |> Maybe.andThen goLeft
```

This may feel a little cumbersome, though, after a while, so we can
define a helper function that will combine the forward application
with `Maybe.andThen`. Keeping in mind that `|>`'s type signature is
`a -> (a -> b) -> b` (and remember it's an infix function), and that
`andThen`'s type signature is `(a -> Maybe b) -> Maybe a -> Maybe b`
it should hopefully be clear, after some careful consideration, that
what we're after is a function with the following signature:
`Maybe a -> (a -> Maybe b) -> Maybe b`. Which, if you squint at it
you may notice this is just `andThen` with the first two arguments
flipped. And we already know a function which can flip the first
two arguments for us! It's called `flip`! So putting it all together,
we come up with:

```elm
(>>=) : Maybe a -> (a -> Maybe b) -> Maybe b
(>>=) = flip Maybe.andThen
infixl 0 >>=
```

Using our new function, our example above becomes:

```elm
> newFocus = (freeTree,[])
  |> goLeft
  >>= goRight
  >>= goLeft
  >>= goLeft
```

This new function we've created is actually called `bind` in Haskell (which
is where we borrowed the `>>=` syntax from), and
is called `flatMap` in some other languages like Scala, and it's one of the
two fundamental operations for monads (the other being `return`, aka `identity`,
which just takes a plain value and turns it into a monad). The significance to us
here is that `bind` allows us to take our `Maybe` monad, which is (potentially)
wrapping up some value, apply some function directly to the wrapped value itself,
and to return a new monad. In other words, it's unwrapping the plain value
`a` from `Maybe a`, applying a function `(a -> Maybe b)` to this value,
and returning a newly wrapped `Maybe b`, which we can feed into the next
function in the chain via `>>=`. It's also handling the special context of
the `Maybe` monad, namely the potential for failure, and as such if there's
no value to unwrap, this context gets maintained and passed along.
And, in fact `Maybe.andThen` really already did
all of this for us. It was already equivalent to the monadic `bind` or `flatMap`.
All we did here was put the arguments in a nicer order, and provide an infix
operator which we could use to make our program a little more succinct. Either
method is fine, so take your pick.
