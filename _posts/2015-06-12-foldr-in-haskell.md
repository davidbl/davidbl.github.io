---
layout: post
title: foldr vs foldl in haskell
---

I am re-reading [Learn You a Haskell for Great Good!](http://learnyouahaskell.com/).
and I can recall my confusion from my initial reading over the example of foldr.

> If we're mapping (+3) to [1,2,3], we approach the list from the right side. We take the last element, which is 3 and apply the function to it, which ends up being 6. Then, we prepend it to the accumulator, which is was []. 6:[] is [6] and that's now the accumulator. We apply (+3) to 2, that's 5 and we prepend (:) it to the accumulator, so the accumulator is now [5,6]. We apply (+3) to 1 and prepend that to the accumulator and so the end value is [4,5,6].
>
> -- <cite>[Miran Lipovača][1]</cite>

[1]:http://learnyouahaskell.com/higher-order-functions

To the casual reader, that *might* indicate that the list is read from the right.  But, of course, that is not the case.
As Miran states in that same chapter,  for right fold
> ... the accumulator eats up the values from the right

The list is iterated from the left, but the first application of the function with the accumulator is with the right-most element

A simple implementation of right fold might look like

```haskell
foldr' _ acc [] = acc
foldr' f acc (x:xs) = f x (foldr' f acc xs)
```

If we expand the foldr example from the book, we get

```haskell
foldr (+) 0 [1,2,3]

(+) 1 (foldr' (+) 0 [2,3])
(+) 1 (+ 2 (foldr' (+) 0 [3]))
(+) 1 (+ 2 (+ 3 (foldr' (+) 0 [])))
(+) 1 (+ 2 (+ 3 0)))
```

then,  if we *pop* off the operations, the first addition is the initial accumlator value
with the right-most element of the list

```haskell
(+) 1 (+ 2 (+ 3 0)))
(+) 1 (+ 2 (3))
(+) 1 (5)
6
```

and, for completeness, here is a left fold expanded

```haskell
foldl' _ acc [] = acc
foldl' f acc (x:xs) = foldl' f (f x acc) xs
```

which, for the sum example, would expand to

```haskell
foldl' (+) 0 [1,2,3]

foldl' (+) (+ 0 1) [2,3]
--  note the function application expression will be evaluated before the next iteration
foldl' (+) 1 [2,3]

foldl' (+) (+ 1 2) [3]
foldl' (+) 3 [3]

foldl' (+) (+ 3 3) []
foldl' (+) 6 []

6
```

so, we can see that both foldr and foldl iterated the items of the list starting from the left,
but foldr first applies the function (with the accumulator) to the right-most elem whilst
foldl first applies the function to the left-most element
