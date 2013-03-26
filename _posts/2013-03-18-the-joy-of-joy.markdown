---
layout: post
title: "The Joy of Joy"
date: 2013-03-18 06:21
comments: true
categories: [Joy]
---

**Intro**: `[swap  dip  dup  dip  pop]  dip  dup  dip  pop`  
The above should be read out loud accompanied by a Jazz trio, or maybe you can even go a cappella with it.

Anyways, this is actual Joy source code, taken from the [Mathematical foundations of Joy](http://www.kevinalbrecht.com/code/joy-mirror/j02maf.html) article. No, really, you're not squinting hard enough, there is some actual math there. 

Well, this is my first language from the [Perlis Languages](http://blog.fogus.me/2011/08/14/perlis-languages/) list.  It's a concatenative language, colloquially known as a stack based language (not sure what sort of person goes all colloquial about anything concatenative, probably the sort of person that uses the word "colloquial"). Stack based languages tend to have very little in the sense of syntax, one may even call them "Lisp without the parentheses". So that makes Joy a good candidate to be the first language for this little project, it's rather simple and self contained.  You can pretty much get all the material you might need to tackle Joy from the (mirror of) the [official site](http://www.kevinalbrecht.com/code/joy-mirror/joy.html). Other than that, as recommended by Fogus, there's the really nice ["The Joy of Concatenative Languages" ](http://www.codecommit.com/blog/category/cat) series by [Daniel Spiewak](http://www.codecommit.com), although his language of choice is Cat, it is still, as usual for Daniel's writings, a very instructive and fun read.

Let's take a quick tour of Joy, though for a proper introduction you should consult the [official one](http://www.kevinalbrecht.com/code/joy-mirror/j01tut.html). 
 
<!-- more -->
 
To the eyes of a conventional programmer, the first thing that stands out in a stack based language is the Reverse Polish notation. 
```
3 4 + 5 *
```
(Note that to actually evaluate the snippet in the Joy interpreter you'll need to put a period at the end, as in `3 4 + 5 * .`, but I'll be omitting it here).

The above happens to be 35. And that obviously screams at you with the horrid unnaturalness of the out of order operators. Of course, the average programmer, being to some extent a form of a humanoid, expects this to be written infix as in:
```
3 + 4 * 5
```
Wait, that's actually 23. Of course I meant to write:
```
(3 + 4) * 5
```
Now it's all natural and pleasing to the eye. And that's the last time in the Joy series that you'll be seeing anything written infix, get used to it, sorry...

Actually, there's a whole thing based around RPN having no precedence issues, it allows one to compose programs (functions) by just *concatenating* their sources, no need to worry about any missing parentheses. And that's why you call them *concatenative* languages, the stack is actually an optional implementation detail.  
Having crossed the RPN chasm, we can continue to some Joy basics.

We've already seen number literals and arithmetic operators. We also have the usual boolean values `true` and `false`, string literals are written in double quotes, and characters are prefixed with a single quote. So the following snippet yields `true`:
```
"abcd" 1 at 'b equal
```
Now I'll pretend that you know practically nothing about stacks and explain this snippet step by step. Having done that, I'll assume, that by some miracle, you are now fluent in everything stack, and I wouldn't be bothered to give any more accounts of the source at such level of detail. Ready? Here we go:  
The first word pushes `"abcd"` on the stack, the second `1`. `at` looks at the last two things on the stack and treats the top one as a (zero based) index and the one below it as a string (or a list, we'll get to those in a bit). It then pops them off and leaves on top the character at the indexed position, in our case `'b`. Next we push another `'b` on top of the stack and test the equality of the top two items on the stack with `equal`, popping them off and pushing the result (`true`) on top.

Feeling fluent now? Great, moving on...

Next we have list literals, written in square brackets so this yields 3:
```
[1 2 3 4 5] 2 at
```
Lists are heterogeneous and can contain anything, so this `[1 'c 2 "abc" [1 2 3] 5]` is a valid list. 

Set literals can be written in curly brackets, as in `{1 2 3}`, but they can only contain "small integers", so I didn't really have any use for them further down the road (too bad you can't plug in other implementations into this syntax, oh well...).

There is also a whole array of stack manipulation functions, such as swapping and duplicating items on the stack. So the following yields the square of 4:
```
4 dup *  # => 16
```
(The hash sign designates a comment)

**Interlude**: `[[dup  dip  pop]  dip]  dip  swap  dip  dup  dip  pop`

Now come the interesting bits, the lists introduced above are not plain lists, they are quoted programs (think Lisp lists). This means that the list `[4 dup *]` is actually a value representing the program above, it can be passed around and evaluated at will. Evaluation is accomplished via *combinators*. These take quoted programs as input and use them to calculate new values.  
The simplest combinator is `i`, it just evaluates (unquotes) the quoted program at the top of the stack, like this:
```
[4 dup *] i # => 16
```

Let's look at a slightly more interesting example. The combinator `map` takes a quoted program and applies it to the elements of a list:
```
[1 2 3 4] [dup *] map # => [1 4 9 16]
```
The above is one of the most common use cases for higher order functions, so we got that topic covered.

By now, you might've noticed the conspicuous lack of control structures, guess what, these are also implemented with combinators. The standard `if then else` form looks like:
```
1500 [1000 >]  [2 /]  [3 *]  ifte  # => 750
```
The first list is the condition, which is checked against the top of the stack (in this case 1500). If the condition is met, we apply the second quote to the top of the stack, otherwise the third.  
It takes a little time to get used to this sort of syntax, but after you get it, there's a whole world of flexibility here...

The last type of combinators we'll look at are recursive combinators. These allow one to create recursive functions using anonymous recursion. The most basic recursive combinator is `linrec`, which performs linear recursion. The textbook example for linear recursion is the factorial, implementing it with `linrec` we get
```
[null]  [succ]  [dup pred]  [*]  linrec
```

The first quote is the `if` part, it checks for the base case, here, whether the current argument is 0. The second quote is the `then` part, which is executed when we reach the base case, here, when we reach 0, we take its successor, 1, and leave it on the stack. The last two quotes are the `else1` and `else2` parts. The first is executed before we take the recursive step, in here we duplicate the argument and take the predecessor of the copy. The second quote is executed after the recursion step, in this case we multiply the top two items on the stack. To sum up, we are recursively filling up the stack with the numbers from the given argument down to 1, then, when we back up we are multiplying them in pairs leaving the result on top all the time, until we reach the original value. The last multiplication step gives us the value of the factorial at the top of the stack.

I found combinators to be my biggest stumbling block on the path to readability, when looking at a new combinator, you really haven't a chance of figuring out its purpose unless its name is very obvious or you have its documentation at hand. Take this for example:
```
[1]  [*]  primrec
```
Can you guess what that function does? It happens to be the very same factorial function from before, implemented with a specialized version of `linrec`, it wasn't that obvious to me...  
But as you know, with great flexibility come great code obfuscation powers. But also, great DSL powers, and I really like DSLs, so overall, I'm sure the whole combinators thing works out fine.

There are plenty more useful functions and combinators, you can see the whole standard library [here](http://www.kevinalbrecht.com/code/joy-mirror/html-manual.html).

The last bit of syntax I left out are definitions, you can actually give names to stuff in Joy. So if we wanted to name our square and factorial functions we could do it like so:
```
DEFINE 
   square == dup *;
   factorial == [1]  [*]  primrec.
```
And use it like so:
```
4 square # => 16
5 factorial # => 120
```
There are also some modularization/information hiding facilities built in, but I won't be using them here, so moving right along.

This sums up my not so brief but definitely incomplete introduction to Joy. The most glaring thing I glossed over are the mathematical foundations of Joy. It so happens that Joy is a function level programming language, in the sense that there are no values in the language, just functions. No, `42` is not a value, `42` is a function that takes a stack and returns a new stack with the value `42` on top. So without noticing it, all along we've been composing functions and not just that, but we've been using point-free style while at it.  
Although I didn't delve deep into the maths myself, its presence definitely makes me feel better, I like it when a language is well thought through, such elegance can only come from math.

All together now:  
**Outro**: `[dup dip pop]  dip  dup  dip  pop`