---
layout: post
title: "Hello Frink"
date: 2015-01-08 16:54:37 +0200
comments: true
categories: Frink
---

In this post, I'll be introducing the [Frink](https://futureboy.us/frinkdocs/) programming language. As stated on Frink's homepage:

>Frink is a practical calculating tool and programming language designed to make physical calculations simple, to help ensure that answers come out right, and to make a tool that's really useful in the real world. It tracks units of measure (feet, meters, kilograms, watts, etc.) through all calculations, allowing you to mix units of measure transparently, and helps you easily verify that your answers make sense. It also contains a large data file of physical quantities, freeing you from having to look them up, and freeing you to make effortless calculations without getting bogged down in the mechanics.

So Frink's "killer feature" is that you can write something like:
```
12.5 kilotons TNT / (6 years + 9 months) -> horsepower
// 329.48048477017130757
```

<!-- more -->

Although Frink is a full-featured programming language, including some support for object-oriented and functional programming, in this introductory post we'll be focusing only on Frink's ability to handle units. Since Frink's syntax should be pretty intuitive to anyone coming from the world of C-like languages, in future posts about Frink, I'll introduce new syntax as it comes up. (For some distant future readers living in a world where C-like syntax is long forgotten and Javascript is buried in the unreachable depths of the Internet Archive, if you haven't got a clue what I'm on about, I envy you.)

First things first, anything Frink related, including documentation and an interpreter, can be found on its [homepage](https://futureboy.us/frinkdocs/). Simple calculations can be run directly on the [web interface](https://futureboy.us/fsp/frink.fsp). Having set up a working environment (note that Frink is very frequently updated, hopefully, the code written here won't get dated too soon), we can move on to the premises of our exploratory session.

*<u>Clarification</u>: the author of the text below is not completely out of his mind, despite evidence to the contrary.*

Without further ado, the premises: we'll be trying to come up with a totally scientific method of making money ("...using this one weird trick discovered by a mad scientist..."), lots and lots of shiny money. I'll outline the revenue-generating method, and we'll use Frink to perform back-of-the-proverbial-napkin calculations to see how profitable it is (where, in our case, the "napkin" includes a computer and an internet connection). There will be some code soon, bear with me.

It's common practice to make money from currency exchange, which can be thought of as taking advantage of geography to make money. Looking at it from a slightly different angle, we are using some form of spatial relationships to generate profit. Since in physics there isn't much difference between space and time, why can't we use temporal relationships to earn money? "Ah", you might say, "he must mean stock trading". But no, that's too well-trodden a path to take, and it's not that interesting anyways. What I propose is to use time travel, specifically, time travel to the past, to generate profit. A naive approach might include using current stock market information to make money in the past, but I don't consider it to be a viable approach, since if you're too successful you're likely to get caught and be sent to jail for insider trading or something; and that's not cool at all. Another method would be to go to the past with a bunch of current dollars, and use the dollars' increased buying ability in the past to import some goods into the future. But using money from the future might raise suspicion as well. So we'll use a more subtle approach.

The method relies on the fact that prices of various goods vary differently over time. As a reference for historical prices, we'll use this table (adapted from [here](http://www.infoplease.com/ipa/A0873707.html), all prices are given in dollars):

<pre>
| Year | Bread (lbs) | Potatoes (10 lbs) |
|------|-------------|-------------------|
| 2010 | 1.41        | 5.79              |
| 1920 | 0.115       | 0.630             |
</pre>

So to make money, we can do the following:

* For `$5.79`, buy `10` pounds of potatoes today (which, according to the table, is 2010, but that's beside the point)
* Go back to 1920 and sell the potatoes for `$0.63`
* Buy `0.63 / 0.115 = 5.47` pounds of bread
* Return to the present and sell our bread for `$7.7`

Having executed this simple procedure, we just made a hefty profit of `7.7 - 5.79 = 1.91` dollars.

Of course, nothing is that simple, we should further explore this scenario.
For starters, we should convert our table into some Frink code. We could write something like:

```
breadNow = 1.41
bread1920 = 0.115

potatoesNow = 5.79
potatoes1920 = 0.630

// there are more civilized methods to import external data into Frink
// but that'll do for now
```

You might note that these are just plain numbers; given how Frink is all about the physical units, we should probably give these numbers some concrete units. We could use the units actually provided in the table, but as someone once said:
> If God had wanted us to use the metric system, he would have given us 10 fingers and 10 toes

(Though the distinguished scientist Frink, who lent his name to the Frink language, actually has 8 fingers, not sure what that implies...)

Luckily, unit conversion is deeply ingrained into Frink. Let's start with bread - the units implied by the table are dollar per lbs, writing this down in the interpreter gives us:
```
dollar / lbs
// 100000000/45359237 (approx. 2.2046226218487758) kg^-1 dollar (price_per_mass)
```

So no need for conversions, MKS is the (very sane) default. We can write down the rest of the values in the same manner:
```
// Frink recognizes juxtaposition as multiplication

breadNow = 1.41 dollar / lbs
bread1920 = 0.115 dollar / lbs

potatoesNow = 5.79 dollar / (10 lbs)
potatoes1920 = 0.630 dollar / (10 lbs)
```

And now we have all of our data with proper units. We can describe our money making scheme using Frink, though we'll be trying to rake in more profit now:
```
moneyInvested = tonne potatoesNow
// 1276.4764980504411924 dollar (currency)

moneyFromPotatoes = tonne potatoes1920
// 138.89122517647287585 dollar (currency)

breadBought = moneyFromPotatoes / bread1920
// 547.82608695652173911 kg (mass)

moneyFromBread = breadBought breadNow
// 1702.9271956419717822 dollar (currency)

moneyMade = moneyFromBread - moneyInvested
// 426.4506975915305898 dollar (currency)
```

Ha, it works! We've just made ourselves about 430 dollars, and all it took is to buy a mere tonne of potatoes. Well almost, I'm ignoring the fact that we actually don't have a time machine. Speaking of which, though I've little experience with actual time travel, I would imagine that, just like in space travel, in a real time machine (and not those made-up ones you see in the movies) the allowed weight and volume on a mission would be seriously constrained.

We know how much weight we'll be carrying, but how much volume would that make? Simple enough, we just need the densities of bread and potatoes, which can be found, for example, [here](http://www.fao.org/docrep/017/ap815e/ap815e.pdf):
```
bread = 0.29 g / ml
potato = 0.59 g / ml
```

And so:
```
548 kg / bread // 1.8896551724137931034 m^3 (volume)
tonne / potato // 1.6949152542372881356 m^3 (volume)
```
From which we can conclude that bread is the constraining element here. But these numbers are a bit meaningless; we need to compare them to something familiar. An average fridge is about 20 cubic feet, converting manually is simple enough (I'm using weird American units just to show that Frink doesn't scare easily):
```
(548 kg / bread) / (20 cubic feet)
// 3.3366271316165081822
```

But we don't actually need to bother with manual unit conversion, since Frink has a nifty unit conversion operator: `->`. For example, we can write:
```
1.67 meter -> feet 
// 5.4790026246719160105
```

Adding quotes on the right-hand side gives a bit of nice formatting:
```
1.67 meter -> "feet" 
// 5.4790026246719160105 feet
```

Note that the result is a string value and not a number.

What's even nicer is that the right-hand side can contain an arbitrary numeric expression with compatible units, so we are not limited to just built-in units. Back to our example, we can write it as:

```
548 kg / bread -> 20 cubic feet
// 3.3366271316165081822
```

We can also define our own custom unit for fridge related measurements (which come up quite often, so definitely a worthwhile investment) and have:
```
fridge = 20 cubic feet

548 kg / bread -> "fridge"
// 3.3366271316165081822 fridge
```

Making our 430 dollars would require us to carry 3 fridges full of bread, from 90 years back in the past. Fair enough, nothing comes for free; making a bit of an effort for money builds character, or something.

Let's get more ambitious, we can arrange our retirement with a big score of 1 million dollars (and when I say "our", I mean "my", since it's my plan, I'm just thinking out loud here). Rearranging the equations from the previous excursion to the past, we have:
```
potatoesToBuy = 1 million dollar / (potatoes1920 breadNow / bread1920 - potatoesNow)
potatoesToBuy ->  "tonne"
// 2344.9369543717689376 tonne
```

Whoa, that's a lot of potatoes, let's see how many fridges we'll need:
```
potatoesToBuy / potato -> "fridge"
// 7017.8531378425014657 fridge
```

And it gets worse with the bread we'll have to carry back from the past:
```
breadVolume = potatoesToBuy potatoes1920 / bread1920 / bread
breadVolume -> "fridge"
// 7821.6971854154656513 fridge
```

At these volumes, fridges become a bit meaningless as well; I, personally, have never seen so many fridges in one place (if you did, you can share this bizarre experience in the comments). A [space shuttle fuel tank](http://en.wikipedia.org/wiki/Space_Shuttle_external_tank) is about 2.6 million liters:
```
spaceFuelTank = 2.6 million liters

breadVolume -> "spaceFuelTank"
// 1.7037369176037532356 spaceFuelTank
```

Hmm, 2 space shuttle fuel tanks, that's a lot. Makes me wonder... If a human can be "fueled" by bread or potatoes, can we fuel a space shuttle with them?

```
// food calories, as apposed to regular calories, are written with a capital 'C'
breadCalories = 260 Calories / (100 gram)
potatoCalories = 80 Calories / (100 gram)

// rocket fuel
liquidH2 = 8.5 MJ / L

fullFuelTank = liquidH2 spaceFuelTank

bread breadCalories spaceFuelTank -> "fullFuelTank"
// 0.37139378823529411764 fullFuelTank

potato potatoCalories spaceFuelTank -> "fullFuelTank"
// 0.23249054117647058823 fullFuelTank
```

Not bad, of the two, bread seems to be the better alternative to rocket fuel; although we'll need more than two tanks of bread to compensate for its inefficiency (this is not, hmm, quite accurate, since in an actual fuel tank, not all volume is taken by liquid hydrogen, but, whatever). I won't take any responsibility for the consequences of actually trying to send a space mission on pure bread fuel; I am willing to take credit if it works out, though.

Anyways, we got a bit sidetracked by a somewhat ridiculous idea - back to our more serious business plans. So making one big score and retiring doesn't seem to be in our cards. We'll probably have to devise some working schedule and move our goods in batches; time travel can really become a 9-to-5 job. But still, for now, it's a niche market, I'm sure that profit is guaranteed.

Since it's a 9-to-5 job, we can try to figure out what our daily commute would be like. Judging by various movies, it seems that time travel is nearly instantaneous, but that's ridiculous, one's daily commute can't be that fast (I'm sure there's a law of physics that governs that). I propose a more reasonable rate of half an hour per one year traveled into the past. So traveling ninety years back takes:
```
ninety years (half hour / year) -> "hours"
// 45.0 hours
```

That's quite the commute, but we're aiming for much profit, so that's how it is. 

One small wrinkle though, in the calculations above, we didn't account for the monetary investment in actual time travel. Let's sort this out. For the present discussion, I'll ignore the one-time investment of actually obtaining a time machine; as it's just a minor technical detail, a bit of research and engineering should settle it. Given that we are probably going to make many time trips, I would like to have an estimate of how much a single trip actually costs.

In physics, when one goes between time and space values, one uses the speed of light (`c`) as a conversion factor. We're going to stretch this analogy (till it utterly bursts) to provide some estimates for the energy consumption of a time trip. 

Since we are not actually moving, I think it's reasonable to assume that time travel doesn't have any friction (like air friction when flying). If one day someone discovers some kind of "time friction", we'll have to modify our calculations accordingly. But for now, ignoring friction, we can estimate the energy consumption of our trip just by knowing the difference in the required "velocities". The starting point is that we are moving forwards in time at a rate of one second per second. What we want to achieve, is a movement backwards in time one year per half hour, and of course, we need not forget to convert everything to velocity values using the speed of light:
```
startVelocity = c second / second
targetVelocity = -c year / (half hour)
```

We can plug our "velocities" into the formula for kinetic energy (`m/2 v^2`) and take the difference for our energy consumption. Assuming that the payload is a tonne of potatoes, we have:
```
timeTravelEnergy = tonne/2 (targetVelocity^2 - startVelocity^2)
// 1.3811974908674380205e+28 m^2 s^-2 kg (energy)
```

Which is the energy we need to invest to accelerate back in time. We could similarly ballpark the return trip, but that `e+28` in the result looks really suspicious. Let's try to see what it means:
```
// the atomic bomb dropped on Hiroshima
littleBoy = 15 kilotons TNT

timeTravelEnergy -> "littleBoy"
// 2.1992890208392057905e+14 littleBoy

// not even close...

// a hydrogen bomb, the most powerful nuclear weapon ever detonated
tsarBomba = 50 megatons TNT

timeTravelEnergy -> "tsarBomba"
// 6.5978670625176173713e+10 tsarBomba

// nope

// world energy consumption in the years 1980 - 2012
worldEnergyConsumption = 12676 quadrillion Btu

timeTravelEnergy -> "worldEnergyConsumption"
// 1.0327568857509895368e+6 worldEnergyConsumption

// nah ah

// the asteroid that killed the dinosaurs
chicxulubImpact = 100 teratons TNT

timeTravelEnergy -> "chicxulubImpact"
// 32989.335312588086856 chicxulubImpact

// getting warmer (pun intended)...

energyToEvaporateAllOceans = 3 octillion joules

timeTravelEnergy -> "energyToEvaporateAllOceans"
// 4.6039916362247934016 energyToEvaporateAllOceans
```
(the figures above come from [a](http://en.wikipedia.org/wiki/Little_Boy) [number](http://en.wikipedia.org/wiki/Tsar_Bomba) [of](http://www.eia.gov/cfapps/ipdbproject/IEDIndex3.cfm?tid=44&pid=44&aid=2) [various](http://en.wikipedia.org/wiki/Chicxulub_crater) [sources](http://scientificlogic.blogspot.co.il/2010/04/how-much-heat-is-needed-to-vaporize.html))

Got it! So all we need to power our little business trip is the energy equivalent of evaporating all oceans on Earth, five times. Right. So how does *that* affect our revenue? Assuming we'll be using rocket fuel for our travels:

```
liquidH2Price = 3 dollar / kg
liquidH2SpecificEnergy = 142 MJ / kg

requiredFuel = timeTravelEnergy / liquidH2SpecificEnergy

fuelMoney = requiredFuel liquidH2Price

totalRevenue = moneyMade - fuelMoney
// -2.9180228680297986306e+20 dollar (currency)
```

Well, that sucked...