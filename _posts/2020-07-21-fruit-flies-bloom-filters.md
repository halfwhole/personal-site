---
layout: post
title:  "Fruit Flies and Bloom Filters"
date:   2020-07-21 12:30:00 +0800
tags:   fruit fly bloom filter
---

A few months ago, while reading the [Wikipedia page on Bloom filters][wikipedia-bloom-filter], a friend and I came across a note on fruit flies:

> Fruit flies use a modified version of Bloom filters to detect novelty of odors, with additional features including similarity of novel odor to that of previously experienced examples, and time elapsed since previous experience of the same odor.

Who knew fruit flies and Bloom filters had anything to do with each other? This was a fun tidbit too interesting to ignore.

<div style="text-align:center; margin-bottom:15px">
  <img src="/assets/images/fruit-fly-bloom-filter/bloom-filter.png" alt="bloom filter data structure" style="display:inline-block; height:160px; margin-right:10px">
  <img src="/assets/images/fruit-fly-bloom-filter/fruit-fly.jpg" alt="fruit fly image" style="display:inline-block; height:200px">
</div>

<div style="text-align:center; font-style:italic; font-size:14px; margin-bottom:25px">
(R) <a href="https://commons.wikimedia.org/wiki/File:Drosophila_melanogaster_-_front_(aka).jpg">Drosophila melanogaster</a> by <a href="https://commons.wikimedia.org/wiki/User:Aka">André Karwath</a>, under <a href="https://creativecommons.org/licenses/by-sa/2.5/">CC BY-SA 2.5</a>
</div>

#### Drawing Parallels

In computer science, we know Bloom filters to be a classic probabilistic data structure used to solve the _membership problem_,
i.e. to determine if an item is a member of a set.
And in practice, Bloom filters are very commonly used to determine if a user has seen some item before:
[OkCupid][okcupid-bloom-filter] uses it to track whether a user has already swiped someone,
and [Medium][medium-bloom-filter] uses it to avoid recommending its users articles that have already been read.

In the natural world, fruit flies face a similar problem known as _novelty detection_:
when it smells an odor, it needs to determine if the odor stimulus is new, so it can pay greater attention to novel ones.
Since novelty detection can be framed as an instance of the membership problem,
it can be solved with Bloom filters too (with a few important modifications!).

As it so happens, according to a [2018 paper][fruit-fly-paper],
fruit flies do have a Bloom filter-like data structure in their brains.
When a fruit fly smells an odor, it is converted into a corresponding neural signal,
which then passes through neural connections that act as a hash function.
Finally, the resulting hashed neural output is stored into the Bloom filter-like structure in its brain.
To test if an odor is new, its corresponding signal is simply hashed and compared against the data structure to determine if it is present.

#### Modifications

There are some slight but important differences between the abstract Bloom filter and the version we see in fruit flies.

For one, the fruit fly's Bloom filter doesn't store 0/1 bits, but something akin to a continuous real value.
This might be less space-efficient in code, as floats may take say 32 or 64 times the space.
But these continuous values serve an important purpose to fruit flies: they represent _how novel_ the odor is.
When the odor is experienced, it bumps up its respective values in the Bloom filter, making it less novel;
as time passes, these values slowly decay, and so old odors 'become' more novel over time.

Another difference would be that of locality-sensitive hashing:
similar odors are hashed to similar locations, and hence share similar novelty values.
This can be disadvantageous in a traditional Bloom filter as it can disrupt uniformity across the data structure.
But for fruit flies, similar odors ought to be recognised similarly and given similar novelty values,
so the use of locality-sensitive hashing is appropriate.

These modifications allow for distance and time-sensitivity, making the fruit fly's version of
the Bloom filter robust to noise (in terms of odor input) and giving it a recency effect.

It would be interesting to see these modifications implemented in regular Bloom filters, as they have some useful applications.
Time-sensitivity can be useful where novelty or recency matters, say if
we want to re-recommend old articles to users after some amount of time.
Distance-sensitivity could help in applications where we want to treat similar inputs similarly,
or when the inputs may be noisy (e.g. image, audio, or video data).

#### Thoughts

These parallels between fruit flies and Bloom filters are a rather quirky fact,
and it reminds us that we can often draw connections between nature and mathematics.
The similarities that come from these parallels may be important,
but so are the distinctions---these inform potentially new applications and areas of research.

Often we think of nature as an inspiration for human innovations and discoveries:
it has given us engineering marvels like the speed rail, the law of gravitation, and so on.
But among others, examples like the fruit fly seem to show that the converse is also true;
sometimes, nature takes a hint from mathematics as well.

<img src="/assets/images/fruit-fly-bloom-filter/aloe-spiral.jpg" alt="aloe spiral" style="display:block; width:40%; margin-left:auto; margin-right:auto"/>

<div style="text-align:center; font-style:italic; font-size:14px">
<a href="https://www.flickr.com/photos/7326810@N08/1636684209">Aloe polyphylla spiral</a> by <a href="https://www.flickr.com/photos/7326810@N08/">Jean</a>, under <a href="https://creativecommons.org/licenses/by/2.0/">CC BY 2.0</a>
</div>

[medium-bloom-filter]: https://blog.medium.com/what-are-bloom-filters-1ec2a50c68ff
[okcupid-bloom-filter]: https://tech.okcupid.com/swiping-right-on-bloom-filters/
[wikipedia-bloom-filter]: https://en.wikipedia.org/wiki/Bloom_filter
[fruit-fly-paper]: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6304992/
[locality-sensitive-bloom-filter-paper]: https://ieeexplore.ieee.org/abstract/document/5928322
