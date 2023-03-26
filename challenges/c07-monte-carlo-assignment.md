Estimating Pi With a Shotgun
================
Lily Jiang
2023-3-26

- [Grading Rubric](#grading-rubric)
  - [Individual](#individual)
  - [Due Date](#due-date)
- [Monte Carlo](#monte-carlo)
  - [Theory](#theory)
  - [Implementation](#implementation)
    - [**q1** Pick a sample size $n$ and generate $n$ points *uniform
      randomly* in the square $x \in [0, 1]$ and $y \in [0, 1]$. Create
      a column `stat` whose mean will converge to
      $\pi$.](#q1-pick-a-sample-size-n-and-generate-n-points-uniform-randomly-in-the-square-x-in-0-1-and-y-in-0-1-create-a-column-stat-whose-mean-will-converge-to-pi)
    - [**q2** Using your data in `df_q1`, estimate
      $\pi$.](#q2-using-your-data-in-df_q1-estimate-pi)
- [Quantifying Uncertainty](#quantifying-uncertainty)
  - [**q3** Using a CLT approximation, produce a confidence interval for
    your estimate of $\pi$. Make sure you specify your confidence level.
    Does your interval include the true value of $\pi$? Was your chosen
    sample size sufficiently large so as to produce a trustworthy
    answer?](#q3-using-a-clt-approximation-produce-a-confidence-interval-for-your-estimate-of-pi-make-sure-you-specify-your-confidence-level-does-your-interval-include-the-true-value-of-pi-was-your-chosen-sample-size-sufficiently-large-so-as-to-produce-a-trustworthy-answer)
- [References](#references)

*Purpose*: Random sampling is extremely powerful. To build more
intuition for how we can use random sampling to solve problems, we’ll
tackle what—at first blush—doesn’t seem appropriate for a random
approach: estimating fundamental deterministic constants. In this
challenge you’ll work through an example of turning a deterministic
problem into a random sampling problem, and practice quantifying
uncertainty in your estimate.

<!-- include-rubric -->

# Grading Rubric

<!-- -------------------------------------------------- -->

Unlike exercises, **challenges will be graded**. The following rubrics
define how you will be graded, both on an individual and team basis.

## Individual

<!-- ------------------------- -->

| Category    | Needs Improvement                                                                                                | Satisfactory                                                                                                               |
|-------------|------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------|
| Effort      | Some task **q**’s left unattempted                                                                               | All task **q**’s attempted                                                                                                 |
| Observed    | Did not document observations, or observations incorrect                                                         | Documented correct observations based on analysis                                                                          |
| Supported   | Some observations not clearly supported by analysis                                                              | All observations clearly supported by analysis (table, graph, etc.)                                                        |
| Assessed    | Observations include claims not supported by the data, or reflect a level of certainty not warranted by the data | Observations are appropriately qualified by the quality & relevance of the data and (in)conclusiveness of the support      |
| Specified   | Uses the phrase “more data are necessary” without clarification                                                  | Any statement that “more data are necessary” specifies which *specific* data are needed to answer what *specific* question |
| Code Styled | Violations of the [style guide](https://style.tidyverse.org/) hinder readability                                 | Code sufficiently close to the [style guide](https://style.tidyverse.org/)                                                 |

## Due Date

<!-- ------------------------- -->

All the deliverables stated in the rubrics above are due **at midnight**
before the day of the class discussion of the challenge. See the
[Syllabus](https://docs.google.com/document/d/1qeP6DUS8Djq_A0HMllMqsSqX3a9dbcx1/edit?usp=sharing&ouid=110386251748498665069&rtpof=true&sd=true)
for more information.

``` r
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.2 ──
    ## ✔ ggplot2 3.4.0      ✔ purrr   1.0.1 
    ## ✔ tibble  3.1.8      ✔ dplyr   1.0.10
    ## ✔ tidyr   1.2.1      ✔ stringr 1.5.0 
    ## ✔ readr   2.1.3      ✔ forcats 0.5.2 
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

*Background*: In 2014, some crazy Quebecois physicists estimated $\pi$
with a pump-action shotgun\[1,2\]. Their technique was based on the
*Monte Carlo method*, a general strategy for turning deterministic
problems into random sampling.

# Monte Carlo

<!-- -------------------------------------------------- -->

The [Monte Carlo
method](https://en.wikipedia.org/wiki/Monte_Carlo_method) is the use of
randomness to produce approximate answers to deterministic problems. Its
power lies in its simplicity: So long as we can take our deterministic
problem and express it in terms of random variables, we can use simple
random sampling to produce an approximate answer. Monte Carlo has an
[incredible
number](https://en.wikipedia.org/wiki/Monte_Carlo_method#Applications)
of applications; for instance Ken Perlin won an [Academy
Award](https://en.wikipedia.org/wiki/Perlin_noise) for developing a
particular flavor of Monte Carlo for generating artificial textures.

I remember when I first learned about Monte Carlo, I thought the whole
idea was pretty strange: If I have a deterministic problem, why wouldn’t
I just “do the math” and get the right answer? It turns out “doing the
math” is often hard—and in some cases an analytic solution is simply not
possible. Problems that are easy to do by hand can quickly become
intractable if you make a slight change to the problem formulation.
Monte Carlo is a *general* approach; so long as you can model your
problem in terms of random variables, you can apply the Monte Carlo
method. See Ref. \[3\] for many more details on using Monte Carlo.

In this challenge, we’ll tackle a deterministic problem (computing
$\pi$) with the Monte Carlo method.

## Theory

<!-- ------------------------- -->

The idea behind estimating $\pi$ via Monte Carlo is to set up a
probability estimation problem whose solution is related to $\pi$.
Consider the following sets: a square with side length one $St$, and a
quarter-circle $Sc$.

``` r
## NOTE: No need to edit; this visual helps explain the pi estimation scheme
tibble(x = seq(0, 1, length.out = 100)) %>%
  mutate(y = sqrt(1 - x^2)) %>%

  ggplot(aes(x, y)) +
  annotate(
    "rect",
    xmin = 0, ymin = 0, xmax = 1, ymax = 1,
    fill = "grey40",
    size = 1
  ) +
  geom_ribbon(aes(ymin = 0, ymax = y), fill = "coral") +
  geom_line() +
  annotate(
    "label",
    x = 0.5, y = 0.5, label = "Sc",
    size = 8
  ) +
  annotate(
    "label",
    x = 0.8, y = 0.8, label = "St",
    size = 8
  ) +
  scale_x_continuous(breaks = c(0, 1/2, 1)) +
  scale_y_continuous(breaks = c(0, 1/2, 1)) +
  theme_minimal() +
  coord_fixed()
```

    ## Warning: Using `size` aesthetic for lines was deprecated in ggplot2 3.4.0.
    ## ℹ Please use `linewidth` instead.

![](c07-monte-carlo-assignment_files/figure-gfm/vis-areas-1.png)<!-- -->

The area of the set $Sc$ is $\pi/4$, while the area of $St$ is $1$. Thus
the probability that a *uniform* random variable over the square lands
inside $Sc$ is the ratio of the areas, that is

$$\mathbb{P}_{X}[X \in Sc] = (\pi / 4) / 1 = \frac{\pi}{4}.$$

This expression is our ticket to estimating $\pi$ with a source of
randomness: If we estimate the probability above and multiply by $4$,
we’ll be estimating $\pi$.

## Implementation

<!-- ------------------------- -->

Remember in `e-stat02-probability` we learned how to estimate
probabilities as the limit of frequencies. Use your knowledge from that
exercise to generate Monte Carlo data.

### **q1** Pick a sample size $n$ and generate $n$ points *uniform randomly* in the square $x \in [0, 1]$ and $y \in [0, 1]$. Create a column `stat` whose mean will converge to $\pi$.

*Hint*: Remember that the mean of an *indicator function* on your target
set will estimate the probability of points landing in that area (see
`e-stat02-probability`). Based on the expression above, you’ll need to
*modify* that indicator to produce an estimate of $\pi$.

``` r
set.seed(100)
n <- 1000 # Choose a sample size
df_q1 <- tibble(
  x = runif(n, min = 0, max = 1),
  y = runif(n, min = 0, max = 1),
  stat = ifelse(x^2 + y^2 < 1, 4, 0)
) # Generate the data

df_q1
```

    ## # A tibble: 1,000 × 3
    ##         x      y  stat
    ##     <dbl>  <dbl> <dbl>
    ##  1 0.308  0.0740     4
    ##  2 0.258  0.112      4
    ##  3 0.552  0.624      4
    ##  4 0.0564 0.671      4
    ##  5 0.469  0.366      4
    ##  6 0.484  0.183      4
    ##  7 0.812  0.0267     4
    ##  8 0.370  0.490      4
    ##  9 0.547  0.596      4
    ## 10 0.170  0.956      4
    ## # … with 990 more rows

### **q2** Using your data in `df_q1`, estimate $\pi$.

``` r
pi_est <- df_q1 %>%
  pull(stat) %>%
  mean()
pi_est
```

    ## [1] 3.092

# Quantifying Uncertainty

<!-- -------------------------------------------------- -->

You now have an estimate of $\pi$, but how trustworthy is that estimate?
In `e-stat06-clt` we discussed *confidence intervals* as a means to
quantify the uncertainty in an estimate. Now you’ll apply that knowledge
to assess your $\pi$ estimate.

### **q3** Using a CLT approximation, produce a confidence interval for your estimate of $\pi$. Make sure you specify your confidence level. Does your interval include the true value of $\pi$? Was your chosen sample size sufficiently large so as to produce a trustworthy answer?

``` r
confidence_level <- 0.99

lo <- pi_est - qnorm( 1 - (1 - confidence_level) / 2 ) * sd(df_q1 %>% pull(stat)) / sqrt(nrow(df_q1))
hi <- pi_est + qnorm( 1 - (1 - confidence_level) / 2 ) * sd(df_q1 %>% pull(stat)) / sqrt(nrow(df_q1))

lo
```

    ## [1] 2.955448

``` r
hi
```

    ## [1] 3.228552

**Observations**:

- Does your interval include the true value of $\pi$?
  - It does, 3.14 \> 2.955 and 3.14 \< 3.228
- What confidence level did you choose?
  - I chose a confidence level of 99%, as that allowed for a large range
    and would therefore have a good chance of including the true value
    of pi within it.
- Was your sample size $n$ large enough? Why do you say that?
  - I would say that a sample size of 1000 is large enough, as my
    confidence interval not only includes the real value of pi but also
    extends relatively broadly past it. This means that if the
    randomly-generated sample was somehow skewed, it is likely that the
    real value of pi will still fall within the interval. Of course,
    larger samples are always better for statistical precision (like
    narrowing the confidence interval), but it appears that 1000 is a
    reasonable size to catch the true value of pi.

# References

<!-- -------------------------------------------------- -->

\[1\] Dumoulin and Thouin, “A Ballistic Monte Carlo Approximation of Pi”
(2014) ArXiv, [link](https://arxiv.org/abs/1404.1499)

\[2\] “How Mathematicians Used A Pump-Action Shotgun to Estimate Pi”,
[link](https://medium.com/the-physics-arxiv-blog/how-mathematicians-used-a-pump-action-shotgun-to-estimate-pi-c1eb776193ef)

\[3\] Art Owen “Monte Carlo”,
[link](https://statweb.stanford.edu/~owen/mc/)