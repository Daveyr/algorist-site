---
title: Latin Hypercube Sampling
date: '2020-12-18'
slug: latin-hypercube-sampling
categories:
  - Guide
tags:
  - R
  - Algorithm
---

## What is LHS?
Latin hypercube sampling aims to bring the best of both worlds: the unbiased random sampling of monte carlo simulation; and the even coverage of a grid search over the decision space. It does this by ensuring values for all variables are as uncorrelated and widely varying as possible (over the range of permitted values).

## Why do I need one?
The background to this is that I have been working on improving the convergence of an optimisation package that uses genetic algorithms. Having a good distribution of "genes", or decisions within the initial population is key to allowing a GA to effectively explore the decision space. The `ga()` function within the GA package in R allows for a _suggestions_ argument, which takes a matrix of decision values and places them within the starting population. Initially I wrote one function to create evenly spaced sequences for every decision between the lower and upper bound of allowable values, from which I enumerated all possible combinations using `expand.grid()`. I then wrote another function to take a model object and a user-defined population size and automatically work out the nearest population count that allowed for even sampling over the model's decision bounds.

For models with large numbers of decisions the potential number of combinations is enormous. This is true even if you only select the upper and lower bound per decision. Already with 30 independent decisions with two possible values the number of combinations is 2^30 = 10,737,41,824, 10 times more than the number of stars in the average galaxy. Combinatorial algorithms are the worst type for growth in complexity (`\(O(n!)\)`). I needed a way of randomly sampling from the decisions but in a way that ensured the starting population had lots of diversity. This is the exact use case for LHS.

## Practical examples
Let's assume we have three decisions, where:

* decision a can be between 1 and 3,
* decision b can be TRUE or FALSE, and
* decision c can be red, green, blue or black

There are `\(3 * 2 * 4 = 24\)` possible unique combinations of these decisions (if a can only take on integer values). An individual in the starting population of a genetic algorithm would be of the form, `1_TRUE_red`, for instance.

Let's assume we would like only 12 individuals from the 24 potential unique combinations, but we still want good representation of all/most possible decisions. Using the lhs library from R, we first create 12 random uniform distributions between 0 and 1 for each of the three decisions, a, b and c.


```
##              a          b          c
## 1  0.919315659 0.29722517 0.89067913
## 2  0.219737090 0.54003818 0.97684103
## 3  0.786035592 0.09884228 0.02435949
## 4  0.291249510 0.80647784 0.72672712
## 5  0.385983258 0.69965332 0.36750225
## 6  0.142970749 0.42988014 0.33153931
## 7  0.681980810 0.89320886 0.82536674
## 8  0.611891701 0.05151168 0.42422109
## 9  0.003686905 0.38770931 0.13295385
## 10 0.553027147 0.60422350 0.62108657
## 11 0.847629535 0.19273206 0.52636935
## 12 0.443599532 0.98950661 0.21348403
```

Next we map the 0-1 distributions onto the real distributions of a, b and c. For instance, c has four possible values so the distribution should be 1-4. We also convert the values to factors in order to label them properly.


``` r
# Random choices for each decision with distributions based on a, b and c
choices <- map2(sample, all_decisions,
                ~cut(.x, length(.y), labels = as.character(.y))) %>%
  bind_rows()
  
choices
```

```
## # A tibble: 12 × 3
##    a     b     c    
##    <fct> <fct> <fct>
##  1 3     TRUE  black
##  2 1     FALSE black
##  3 3     TRUE  red  
##  4 1     FALSE blue 
##  5 2     FALSE green
##  6 1     TRUE  green
##  7 3     FALSE black
##  8 2     TRUE  green
##  9 1     TRUE  red  
## 10 2     FALSE blue 
## 11 3     TRUE  blue 
## 12 2     FALSE red
```

For convenience we can bring this into a single function like so.

``` r
lhs_sample <- function(decisions, n){
  stopifnot(is.list(decisions))
  len_decisions <- length(decisions)
  samples <- lhs::randomLHS(n, len_decisions) %>%
    as.data.frame()
  names(samples) <- names(decisions)
  choices <- purrr::map2(samples, decisions,
                  ~cut(.x, length(.y), labels = as.character(.y)))
  bind_rows(choices)
}

lhs_sample(list(cars = rownames(mtcars), species = unique(iris$Species), letter = LETTERS[24:26]), n = 10)
```

```
## # A tibble: 10 × 3
##    cars               species    letter
##    <fct>              <fct>      <fct> 
##  1 Pontiac Firebird   setosa     Z     
##  2 Valiant            virginica  Z     
##  3 Ford Pantera L     setosa     Y     
##  4 Hornet Sportabout  setosa     X     
##  5 Cadillac Fleetwood setosa     Y     
##  6 Honda Civic        virginica  Z     
##  7 Volvo 142E         versicolor Y     
##  8 Mazda RX4          versicolor X     
##  9 Merc 280           virginica  X     
## 10 AMC Javelin        virginica  X
```

## Continuous distributions
The above example was for sampling from distributions where allowable values were either factors or whole integers. For continuous numerical distributions we can use the following.


``` r
library(tidyr)
a_continuous <- all_decisions[[1]]

choices_continuous <- qunif(sample$a, min = min(a_continuous), max = max(a_continuous))
  
choices_continuous
```

```
##  [1] 2.838631 1.439474 2.572071 1.582499 1.771967 1.285941 2.363962 2.223783
##  [9] 1.007374 2.106054 2.695259 1.887199
```

## Testing coverage
In theory, each sample should be orthogonal, or independent, of each other sample to the greatest possible extent. Another way of putting it is that there should neither be any one value overrepresented or under-represented in the sample set. Let's test this in practice:


``` r
library(ggplot2)
big_sample <- lhs_sample(decisions = list(a = a,b = b,c = c), n = 1000)

ggplot(big_sample, aes(x = c, fill = c)) +
  geom_bar() +
  scale_fill_manual(values = c("red", "green", "blue", "black")) +
  labs(title = "For a single decision, each value has been sampled equally")
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/coverage_chart-1.png" alt="" width="672" />

``` r
ggplot(big_sample, aes(x = a, y = b, colour = c)) +
  geom_jitter() +
  scale_colour_manual(values = c("red", "green", "blue", "black")) +
  labs(title = "Every value combination across all decisions has also been sampled equally")
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/coverage_chart-2.png" alt="" width="672" />
