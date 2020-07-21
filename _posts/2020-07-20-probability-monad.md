---
layout: post
title:  "The Probability Monad: Modelling Discrete Probabilities in Haskell"
date:   2020-07-20 22:30:00 +0800
tags:   probability monad haskell
---

<script src="/assets/js/jquery-1.11.1.min.js"></script>
<script src="/assets/js/katex.min.js"></script>
<link rel="stylesheet" href="/assets/css/katex.min.css">

Those who've used Haskell before have likely worked with a couple of well-known monads.
For example, the `List` represents a non-deterministic computation, and `Maybe` represents a computation
with a success or failure outcome.

What about probability distributions? As it turns out, it too is a Monad that we can implement in Haskell.

## 1. Probability Distributions

A _discrete probability distribution_ is a finite collection of outcomes that a random variable can take, whereby each outcome
has some non-zero probability of occurring.

We can represent this distribution as a list of
outcome-probability pairs `[(x, p)]`, where `x` represents an outcome
and `p` represents its probability. For example, the distribution of tossing a 
fair coin can be represented as `[('H', 1 % 2), ('T', 1 % 2)]`.

- The outcome can have a polymorphic type `a`, as it may be any type we wish.
- The probability can have a type synonym `Prob` defined for it. This could be
  `Double` or `Rational`, depending on your purposes.
- 👉 (We'll switch between `Double` and `Rational` freely in this post, so if some
  parts of the code don't work with one representation, try the other.)

```haskell
type Prob = Rational
newtype Dist a = Dist { runDist :: [(a, Prob)] }

-- For convenience's sake
instance (Show a) => Show (Dist a) where
  show dist = foldr (\(x, p) acc -> show x ++ ": " ++ show p ++ "\n" ++ acc) "" $ runDist dist
```

#### Example: Defining Probability Distributions

Now that we can represent discrete probability distributions, let's define a few well-known ones:

- _Uniform distribution_, over a list of outcomes as given in `xs`;
- _Bernoulli distribution_, with a success probability of `p` and failure probability of `1-p`;
- _Binomial distribution_, with `n` number of trials and a success probability of `p` in each trial.

```haskell
uniform :: [a] -> Dist a
uniform xs = Dist [(x, 1 / (toEnum $ length xs)) | x <- xs]

bernoulli :: Prob -> a -> a -> Dist a
bernoulli p success failure = Dist [(success, p), (failure, 1-p)]

binomial :: (Num a) => Int -> Prob -> Dist a
binomial n p = Dist [(fromIntegral k, coeff n p k) | k <- [0..n]]
  where coeff n p k = (fromIntegral $ choose n k) *
                      (p ^^ (fromIntegral k)) *
                      (1-p) ^^ (fromIntegral (n-k))
```

```haskell
-- Example: uniform 6-sided die
die = uniform [1..6]
{- 1: 1 % 6
   2: 1 % 6
   3: 1 % 6
   4: 1 % 6
   5: 1 % 6
   6: 1 % 6 -}
   
-- Example: binomial distribution, with n=5 and p=0.8
binom = binomial 5 0.8
{- 0: 1 % 3125
   1: 4 % 625
   2: 32 % 625
   3: 128 % 625
   4: 256 % 625
   5: 1024 % 3125 -}
```

## 2. Probability Functions 

With a given probability distribution, we can define some functions to help us find some of its statistics:
- `probOf`, which gives the probability of all outcomes that satisfy a given predicate;
- `expect`, `variance`, and `stdev`, which give the expectation, variance, and standard deviation of a distribution respectively.

```haskell
probOf :: (a -> Bool) -> Dist a -> Prob
probOf pred = sum . map snd . filter (\(x, p) -> pred x) . runDist

expect :: Dist Prob -> Prob
expect dist = sum [ x * p | (x, p) <- runDist dist ]

variance :: Dist Prob -> Prob
variance dist = sum [ (x - mean) ^^ 2 * p | (x, p) <- runDist dist ]
  where mean = expect dist

stdev :: Dist Prob -> Prob
stdev = sqrt . variance
```

```haskell
dieExp = expect die   -- 3.5
dieVar = variance die -- 2.916666...
dieSD = stdev die     -- 1.707825
dieProbGreater3 = probOf (>3) die -- 0.5
```

## 3. Dist as a Monad

This is all well and good, but why should the `Dist` probability distribution be a monad?
To get an intuition for this, let's first consider an example.

Let's say that on any given day of the week, it can be either `Sunny` with a probability of 0.8, or `Cloudy` with a
probability of 0.2. That’s simple---we can model this with a Bernoulli distribution with a parameter of 0.8.

But let's also say that depending on the weather condition, it can rain with some probability. If it's `Sunny`, it's
unlikely to rain, so our day has a 0.1 chance of being `Wet`. If it's `Cloudy`, then it's more likely to rain, with a 0.5
chance of our day being `Wet`.

![Weather-wetness probability tree](/assets/images/probability-monad-weather-wetness-tree.png)

```haskell
data Weather = Sunny | Cloudy deriving (Eq, Show)
data Wetness = Dry   | Wet    deriving (Eq, Show)

weather :: Dist Weather
weather = bernoulli 0.8 Sunny Cloudy

wetness :: Weather -> Dist Wetness
wetness Sunny  = bernoulli 0.1 Wet Dry
wetness Cloudy = bernoulli 0.5 Wet Dry
```

If we wish to find the overall probability of any given day being `Wet` or `Dry`, we’d ideally have some way to chain
`weather` and `wetness` together. First, we find the probability of a day being `Sunny` or `Cloudy`; then for each case,
we find the probability of it being `Wet` or `Dry`; finally, we combine the results together to give the overall distribution.

![Weather-wetness probability tree, condensed](/assets/images/probability-monad-weather-wetness-tree-2.png)

But here, we run into a problem. `weather` is of type `Dist Weather`, but
`wetness` is of type `Weather -> Dist Wetness`, so we cannot simply use function application:

```haskell
weather :: Dist Weather
wetness :: Weather -> Dist Wetness
wetness weather      -- this does not work!
wetness >>= weather  -- what about this instead?
```

Here's where the `>>=` operator comes in. It has a type signature of `m a -> (a -> m b) -> m b`, which
lines up perfectly well with what we want to do: to take `weather` of type `Dist Weather`, pass it using `>>=` into
`wetness` of type `Weather -> Dist Wetness`, and return a resulting distribution of type `Dist Wetness`! Naturally,
`m` would stand in for `Dist`, which implies that `Dist` is a monad.

How should we then implement the bind operator? In general, what should `xs >>= f` look like in the context of
a probability distribution?

Our initial distribution `xs` has a list of outcome-probability pairs, which we’ll call `(x, xp)`. For each
outcome `x`, let's feed it into the function `f`, which generates a new list of outcome-probability pairs for `x`, which
we'll call `(y, yp)`---the resulting outcome should then be `y`, where its probability is given by `xp * yp`.
And that's all for `>>=`!

As for `return`, it should simply put an outcome `x` in a minimal context---that is, a
distribution where its probability is simply 1.

```haskell
instance Monad Dist where
  return x = Dist [(x, 1)]
  xs >>= f = Dist [(y, xp*yp) | (x, xp) <- runDist xs,
                                (y, yp) <- runDist (f x)]
```

Let's try this on our previous example, and voilà! We get our desired probability distribution:

```haskell
day = weather >>= wetness
{- Wet: 2 % 25
   Dry: 18 % 25
   Wet: 1 % 10
   Dry: 1 % 10 -}
```

We're almost there, but there are repeated outcomes for `Wet` and `Dry`.
To help us eliminate duplicates by summing up the probabilities for each outcome, let's define a new helper function `condense`:

```haskell
condense :: (Eq a) => Dist a -> Dist a
condense dist = Dist (condenseList $ runDist dist)
  where condenseList [] = []
        condenseList xs@((x, p):ys) = (x, p'):(condenseList dropX)
          where takeX = filter (\(y, p) -> y == x) xs
                dropX = filter (\(y, p) -> y /= x) xs
                p' = sum $ map snd $ takeX

condense day
{- Wet: 9 % 50
   Dry: 41 % 50 -}
```

And we've solved our problem! Any given day has a <script type="math/tex">\frac{9}{50}</script> probability
of being `Wet`, and a <script type="math/tex">\frac{41}{50}</script> probability of being `Dry`.

Making further use our the probability monad, we can also model other interesting things, such as the probability
distribution of the sum of two independent 3-sided die rolls:

```haskell
threeDie = uniform [1..3]
sumDice = condense $ do
  x <- threeDie
  y <- threeDie
  return (x + y)
{- 2: 1 % 9
   3: 2 % 9
   4: 1 % 3
   5: 2 % 9
   6: 1 % 9 -}
```

## 4. Conditional Probability

Last but not least, with the probability monad, we can also apply conditions to prior probabilities to give us
conditional probabilities.

We can define `condition`, a function which applies a condition which we'll call `cond` to our distribution `dist`.
This condition filters for only the selected outcomes in our distribution, through the use of `guard`. Lastly, we `normalize`
the distribution to ensure a total probability of 1.

```haskell
condition :: Dist a -> (a -> Bool) -> Dist a
condition dist cond = normalize $ do
  x <- dist
  guard (cond x)
  return x

guard :: Bool -> Dist ()
guard True = Dist [((), 1)]
guard False = Dist []

normalize :: Dist a -> Dist a
normalize dist = Dist $ map (\(x, p) -> (x, p / totalP)) $ runDist dist
  where totalP = sum $ map snd $ runDist dist
```
 
#### Example: Boy-Girl Paradox

Now we can apply conditioning to solve some problems. Consider the classic Boy-Girl paradox:

| Mr. Smith has two children, where each has an equal probability of being either a boy or a girl. <br/> At least one of them is a boy. What is the probability that both children are boys? | 

```haskell
data Gender = Male | Female | Other deriving (Eq, Show)
child = bernoulli 0.5 Male Female
children = do
  x <- child
  y <- child
  return (x, y)
```

Conditioning `children` on either child being a boy, we find the probability of both children being boys to be
<script type="math/tex">\frac{1}{3}</script>:


```haskell
mrSmith = condition children (\(x, y) -> x == Male || y == Male)
probOf (==(Male, Male)) mrSmith -- 1 % 3
```

### References

1. Erwig, Martin, and Steve Kollmansberger. “Functional Pearls: Probabilistic Functional Programming in
Haskell.” Journal of Functional Programming, vol. 16, no. 1, 12 Sept. 2005, pp. 21–34.
2. Kidney, Donnacha Oisín. Probabilistic Functional Programming. 18 July 2018,
https://doisinkidney.com/pdfs/prob-presentation.pdf.
3. Ścibior, Adam D., et al. “Practical Probabilistic Programming with Monads.” Haskell ’15, Proceedings of the
2015 ACM SIGPLAN Symposium on Haskell, pp. 165–176.

_Adapted from an assignment made for CS2104: Programming Language Concepts._

<script src="/assets/js/katex_render.js"></script>
