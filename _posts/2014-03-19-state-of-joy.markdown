---
layout: post
title: "State of Joy"
date: 2014-03-19 01:54:19 +0200
comments: true
categories: [Joy, stack-based, concatenative, state machine]
---

Having finished with [binary trees]({% post_url 2013-04-21-joyous-tree-friends %}) and getting somewhat sidetracked by [metaprogramming]({% post_url 2013-05-24-meta-joy %}), we are now ready for our next small project. Implementing binary trees was sort of a warm-up for getting used to Joy, now I would like to do something a bit less textbooky. 

As you may recall, Joy has a very flexible syntax, stemming from the fact of its homoiconicity (as can be seen in the metaprogramming post). Having a flexible syntax should make a language easily amenable to the embedding of [domain-specific languages](http://en.wikipedia.org/wiki/Domain-specific_language) (DSLs). Personally, I'm a big fan of DSLs, so trying to implement one in Joy seems like a nice idea for a small project.

In order to actually have a domain-specific language, we first need to come up with a domain. Having gone through a careful process of examining and comparing various domains (i.e. choosing the first thing that randomly came up in my mind at some distant point in time), I decided to model [state machines](http://en.wikipedia.org/wiki/State_machine). More specifically, I'll be implementing a small language that can describe state machines augmented by a stack, which makes it is a [pushdown automaton](http://en.wikipedia.org/wiki/Pushdown_automaton). Adding a stack to our machine seemed like a natural idea for a stack-based language and also makes the domain a tad more interesting, we'll see how that works out later on.

<!-- more -->

(For the more pedantically inclined, what we'll implement will actually have access to the whole stack and to the whole of Joy's stack manipulation power, so strictly speaking it's not going to be a pushdown automaton, but something more powerful, but we won't linger on that point.)

Before we get to our state machine DSL, we'll write some code that can evaluate state machines, that'll give us a better understanding of our domain; let us get to that. 

A state machine is simply a list of states with their corresponding transition functions. The transition functions take the current input and based on its value and the current stack, decide what is the next state. In our implementation a state will be a list pair, where the first item is the value of the state, e.g. its name, and the second is the state transition function. To make this more obvious in the code, we'll define their corresponding accessors:

```
state-value == 0 at # fetch the state value
next-state == 1 at # fetch the transition function
```

(All code for this post can be found in [this folder](https://github.com/ncreep/language_perils/tree/master/Joy/state_machine), the main code for the state machine implementation is in [state_machine.joy](https://github.com/ncreep/language_perils/blob/master/Joy/state_machine/state_machine.joy))

The core part of our state machine evaluation process is this function:

```
# runs a state against an input list: input state stack -> final-state-value
run-state-from-stack == 
  swap # arguments are now: input stack state
  [finish-state-run] [popd popd state-value] [
    [move-first] dip
    next-state i # input [stack value] -> input new-stack new-state
  ] tailrec;
```

It takes a list of inputs, an initial state and a stack (which is just a list which we'll treat as a stack). It then runs the state machine against the inputs while maintaining the stack in the background. The whole thing works by recursively evaluating the current transition function against the current input and stack; the recursion is anonymous using the `tailrec` combinator. Let's break down the definition:

* We swap the arguments to have them in the right order for our recursive step
* We check whether we are done (i.e. there's no more input) with the `finish-state-run` function
* If so, we throw out all our arguments and pick out the current state value
* Otherwise, we pair up our current input (the first item in the input list) and the current stack using the `move-first` function
* Using the `i` combinator we evaluate the transition function against the input/stack pair
* The output of the transition function is the next state, at this point we recurse and start all over

And that's all we need to evaluate a state machine. To see that it actually works, let's code up a concrete state machine. Apart form actually demonstrating what we've done thus far, this will also be helpful in the design of our DSL. It may so happen that what we already have is close enough to our domain and we won't have a need for a special language. That being very unlikely, at the very least we may get some pointers to what aspects of our syntax we should optimize to get closer to the domain.

*(cue the narrator)*

Family is important, and as programmers we should strive to help other programmers deal with realistic scenarios that come up in family life. In this installment of our "Family for Geeks", we'll see how to deal with babies.

*(fading out with happy jingle music)*

So, we'll implement a state machine that totally realistically models the behavior of a typical baby. Our baby has a number of possible states: sleepy, hungry, asleep and crying. Obviously, the last one is the most common. Our main goal is to choose the right action to get the baby to fall asleep. We have a number of actions that we can do with the baby, these are the inputs to the state machine: sing a lullaby, feed and soothe. To make this more realistic, our baby is going to ~~be vindictive~~ have a memory. The baby is going to keep count for every time we mess up and choose the wrong action. Once we've messed up too many times, the baby is going to call a social worker. Calling a social worker will be counted as yet another state of a baby. Let us show our baby's behavior using a table:

<pre id="baby-table">
             || sleepy | hungry | asleep |          crying            | call social worker |
============================================================================================
sing lullaby || asleep | crying | crying | should call social worker? | call social worker |
        feed || crying | asleep | crying | should call social worker? | call social worker |
      soothe || crying | crying | crying |          asleep            | call social worker |
</pre>

The header row contains the possible states of the baby; the first column has the possible actions. For every state/action pair we choose the next state of the baby. Getting the baby into the crying state is considered wrong, and we'll use the stack to keep track of the number of wrongs. If we get the baby into the asleep state, we get to decrement our wrongs count. Once the baby is in the crying state, we check whether the baby was wronged too many times ("should call social worker?"), if so we move to the call social worker state (which is pretty much game over in this state machine). Otherwise, we keep on crying.

This sums up the behavior of a typical baby in a completely life-like fashion. Now, we can try and write it down as real code (which can be found in [baby_state_machine.joy](https://github.com/ncreep/language_perils/blob/master/Joy/state_machine/baby_state_machine.joy)):

<div id="baby-states">
```
sleepy == ["sleepy" [[
  [["sing-lullaby" input-is] right asleep]
  [["feed" input-is] wrong crying]
  [["soothe" input-is] wrong crying]
] condn]];
  
hungry == ["hungry" [[
  [["sing-lullaby" input-is] wrong crying]
  [["feed" input-is] right asleep]
  [["soothe" input-is] wrong crying]
] condn]];
  
asleep == ["asleep" [[
  [["sing-lullaby" input-is] wrong crying]
  [["feed" input-is] wrong crying]
  [["soothe" input-is] wrong crying]
] condn]];
  
crying == ["crying" [[
  [["sing-lullaby" input-is] should-call-social-worker]
  [["feed" input-is] should-call-social-worker]
  [["soothe" input-is] right asleep]
] condn]];

call-social-worker == ["call-social-worker" [cur-stack call-social-worker]];
```
</div>

Well, that sucked... If you squint hard enough, you may recognize our state machine from before, but there's a whole lot of noise obscuring it from us. 

As you can see, every state gets it's own top-level definition. The first bit of the state is just its name as a string. Next, comes the state transition function in the form of a conditional `condn`, which is just like the built-in `cond` but does not require a default case. Every line corresponds to one possible input. The `input-is` function checks the current input against the string, if it matches we execute the following code. Because our input is part of an input/stack pair, the `input-is` function has to break it down to fetch out the input from the pair and only then compare it to the input in the current branch. 

The code that we execute after choosing a branch works against the input/stack pair and must produce a new stack value and a new state as a result. In most cases we need to decide whether the action is wrong or right and increment/decrement the stack, then we choose the next state. For this purpose we use one of either `wrong` or `right`:
```
wrong == [succ] on-stack;
right == [pred] on-stack;

# [stack value] func -> new-stack
on-stack  == swap cur-stack uncons [swap i] dip cons;
```

The `on-stack` combinator takes a piece of code and executes it against the top value of the current stack. It takes care of splitting out the stack from the input/stack pair (using `cur-stack`) and applying a function against its top value. The `wrong` and `right` functions just pass the `succ` (increment) or `pred` (decrement) to the `on-stack` combinator to achieve the required effect.

For example, the line `[["feed" input-is] wrong crying]` checks whether the input is `feed`, if so, the action was wrong and we increment the counter and move on to the `crying` state.

In the case where we need to check whether we should call a social worker we invoke `should-call-social-worker`:
```
should-call-social-worker == 
  [cur-stack first max-mistakes >=] 
  [cur-stack call-social-worker] 
  [wrong crying] ifte
```

This checks whether the counter on the top of the stack went past our mistakes limit, if so it leaves the stack as is (`cur-stack`) and chooses the `call-social-worker` state, otherwise we invoke the `wrong` function and stay on the `crying` state.

The `call-social-worker` state is trivial, and just keeps the stack as is and remains in the same state.

We can now spot some patterns of repetitiveness that, hopefully, our DSL will be able to eradicate. Firstly, we are repeating our state names both as their definition name and their string name in the state value. Secondly, the list of possible inputs is repeated in almost all of the states. Lastly, manipulating the stack is rather explicit, every function we want to invoke on the stack has to be wrapped in an `on-stack` combinator, and even if we don't need anything on the stack at all, we still have to mention it with the `cur-stack` function. You'd expect to be able to achieve something more transparent than that from a stack-based language.

All that aside, we can actually execute our state machine with the following code:

```
["soothe"] sleepy run-baby-state # => "crying"
["soothe" "feed" "sing-lullaby" "feed"] sleepy run-baby-state # => "call-social-worker"
```

The first argument is the inputs list, the second is the state we start from.  
So at least it works as expected.

This concludes the exposition of the domain, in the next [installment]({% post_url 2014-03-19-domain-specific-joy %}) we'll try to come up with an actual language specific to it.