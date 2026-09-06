---
layout: blog
title: A Normal Post
---

I've been looking into [Tempest][tempest]. It's a new PHP framework that is built for modern PHP: type hinting, property accessors, the works. I'm most excited about its approach to discovery. It closely models what I've been building towards in Smolblog's framework, but with infinitely less manual configuration.

[tempest]: https://tempestphp.com

So this past weekend, I started playing around with seeing if I could get Smolblog's core domain model working. And... I ran into a speed bump. It wasn't hard to get around, but I was curious about the reasons behind it. And when I asked about said reasons... well, apparently I wasn't the first.

Which brings me to this blog post. This isn't me trying to die on this hill or even claim some semblance of high ground. I've just been thinking about this _a lot_ since the topic was broached and I wanted to get my thoughts in order. But I especially want to emphasize: **Nothing in here is meant as an insult to others or a statement on the quality of their code.** I have my perspective, and they have theirs, and none of that means we can't work together.

That being said, let's get some context.

## The Umpteenth Rewrite

I've been working on Smolblog for way too long. I [started it as a WordPress plugin][sb-announce], but as I learned more about [SOLID principles][sb-solid] and [modern Object-Oriented software][poodor], I realized I had bigger ambitions. Especially when it felt like what I wanted to do and how I was thinking about content management was running headlong into some of the decisions WordPress made about how _it_ approached things.

[sb-announce]: https://oddevan.com/2019/05/28/introducing-smolblog.html
[sb-solid]: https://oddevan.com/2022/10/23/building-smolblog-separation.html
[poodor]: https://www.poodr.com

So, I made a decision that honestly haunts me to this day: **the core of Smolblog's code would be framework-independent.** I say "haunts" because while it's meant the code I'm writing is—theoretically—incredibly versatile, it also means I can't take advantage of any of the shortcuts that come from using a framework.

That being said, while development has been slow, it's also forced me into some design decisions that I'm happy about:

- I've ended up with a distinction between Value and Service objects: _Values have state but no logic. Services have logic but no state._ It's helped me stay closer to making "single responsibility" objects.
- I haven't had any real input or output with the system, since a user interface is an implementation detail. Which means the only way I know something works is to write a test. So I've got a lot of _cussing_ tests.
- If the code is something that is core to the application, it goes in. If it's an implementation detail, it gets an interface and I move on.

That last point was helped a lot by the existence of [PSR interfaces][psr]. These are recommended standards for pieces of infrastructure that are often found in frameworks. For a concrete example, even though [I wrote my own dependency injection container][sb-container], the core domain model only needs something that implements [PSR-11][psr-11]. Which means, if I want to _finally_ stop building my own stuff I can swap out my container for [the one in Tempest][tempest-container]!

[psr]: https://www.php-fig.org/psr/
[sb-container]: https://oddevan.com/2023/08/31/a-stupidly-simple.html
[psr-11]: https://www.php-fig.org/psr/psr-11
[tempest-container]: https://tempestphp.com/docs/essentials/container

Except... Tempest doesn't implement PSR-11.

## The Other Side Of the Coin

Naturally I asked in the community chat about this, trying to get an idea of the reasoning. Brent, the lead developer/architect/founder? of Tempest pointed me to [an older blog post of his][stitch-psr], with the note that the "long version" (describing how it pertained to Tempest specifically) still needed to be written[^write]. Between that and some of the ensuing conversation, I gathered a few points:

[stitch-psr]: https://stitcher.io/blog/re-on-using-psr-abstractions

[^write]: To be extra clear: as someone who typically writes code five times faster than blog posts, there's no judgement here!

- The design of Tempest's components don't always match up with the PSR specification.
- Having interoperability between frameworks doesn't make sense when code is tied so closely to its framework.
- The interfaces designed by the PHP FIG can be outdated or only fit for a specific purpose.
- For PSR-11 specifically, Brent cites two frameworks, Laravel and Symfony, who both implement the Container standard yet add several methods on top of it. Any real use of the class would inevitably be tied to the specific framework.

I highly encourage you to [read Brent's post][stitcher-psr-2], especially as at the end he acknowledges the popular use case of smaller packages being used in multiple frameworks.

[stitcher-psr-2]: https://stitcher.io/blog/re-on-using-psr-abstractions

It's that particular use case that I end up falling into for Smolblog: the core domain model isn't written as a monolithic application but as a *library* to be used by an application. There's so much more that goes into an app than just the core logic; in fact that's often some of the smallest parts! But in this particular case, I wanted to take the time to get this core logic right... and I had another motivation.

## Fool Me Once

I didn't realize until I went to write this paragraph how ingrained the idea of limiting dependencies is to Smolblog. [The first essays][old-blue] around the idea of Smolblog came in the wake of poorly-implemented policy changes at Tumblr. The original feature set leaned heavily on [POSSE][posse], the IndieWeb idea of owning your own content.

[old-blue]: https://oddevan.com/2018/12/13/what-makes-a.html
[posse]: https://indieweb.org/POSSE

So it makes sense that I wanted to reflect this idea in code. I didn't want to be trapped into a particular framework's idioms and then find out later that it would have been better to pick a different one. Part of this came from [writing a different app with a WordPress backend][grimoire] and realizing... I wasn't getting anything out of WordPress that I couldn't get elsewhere with less hassle.

[grimoire]: https://oddevan.com/2022/04/09/introducing-grimoire.html

So while I do plan on using Tempest—and I have the highest respect for Brent as someone that's _actually doing the thing_ he blogged about!—I wanted to push back against this:

> I'm more than happy to ditch "interoperability" — what does that even mean when you're building a project closely tied to a framework and never intend to change it — and just use a simpler, straight forward, opinionated solution.

You never intend to change it _until you do._ The framework is just what you need _until it isn't._ One thing that I took away from [POODR][poodr-2] was the idea of using dependency injection and interfaces to keep your code reusable. And no one with any serious experience with object-oriented programming disagrees!

[poodr-2]: https://www.poodr.com

What I keep coming back to is [when I finally understood the reasons for dependency injection][sb-solid-2]:

[sb-solid-2]: https://oddevan.com/2022/10/23/building-smolblog-separation.html

> I genuinely think part of the reason SOLID never clicked for me at the agency was the simple fact that we were always going to be using WordPress. [...] Now that I’m working on a project—a big one!—that doesn’t have that constraint, I’m beginning to see the value even if we were staying with WordPress.

If you _know_ your project won't outgrow your framework—perhaps the scope is limited by your contract or ambition—then hack away. Even if you have bigger plans, building a project tightly coupled to a framework is fine if that's what it takes to build the _cussing_ thing! Am I speaking for myself? Yes.

But I was always going use WordPress... [until I wasn't][wp-nope].

[wp-nope]: https://oddevan.com/2024/09/23/matt-mullenweg-recently.html
