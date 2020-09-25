---
layout: post
title:  "The Aphrodite Project: Matchmaking and Stable Matching"
date:   2020-09-03 00:00:00 +0800
tags:   aphrodite project stable roommate marriage problem matching love matchmaking dating
---

<script src="/public/js/jquery-1.11.1.min.js"></script>
<script src="/public/js/katex.min.js"></script>
<link rel="stylesheet" href="/public/css/katex.min.css">

There's been a post making the rounds in school lately on love and matchmaking:
[The Aphrodite Project][the-octant], started last year by students from the National University of Singapore,
aims to match students with each other through a questionnaire and an algorithm.
The project has been pretty successful, drawing over 3000 participants last year,
and is slated for its second run later this month.

<div style="text-align:center">
  <img src="/public/images/aphrodite-project/poster.jpg" alt="Aphrodite Project poster" style="display:inline-block; width:250px; margin-bottom:0px">
</div>

<div style="text-align:center; font-style:italic; font-size:16px; margin-bottom:15px">
  Source: <a href="https://mothership.sg/2019/10/dating-project-nus-yale-nus-students/">The Mothership</a>
</div>

First, one fills up a questionnaire made to tease out one's personality, interests, and beliefs.
To then give every person a single match based on their answers, an "Economics Nobel Prize-winning algorithm" is run.
This could only refer to the Gale-Shapley algorithm,
which provides a good solution to matching pairs of people
given their preference rankings for one another.

It seems to me though that there's more here than meets the eye.
Can romance be so easily formulated and solved by an algorithm?
Let's look at the problem of matchmaking and evaluate it from an algorithmic perspective.

### 1. Stable Marriage Problem (SMP)

The Gale-Shapley algorithm has its origins in a [1962 paper][gale-shapley-paper] by David Gale and Lloyd Shapley.
As they were figuring out the problem of pairing colleges and college applicants,
they formulated the Stable Marriage Problem (SMP) and devised the Gale-Shapley algorithm to solve it.

In the SMP, we have two sets of elements, each of equal size <script type="math/tex">n</script>; let's call them "men" and "women" respectively.
Suppose that each man wants to be partnered to a woman and vice versa, and that each person ranks everyone of the opposite sex
from <script type="math/tex">1</script> to <script type="math/tex">n</script> according to their personal preference.

After running the Gale-Shapley algorithm (which takes <script type="math/tex">O(n^2)</script> time),
all of the men and women are paired up,
and all of these matchings are guaranteed to be _stable_:
in other words, there exist no two pairings such that the man from the first pairing
prefers the woman from the second pairing to their current partner, and vice versa.
We can intuitively see how this is, in a sense, 'optimal', as no two people would rather leave their current partners
for each other.

It is easy to guess how the questionnaire used by the Aphrodite Project might translate into preference rankings, and then matches.
We can represent one's question scores as a normalised <script type="math/tex">n</script>-dimensional vector,
and compute some measure of similarity between vectors using dot products.
Similarity scores are then used as proxies for preference rankings,
upon which the Gale-Shapley algorithm is then run to obtain the final pairings.

But of course, if we were to apply this algorithm naïvely for matchmaking, we would immediately run into problems.
First of all, not all men and women are heterosexual, as there are gay people too:
within this group, any person may be potentially matched to anyone else.
How might one reformulate the matching problem to take gay pairings into account?

### 2. Stable Roommates Problem (SRP)

Suppose we no longer have two distinct classes of "men" and "women" where matchings must occur between groups,
but only a single class of people whereby anyone may potentially be matched to any other.
This reformulation may be described as the _Stable Roommates Problem_ (SRP), where all members are of the same class (i.e. "roommates").

Building upon the Gale-Shapley algorithm, Robert Irving gave in a [1985 paper][irving-roommates-paper]
an algorithm that efficiently solves the SRP.
Unlike the SMP, a stable matching is not guaranteed to exist;
but supposing that it does, Irving's algorithm will find one in <script type="math/tex">O(n^2)</script> time,
where <script type="math/tex">2n</script> is the total number of people.

This suggests that in order to account for gay pairings, we can split the dating pool into three subgroups:
straight men and women, gay men, and lesbians.
For straight men and women, we can run the Gale-Shapley algorithm on it as before;
as for gay men and lesbians, these are both instances of the SRP,
so we can run Irving's algorithm on each group separately to get our final matchings.

But there are more problems. Once again, not everyone falls neatly into the gay-straight divide,
leaving behind those who are bisexual and pansexual, among others.
Further, one's gender might not be strictly binary,
and there should be ways to take into account non-binary and transgender people too.
Some may also wish to impose restrictions upon other grounds, such as dating only those
living in the same area or those of the same religion, and so on.

### 3. Generalised Stable Roommates Problem (SRP) with Forbidden Pairs

All these problems can be tackled with a generalised version of the SRP,
where everyone is open to being matched up with anyone else by default unless specific exclusions are described
(i.e. certain pairs are forbidden).

In the case of matchmaking, for example, heterosexual men and women will be excluded from pairing with those not of the opposite sex,
and gay men and women will be excluded from pairing with those not of the same sex;
in general, any restriction can be made depending on one's identity markers and personal circumstances.

The generalised SRP with forbidden pairs can be solved in <script type="math/tex">O(n^2)</script> time,
but only if a stable matching exists [(Fleiner et al., 2007)][fleiner-et-al-paper].
Otherwise, finding the most stable solution, with the fewest number of unstable matchings, is NP-hard
[(Abraham et al., 2005)][abraham-et-al-paper].

It is still possible to run some modified version of Irving's algorithm on the generalised SRP,
but since a stable matching is very unlikely to exist, we can no longer make good guarantees about the algorithm's performance.
And if we wish to find a matching that is the least unstable, we won't be able to compute it in polynomial time.

### Conclusions

Starting from the SMP and moving to the more generalised SRP with forbidden pairs,
we've seen that any viable matchmaking algorithm in real-life must be many steps removed from the original Gale-Shapley algorithm.
These changes also entail that the algorithm is no longer guaranteed to give an optimally stable outcome.
For that reason, the initial allure and promise of a Nobel Prize-winning algorithm seems to ring a little hollow.

Let's also not forget that matchmaking is a complex problem that can't simply be reduced to the SMP or SRP.
After all, is similarity really the best predictor of compatibility?
The project's methodology seems to be based on the premise that if two people score similarly on a questionnaire,
then they are somehow more likely to be attracted to each other.
But naturally, this might not be the case.

Anecdotally, at least, most matches given by the Aphrodite Project fail.
Many don't go beyond a face-to-face meeting, or a couple of exchanged messages.
Some are even dreadfully ghosted the minute the match is sent.

In truth, predicting romantic desire is a [thorny problem][romantic-desire-predictable-paper].
Dating apps routinely try to do so to [little or no avail][bbc-article-dating-apps],
even if they're supported by a large machinery of analysts and data scientists working from behind the scenes.
With such a difficult problem, how can we expect a single algorithm to perform well?
No matter how "Economics Nobel-Prize winning" an algorithm may be,
it means little if it fundamentally isn't cut out to solve the problem at hand.

<div style="padding-bottom:10px"></div>

_(That being said, I do still think that the project is worthwhile. Even if it might not work at rates significant beyond chance levels, it gives one an opportunity to seek others out. Who knows if a match might actually arise?_ 🙂_)_


[the-octant]: https://theoctant.org/edition/issue/features/from-arrow-to-algorithm-the-aphrodite-project-could-find-you-true-love/
[gale-shapley-paper]: http://www.eecs.harvard.edu/cs286r/courses/fall09/papers/galeshapley.pdf
[irving-roommates-paper]: https://www.sciencedirect.com/science/article/abs/pii/0196677485900331
[fleiner-et-al-paper]: https://www.sciencedirect.com/science/article/pii/S0304397507003386
[abraham-et-al-paper]: https://link.springer.com/chapter/10.1007%2F11671411_1
[romantic-desire-predictable-paper]: https://journals.sagepub.com/doi/abs/10.1177/0956797617714580?journalCode=pssa
[bbc-article-dating-apps]: https://www.bbc.com/future/article/20191112-how-dating-app-algorithms-predict-romantic-desire

<script src="/public/js/katex_render.js"></script>
