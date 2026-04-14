---
title: Latin Hypercube Sampling
date: '2020-12-18'
slug: latin-hypercube-sampling
topics:
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
##             a          b         c
## 1  0.50695490 0.53384091 0.6236271
## 2  0.70381564 0.87217700 0.2892675
## 3  0.09761828 0.77257902 0.5161118
## 4  0.06103976 0.22427145 0.9689555
## 5  0.45270101 0.09700284 0.7680376
## 6  0.58669305 0.97829841 0.1080888
## 7  0.32827328 0.42633203 0.1854635
## 8  0.38817769 0.64999106 0.7082770
## 9  0.97219897 0.39859057 0.0435290
## 10 0.78798645 0.28261713 0.8646416
## 11 0.24214768 0.05467534 0.3580545
## 12 0.86691463 0.70473584 0.4890290
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
##  1 2     FALSE blue 
##  2 3     FALSE green
##  3 1     FALSE blue 
##  4 1     TRUE  black
##  5 2     TRUE  black
##  6 2     FALSE red  
##  7 1     TRUE  red  
##  8 2     FALSE blue 
##  9 3     TRUE  red  
## 10 3     TRUE  black
## 11 1     TRUE  green
## 12 3     FALSE green
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
##  1 Merc 230           virginica  Y     
##  2 Maserati Bora      setosa     X     
##  3 Fiat 128           virginica  Z     
##  4 AMC Javelin        versicolor X     
##  5 Pontiac Firebird   setosa     Z     
##  6 Merc 450SL         virginica  Y     
##  7 Volvo 142E         virginica  Z     
##  8 Cadillac Fleetwood versicolor X     
##  9 Mazda RX4          versicolor Y     
## 10 Valiant            setosa     X
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
##  [1] 2.013910 2.407631 1.195237 1.122080 1.905402 2.173386 1.656547 1.776355
##  [9] 2.944398 2.575973 1.484295 2.733829
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
