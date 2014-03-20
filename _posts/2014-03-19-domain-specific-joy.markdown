---
layout: post
title: "Domain-specific Joy"
date: 2014-03-19 05:10:59 +0200
comments: true
categories: [Joy, stack-based, concatenative, state machine, domain-specific language, DSL]
---

In this post our goal is to create a domain-specific language (DSL) for the state machines domain that we introduced in the last [post]({% post_url 2014-03-19-state-of-joy %}). We already had a working example for a state machine, the totally realistic model of baby, which suffered from being rather verbose and far from cleanly expressing our target domain.

To actually have a language, we need some kind of syntax for it. In our case we already have a nice textual representation of the domain, which is the [ASCII table]({% post_url 2014-03-19-state-of-joy %}#baby-table) format I used to describe the different baby states (funny coincidence how that worked out so comfortably well). This format solves the repetitiveness issues and is definitely close to our domain. 

<!-- more -->

To simplify the thought process, let's have a smaller example of a state machine:

<pre>
  || a | b |
============
1 || b | a |
2 || b | b |
</pre>

This time we have only two states (`a`, `b`) and two inputs (`1`, `2`), and we don't use the stack for anything. Simple enough for our purposes here.

Now, Joy's syntax, or lack thereof, makes it possible to write this thing as is by just quoting it all into a list:

<pre>
[ || a | b |
============
1 || b | a |
2 || b | b | ]
</pre>

Or how Joy really sees it:

```
[|| a | b | ============ 1 || b | a | 2 | | b | b |]
```


This works because quotes are not checked for invalid identifiers, they only need to contain valid Joy syntax, and that's exactly what we have in our table (which is yet another amazing coincidence). As a next step we can try to parse the contents of the quote and convert it into something that is runnable with our state machine runner. But that's not a direction I would actually want to pursue. By doing this, we are not taking any advantage of Joy itself for language representation, we are just using it as a fancy tokenizer for what is essentially plain text. And that's practically an external DSL. I find external DSLs much less interesting than embedded DSLs, as they give us less chances to flex our language's syntax muscles. Let us come up with a proper embedded DSL.

As a first step in this direction, we can remove all the fluff from the table above, and convert it to a plain list of lists:

```
[ [a b] [1 b a] [2 b b] ]
```

This is a valid representation of the data from before, but the grouping is wrong. When actually writing the states [before]({% post_url 2014-03-19-state-of-joy %}#baby-states), each state grouped all the possible inputs, what we have here is the opposite. By transposing the table we get the right grouping:

```
[ [1 2] [a b b] [b a b] ]
```
 Which corresponds to the transposed table:
 
<pre>
  || 1 | 2 |
============
a || b | b |
b || a | b |
</pre>

It's not that complicated to write a transposing function to achieve this transformation, but for simplicity, let us just work with the transposed tables to begin with.

Having an abstract way to represent the required data is quite a common thing when creating a DSL. From here we have two directions we need to proceed with. The first, is creating some nice syntax for constructing the data The second, converting the data into something executable.

Before we can proceed with either of those, we still have some aspects of our domain unattended: we don't have a way to do anything on the stack. We'll take the following approach, each cell is going to be implicitly executing on the current stack, so everything we leave there stays on the stack. The last thing on the stack is going to be the next state. We can embellish our simple state machine with some stack actions like so:

<pre>
  ||   1   |    2  |
====================
a ||  0  b | pop b |
b || pop a |  0  b |
</pre>

Now, in the `a` state when the input is `1` it pushes `0` on the stack and moves to `b`, and when the input is `2` it pops a value from the stack and also moves to `b`. The `b` state acts analogously. Converting this into our list representation gives us:
```
[ [1 2] [ a [0  b] [pop b] ] [ b [pop a] [0  b] ] ]
```

To group all of its action into a single piece of data, each cell has to be quoted now.

Because we are dealing with a very simple data structure, creating a palatable syntax for it shouldn't be too complicated, so we'll get to it first. Without further ado, here's the syntax:

<pre>
    @     1    |    2    |
[a] : [  0  b ]|[ pop b ]|
[b] : [ pop a ]|[  0  b ]|
</pre>

(the code for the DSL can be found in [state_machine_table_dsl.joy](https://github.com/ncreep/language_perils/blob/master/Joy/state_machine/state_machine_table_dsl.joy))

All of it is completely valid Joy syntax and creates the following list:

```
[[[0 b] [pop a] "b"] [[pop b] [0 b] "a"] [2 1]]
```

Oops, it seems our lists are inverted. No biggie, it still represents our data just as well. It just happens to be that it's easier to construct things in reverse when dealing with cons lists. 

Let's delve into our syntax a bit further. The `@` pushes our initial value onto the stack, which is a `[[]]`, all further actions will be adding stuff to this value. The `|` symbol prepends the current thing on the stack to our nested list, e.g.:
```
@ 1 |     # => [ [1] ]
@ 1 | 2 | # => [ [2 1] ]
```

The names of the states are now quoted, otherwise we won't be able to put them on the stack as we are doing while constructing the table (once they are on the stack they get evaluated, but the state names don't have any meaning, so that just gets us an error, possibly a silent one).

The `:` starts a new row, it converts the preceding symbol into a string and adds it to a new list in our nested list:

```
[[2 1]] [a] :   # => [ ["a"] [2 1] ]
```

Converting the symbol into a string makes it a bit simpler to work with it further down the road. Notice that the references to the state names in the cells remain as is, we'll have to deal with those later.

And that's all there is to it. This syntax is close enough to the ASCII table format we started with, and contains very little noise. This was the easy bit, now we actually have to turn this into something we can execute.

I'll spare you the gory details of the transformation and focus on the more interesting bits. Having obtained our abstract representation we convert it, using some not too complicated list manipulation code (the beauty of homoiconicity...), into the following construct:
```
[
  ["b" 
    [[
      [[input 2 =] [cur-stack [0 b]   against-stack] i] 
      [[input 1 =] [cur-stack [pop a] against-stack] i]
    ] condn]] 
  ["a" 
    [[
      [[input 2 =] [cur-stack [pop b] against-stack] i] 
      [[input 1 =] [cur-stack [0 b]   against-stack] i]
    ] condn]]
]
```

We can see a strong semblance to the sort of code we were writing to explicitly define a state machine. There are a couple of things to note here. First, we don't have any top-level definitions for the states, everything is contained in a single list. This means that the `a` and `b` references in the code don't actually have any meaning, so we are yet to obtain properly executable code. The second thing to note is what happened to the code inside the cells. Each piece of code got placed in its appropriate execution branch wrapped around in some mysterious incantations. This warrants a closer look.

The last problem that we had with manually constructing state machines, is the lack of transparency when dealing with the stack; each function required special wrapping to be run against the stack. The `against-stack` function takes care of this problem. It uses the `infra` combinator (see [manual](http://www.kevinalbrecht.com/code/joy-mirror/html-manual.html)) to run our code against the list we are currently using as our stack. So the code we are executing can treat our artificial stack as the regular Joy stack, hence no special wrapping for the functions that we use in the cells is required. When execution is done, `against-stack` takes the result and splits out the next state from the rest of the stack, producing both the new state and the new stack.

The line `[cur-stack [0 b] against-stack] i` can be read as "take the current value of the stack, the code `0 b` and run the code against the stack", the `i` actually forces this code to be executed when we reach the branch.

To make our list executable, we need to to somehow take care of the undefined state references. A first guess would be to just find and replace each reference with its explicit form as it is presented in the code above. The problem with this approach is that the states are mutually recursive. So the replacement process will go into an infinite loop. We didn't have this problem when explicitly defining the states, because we used top-level definitions, which can be mutually recursive, but that's not possible in this case, as we can't dynamically add top-level definitions. What we need to do is to somehow delay the evaluation of these references until they are actually needed.

For this purpose we are going to introduce an execution environment. The environment will contain a map where every entry is a state name with an unexpanded definition of that state, and so every reference to a state can be replaced by a call into the map, fetching the right state. For this to actually work, we first need to do this replacement, yielding:

```
[
  ["b" 
    [[
      [[input 2 =] [cur-stack [0 "b" states-map swap find-in-map]   against-stack] i] 
      [[input 1 =] [cur-stack [pop "a" states-map swap find-in-map] against-stack] i]
    ] condn]] 
  ["a" 
    [[
      [[input 2 =] [cur-stack [pop "b" states-map swap find-in-map] against-stack] i] 
      [[input 1 =] [cur-stack [0 "b" states-map swap find-in-map]   against-stack] i]
    ] condn]]
]
```

The line `"b" states-map swap find-in-map` replaced the state `b`, which is just a fancy way of saying "look for the name `"b"` in the `states-map` variable". The `states-map` variable will be our environment when we execute the state machine.

To run our state machine we'll write a function called `run-state-from-stack-with-env` similar to the `run-state-from-stack` we defined before. The main difference is that we now have an additional argument for our environment. To achieve using the environment when actually executing the state transition function we use the `splice-from-map` function that we've developed in the [metaprogramming]({% post_url 2013-05-24-meta-joy %}) post, it performs the appropriate "find and replace" every time we stumble upon a `states-map` reference.

That, essentially, concludes everything we need to define and run our state machines. But, there's a minor wrinkle about the process as described above, and that's that in the current scheme of finding and replacing state references we may miss any such references that are hidden in other function definitions. For example, if our state machine was:

```
func == pop a

    @      1   |     2   |
[a] : [  0  b ]|[ pop b ]|
[b] : [  func ]|[  0  b ]|
```

We'd miss the `a` reference hidden inside `func`. To avoid this issue, when expanding the state machine, we also recursively expand any user defined symbols with the `expand-user-syms` function, this way all state references are exposed when we do the "find and replace" routine described above.

Okay, now we are ready to rewrite the baby state machine (the full code is in [baby_state_machine_dsl.joy](https://github.com/ncreep/language_perils/blob/master/Joy/state_machine/baby_state_machine_dsl.joy)):
```
# incrementing/decrementing the number of parent mistakes
wrong == succ
right == pred


# the baby decides whether it needs to call a social worker
should-call-sw == [max-mistakes >=] [call-sw] [wrong crying] ifte

# 'call-sw' stands for "call social worker"

baby == 
          @   "sing-lullaby"   |      "feed"      |    "soothe"    |
 [sleepy] : [  right asleep   ]|[  wrong crying  ]|[ wrong crying ]|
 [hungry] : [  wrong crying   ]|[  right asleep  ]|[ wrong crying ]|
 [asleep] : [  wrong crying   ]|[  wrong crying  ]|[ wrong crying ]|
 [crying] : [ should-call-sw  ]|[ should-call-sw ]|[ right asleep ]|
[call-sw] : [    call-sw      ]|[    call-sw     ]|[   call-sw    ]|
```

Apart from being transposed, this closely follows the table definition of the table state machine, so we don't have the redundancies in the description that we had in the explicit version. As you can see, the stack manipulation functions do not require any special wrapping. Hence we've covered all of our pain points from before.

Because DSLs are all about their visual flare, we can add a bit of garnish to this definition. We define the following no-ops which can be used as delimiters:
```
, == id
? == id
```

This yields our final version of the state machine:

```
baby == 
          @    "sing-lullaby"   |       "feed"      |    "soothe"     |
 [sleepy] : [  right, asleep   ]|[  wrong, crying  ]|[ wrong, crying ]|
 [hungry] : [  wrong, crying   ]|[  right, asleep  ]|[ wrong, crying ]|
 [asleep] : [  wrong, crying   ]|[  wrong, crying  ]|[ wrong, crying ]|
 [crying] : [ should-call-sw?  ]|[ should-call-sw? ]|[ right, asleep ]|
[call-sw] : [    call-sw       ]|[     call-sw     ]|[    call-sw    ]|
```

Which I find to be slightly more visually pleasing.

We can run the state machine as before:

```
["soothe"] [sleepy] run-baby-table # => [crying]
["soothe" "feed" "sing-lullaby" "feed"] [sleepy] run-baby-table # => [call-sw]
```

I think this can be declared as a success, and with an elated feeling of self-satisfaction we can call it quits now.
 
This concludes our excursion into the world of DSLs. As we experienced at firsthand, developing a DSL is a lot like developing a real language. We have the actual syntax, we "compile" it into a an intermediate representation,  then "compile" it further down into executable code. And even ran it with a non-trivial environment. If that's not your definition of "fun", I don't know what is.

Using Joy for this purpose made the experience very lightweight, the whole definition of the DSL is about 100 lines of code. Minimal syntax and the corresponding homoiconicity, make a language very amenable to embedding DSLs into it. I think our little exercise definitely demonstrates this point.

This also concludes my excursion into Joy, in the [next post]({% post_url 2014-03-20-end-of-joy %}) I'll sum up the experience.