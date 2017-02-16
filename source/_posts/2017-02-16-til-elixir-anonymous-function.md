---
layout: post
title: "#til Elixir's anonymous function"
date: 2017-02-16 20:46:04 -0800
comments: true
categories:
---

fn is a keyword that creates an anonymous function.

    sum = fn(a,b) ->
      a+b
    end

    IO.puts sum.(1,2)

Invoke using . (dot) similar to Ruby's #call.
Note the dot is not used for named function calls.

    fb = fn
      {0, 0, c} -> "FizzBuzz"
      {0, b, c} -> "Fizz"
      {a, 0, c} -> "Buzz"
      {a, b, c} -> c
    end

A single function with multiple implementations
depending on the arguments passed. Elixir does not have assignments - yes you heard it right.
Instead, it uses pattern matching then it binds the variable to the value.
Note each implementation should have the same number of arguments.

    IO.puts fb.({0,0,3})
    # FizzBuzz
    IO.puts fb.({0,2,3})
    # Fizz
    IO.puts fb.({1,0,3})
    # Buzz
    IO.puts fb.({1,2,3})
    # 3

Since it is just a function, why not define it inside another function.

    fizzbuzz = fn(n) ->
      fb = fn
        {0, 0, c} -> "FizzBuzz"
        {0, b, c} -> "Fizz"
        {a, 0, c} -> "Buzz"
        {a, b, c} -> c
      end

      fb.({rem(n,3), rem(n,5), n})
    end

    IO.puts fizzbuzz.(10)
    IO.puts fizzbuzz.(11)
    IO.puts fizzbuzz.(12)
    IO.puts fizzbuzz.(13)
    IO.puts fizzbuzz.(14)
    IO.puts fizzbuzz.(15)
    IO.puts fizzbuzz.(16)


We now have a baseline [FizzBuzz](http://wiki.c2.com/?FizzBuzzTest) solution with no conditional logic.
Note `rem(a,b)` is an operator that returns the remainder after dividing a by b. The final solution uses
the [Enum module](https://hexdocs.pm/elixir/Enum.html).

    Enum.each(1..100, fn(n) -> IO.puts fizzbuzz.(n) end)

I've read the `|>` [pipe operator](https://elixirschool.com/lessons/basics/pipe-operator/) before, so why not use it. The pipe operator simply takes the result
of one expression and passes it as the 1st parameter to the next expression.

    (1..100) |> Enum.each(fn(n) -> IO.puts fizzbuzz.(n) end)

