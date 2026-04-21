+++
title = "Leetcoding in Haskell"
date = 2026-04-21
+++

I like Haskell, but I feel like a fraud. That's because I haven't done
anything beyond *Introduction to functional programming*. But today that's going
to change!

I found 2 interesting second book on Haskell.
- *Category Theory for Programmers*: hopefully I can finally understand Monad
- *Haskell High Performance Programming*: learning tricks and optimizations
- Maybe also *Effective Haskell*?

Also, I tried to attend programming contest with Haskell once. Wrote an O(n^3) solution
and failed miserably.

I am trying to pick that up again. Do some selective problems
on the [CSES Problem Set](https://cses.fi/problemset/list/), until I reach the
same level as I am writing Elixir or Python.

I also found [Tsoding](https://www.youtube.com/watch?v=h_D4P-KRNKs&list=PLguYJK7ydFE4aS8fq4D6DqjF6qsysxTnx&index=1)
on youtube and learned how to get started. Recording some notes below:

## interact
```haskell
ghci> :t interact
interact :: (String -> String) -> IO ()
```
This is the single most important function IMO, because a big set back I have when
starting is: how can you do anything real with Haskell? Well `interact` lets you do
just that. Quoting the doc:
> `interact f` takes the entire input from stdin and applies `f` to it.
> The resulting string is written to the stdout device.

This works nicely because you can then write a pure function `solve`, and
compose it with the I/O logic.

Let's look at an example taken from the CSES problem Missing Number:
```
You are given all numbers between 1,2,...,n except one.
Your task is to find the missing number.

Input:
5
2 3 1 5

Output:
4
```
In this case the core logic really is a function `solve :: [Int] -> Int`. But we
still need to handle the I/O.

```haskell
import Data.List (find)
import Data.Maybe (fromJust)
 
solve :: [Int] -> Int
solve l = fromJust $ find (`notElem` l) [1..]
 
main = interact $ show.solve.map read.tail.words
```

Lets break down the `show.solve.map read.tail.words` bit.

## Composing functions
The `.` function is called pointfree programming for some
confusing reasons. I was really afraid to use it but hopefully the below can make
the usage clear.

```haskell
ghci> words "5\n2 3 1 5\n"
["5","2","3","1","5"]

ghci> tail (words "5\n2 3 1 5\n")
["2","3","1","5"]

ghci> map read (tail (words "5\n2 3 1 5\n")) :: [Int]
[2,3,1,5]
```

We want to get rid of all the parenthese. In Elixir (or bash) we would do something
like this:
```elixir
"5\n2 3 1 5\n"
|> words
|> tail
|> map read
|> solve
|> show
```

In Haskell we do it from right to left, hence `show.solve.map read.tail.words`
