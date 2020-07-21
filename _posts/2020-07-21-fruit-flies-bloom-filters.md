---
layout: post
title:  "Fruit Flies and Bloom Filters"
date:   2020-07-21 12:30:00 +0800
tags:   fruit fly bloom filter
---

A few months ago, while reading the [Wikipedia page on Bloom filters][wikipedia-bloom-filter], a friend and I came across a note on fruit flies:

> Fruit flies use a modified version of Bloom filters to detect novelty of odors, with additional features including similarity of novel odor to that of previously experienced examples, and time elapsed since previous experience of the same odor.

We found it very interesting. So here's a short discussion on the subject, and how Bloom filters and fruit flies relate to each other.

## Bloom Filters

For context, a Bloom filter is a classic space-efficient data structure that helps you determine if an item is a member of a set. It is, however, probabilistic: if the membership test says no, the item is definitely not in the set; but if the test says yes, it is only _very likely_ to be in the set.

Bloom filters start off as an empty bit array. To add an item, we pass it through several hash functions, each of which map to a location in the bit array; then, we set each location's bit to 1. To test if the item exists in the Bloom filter, we simply run it through the same hash functions; if all of the locations' bits are set to 1, then it has likely been previously added to the data structure. Otherwise, if any of the bits are 0, then the item must not have been added.

![Bloom filter data structure][bloom-filter-data-structure]

In the above, there are three hash functions. To add _x_, _y_, and _z_, we pass each of them through all three hash functions to get three locations each, and set these locations in the bit array to 1. To test if _w_ is a member, we again pass it through all three hash functions; since one of the locations has a 0 bit, we conclude that _w_ has not been added.

As we add new items to the filter, hash collisions may arise, affecting the accuracy of our membership tests. The probability of such a collision depends on the array size and the number of hash functions used; therefore, designing a Bloom filter involves trade-offs between space, speed (proportional to the number of hash functions), and accuracy.

Bloom filters are widely used to determine if an item has been encountered before. For example, [OkCupid][okcupid-bloom-filter] uses it to track whether a user has already swiped someone, and [Medium][medium-bloom-filter] uses it to avoid recommending its users previously-read articles.

## Fruit Flies

As for fruit flies, they face a problem known as novelty detection: when a fly smells an odor, it needs to determine if it is new, so as to pay greater attention to novel stimuli. This bears a strong resemblance to the membership problem we've seen earlier, which can be solved with Bloom filters.

Indeed, according to a [2018 paper][fruit-fly-paper], fruit flies do use a Bloom filter-like data structure in their brains to handle novelty detection. Upon smelling an odor, its corresponding neural signal representing the odor is passed through a hash function of some sort, and stored into the data structure. To test if an odor is new, its signal is simply hashed and compared against the data structure to determine if it is present.

To make the parallels more explicit between fruit flies and Bloom filters, here's a quick rundown:
- Input: odor, represented as a 50-dimensional vector
- Hash function: feedforward neural network
- Hash: firing pattern of 2,000 cells, where only ~5% (100) of them fire

Distinct odors activate a different 5% of these cells, \<TODO\>

## Calculations


## Implications

Two different traits: distance and time-sensitivity
- Similarity/robustness to noise: replacing a (what?) hash function with a (what?) one
- Recency: decay

[medium-bloom-filter]: https://blog.medium.com/what-are-bloom-filters-1ec2a50c68ff
[okcupid-bloom-filter]: https://tech.okcupid.com/swiping-right-on-bloom-filters/
[wikipedia-bloom-filter]: https://en.wikipedia.org/wiki/Bloom_filter
[fruit-fly-paper]: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6304992/

[bloom-filter-data-structure]: /assets/images/fruit-fly-bloom-filter-data-structure.png
