---
layout: post
title: "Hello Frink"
date: 2015-01-08 16:54:37 +0200
comments: true
categories: Frink
---

In this post I'll be introducing the [Frink](https://futureboy.us/frinkdocs/) programming language. As stated on Frink's homepage:

>Frink is a practical calculating tool and programming language designed to make physical calculations simple, to help ensure that answers come out right, and to make a tool that's really useful in the real world. It tracks units of measure (feet, meters, kilograms, watts, etc.) through all calculations, allowing you to mix units of measure transparently, and helps you easily verify that your answers make sense. It also contains a large data file of physical quantities, freeing you from having to look them up, and freeing you to make effortless calculations without getting bogged down in the mechanics.

So the "killer feature" of Frink is that you can write something like:
```
12.5 kilotons TNT / (6 years + 9 months) -> horsepower
// 329.48048477017130757
```

Frink is a full featured programming language, but we'll be mostly focusing on this feature and see how that works out.

<!-- more -->

(The full code for this post can be found at [TODO]())

Since Frink's syntax should be pretty intuitive to anyone coming from the world of C-like languages, I'll skip a detailed introduction of the syntax, and we'll just jump right in into a session of exploratory programming with Frink, introducing any new concepts as they come up. (For some distant future readers living in a world where C-like syntax is long forgotten and Javascript is buried in the unreachable depths of the Internet Archive, I envy you.)

First things first, anything Frink related, including documentation and an interpreter, can be found on its [homepage](https://futureboy.us/frinkdocs/). Simple calculations can be run directly on the [web interface](https://futureboy.us/fsp/frink.fsp). Having set up a working environment (note that Frink is very frequently updated, hopefully, the code written here will not get dated too soon), we can move on to the premises of our exploratory session.

Without further ado, the premises: we'll be trying to come up with a totally scientific method of making money ("...using this one weird trick discovered by a mad scientist..."), lots and lots of shiny money. I'll outline the money producing method, and we'll use Frink to perform back-of-the-proverbial-envelope calculations to see how profitable it is (where, in our case, the back of an "envelope" includes a computer and an internet connection). There will be some code soon, bear with me.

It's common practice to make money from currency exchange, which can be thought of as taking advantage of geography to make money. Looking at from a slightly different angle, we are using some form of spatial relationships to produce money. Since in physics there isn't much difference between space and time, why can't we use temporal relationships to produce money? "Ah", you might say, "he must mean stock trading". But not, that's too well trodden a path to take, and it's not that interesting anyways. What I propose, is to use time travel, specifically time travel to the past, to generate money. A naive approach might include using current stock market information to make money in the past, but I don't see it as a viable approach, since if you're too successful you're likely to get caught and be sent to jail for insider trading or something. And that's not cool at all. So we'll use a more subtle method.

The method relies on the fact that prices of different consumer products vary differently over time. As a reference for historical prices, we'll use this table (adapted from [here](http://www.infoplease.com/ipa/A0873707.html), all prices are given in dollars):

<pre>
| Year | Flour (5 lbs) | Bread (lbs) | Milk (1/2 gal.) | Potatoes (10 lbs) | Coffee (lb) | Sugar (5 lbs) |
|------|---------------|-------------|-----------------|-------------------|-------------|---------------|
| 2010 | 2.36          | 1.41        | 1.66            | 5.79              | 4.16        | 3.11          |
| 1970 | 0.589         | 0.243       | 0.659           | 0.897             | 0.911       | 0.648         |
| 1960 | 0.554         | 0.203       | 0.520           | 0.718             | 0.753       | 0.582         |
| 1950 | 0.491         | 0.143       | 0.412           | 0.461             | 0.794       | 0.487         |
| 1940 | 0.215         | 0.80        | 0.256           | 0.239             | 0.212       | 0.260         |
| 1930 | 0.230         | 0.86        | 0.282           | 0.360             | 0.395       | 0.305         |
| 1920 | 0.405         | 0.115       | 0.334           | 0.630             | 0.470       | 0.970         |
</pre>

As an example of how we can make money, we can do the following:

* For `$5.79`, buy `10` pounds of potatoes today (which, according to the table, is 2010, but that's besides the point)
* Go back to 1920 and sell that potatoes for `$0.63`
* Buy `0.63 / 0.115 = 5.47` pounds of bread
* Return to the present and sell our bread for `$7.7`

Having executed this simple procedure, we just made a hefty profit of `7.7 - 5.79 = 1.91` dollars.

Of course, nothing is that simple, to make this into an actually viable plan, we'll need to optimize the various parameters, to achieve maximum profit.

For starters, let's convert our table into some Frink data structures. Each column in the table will correspond to one array:

```
flourVals = [2.36, 0.589, 0.554, 0.491, 0.215, 0.230, 0.405]
breadVals = [1.41, 0.243, 0.203, 0.143, 0.80, 0.86, 0.115]
milkVals = [1.66, 0.659, 0.520, 0.412, 0.256, 0.282, 0.334]
potatoVals = [5.79, 0.897, 0.718, 0.461, 0.239, 0.360, 0.630]
coffeeVals = [4.16, 0.911, 0.753, 0.794, 0.212, 0.395, 0.470]
sugarVals = [3.11, 0.648, 0.582, 0.487, 0.260, 0.305, 0.970]
```

(There are more civilized methods to import external data into Frink, but that'll do for now)

You might note, these are just plain numbers, given how Frink is all about the physical units, we should probably give these numbers some concrete units. As a first approximation to what we actually need, we could use the units actually provided in the table, but as someone once said:
> If God had wanted us to use the metric system, he would have given us 10 fingers and 10 toes

(Note that the distinguished scientist Frink, who lent his name to the Frink language, actually has 8 fingers, not sure what that implies...)

Luckily, unit conversion is one of Frink's most basic actions. Let's start with flour, the units implied by the table are dollar per 5 lbs, writing this down in the interpreter gives us:
```
dollar / (5 lbs)
// 20000000/45359237 (approx. 0.44092452436975516) kg^-1 dollar (price_per_mass)
```

(Frink recognizes juxtaposition as multiplication)

So no need for conversions, MKS is the (very sane) default. Next, we need to apply the units to each value in the array. Frink has various looping facilities, but we'll use just a standard `map` function:
```
flourWeights = map[{ |x| x dollar / (5 lbs) }, flourVals]]
```

Function application uses square brackets, and `{ |x| ... }` is the syntax for anonymous functions.

We can do the same for the other columns, except for milk, milk comes in volume instead of weight. For the sake of consistency, we'll convert it to weight, and as it happens, Frink knows the value for milk density (as for many other substances), so we can just write:
```
1/2 gal milk // 1.9360000000000000001 kg (mass)
```

And now we have all of our data with proper units. But thinking about what we've done with the milk values, using density instead of actual weight, makes a lot of sense. Though I've little experience with actual time travel, but I would imagine that, just like in space travel, in a real time travel machine (and not those made up ones you see in the movies) the allowed weight and volume on a mission would be seriously constrained. So it would make sense to follow our profit both for weight and for volume.

We can perform the conversion easily enough, though Frink doesn't the densities for all of our products, so we'll have to lookup some of those on our own. Adding the missing values is simple (taken from [here](http://www.fao.org/docrep/017/ap815e/ap815e.pdf)):

```
flour := 0.48 g / ml
bread := 0.29 g / ml
potato := 0.59 g / ml
coffee := 0.25 g / ml
```

(where `:=` defines a new global unit)

Converting our $/weights to $/volumes is a simple as multiplying by the density, e.g. the flour becomes:
```
flourVolumes = map[{ |x| x flour }, flourWeights]
```

And so on for the other columns