---
layout: post
title: "Joy No More"
date: 2014-03-24 03:46:15 +0200
comments: true
categories: [Joy, stack-based, concatenative]
---

I've reached the end of my Joy escapade. I can definitely say that I had a great time (notice how I avoided one of my well trodden puns here) figuring out this nifty little language.

<!-- more -->

Joy is a demonstration of how far a powerful and clean idea can take you. Being a (sadly) abandoned research language it's definitely not as polished as some other industrial grade alternatives, say, [Factor](http://factorcode.org/), but its elegance cannot be disputed. It's a real eye opener to see how easily concatenative code can be reasoned about by just breaking it down in an arbitrary place, and expanding any non-obvious expressions, then rinse and repeat, till you figure out whatever required figuring out. It takes time to get used to the (theoretically optional) stack, and I can't really say it grows on you, at least not during the short period I've been exposed to it. But once you get your handle on it and reach the right level of abstraction for your domain, it kind of fades more into the background.

On the other side of the feature-set spectrum, there is the power of homoiconicity and quoting. Which was plentifully used and abused in all of the Joy posts, but especially in the [metaprogramming post]({% post_url 2013-05-24-meta-joy %}) (partially undermining the ease of reasoning point above). These minimal features give you plenty of rope to hang yourself, but the gained flexibility is way more than worth it. The experience of [writing a DSL]({% post_url 2014-03-20-domain-specific-joy %}) in Joy is one proof of it.

Joy does have its warts, mostly due to its lack of proper tooling; debugging non-trivial code without even line numbers for the errors and with possible bugs in the interpreter is not a great source of joy (see what I did there? Couldn't help it). But you get over it, and the relative ease of reasoning, at least up to the point when you go too meta, alleviates some of the pain.

Nonetheless, I still recommend learning Joy for both fun and profit (not in the monetary sense, we're all idealists here...); it's a very enlightening experience for a very small entrance fee. Maybe in the future I'll get myself into some more concatenative programming, there's plenty more to dig into. I already mentioned Factor as a more feature complete alternative. But being more inclined towards static typing, the [Cat](http://www.cat-language.com/) programming language looks like an appealing option.

Anyways, that's all I have to say about Joy in this series, it's time to choose my next language.