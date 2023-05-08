Estimating Pi With a Shotgun
================
Andrew Mascillaro
2023-03-14

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

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.2     ✔ readr     2.1.4
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.0
    ## ✔ ggplot2   3.4.2     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.2     ✔ tidyr     1.3.0
    ## ✔ purrr     1.0.1     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

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
    ## This warning is displayed once every 8 hours.
    ## Call `lifecycle::last_lifecycle_warnings()` to see where this warning was
    ## generated.

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
## TASK: Choose a sample size and generate samples
set.seed(strtoi("wowow", base = 36))
n <- 10000 # Choose a sample size
df_q1 <- tibble(
  x = runif(n, 0, 1),
  y = runif(n, 0,)
) # Generate the data
```

### **q2** Using your data in `df_q1`, estimate $\pi$.

``` r
## TASK: Estimate pi using your data from q1
df_q2 <- df_q1 %>%
  mutate(
    in_circ = (x ^ 2 + y ^ 2) < 1,
    stat = in_circ * 4
  ) %>%
  summarize(
    in_circ_mean_adj = mean(stat),
    in_circ_sd_adj = sd(stat),
    n = nrow(df_q1),
    in_circ_se = in_circ_sd_adj / sqrt(n),
    pi_est = in_circ_mean_adj
  )
df_q2
```

    ## # A tibble: 1 × 5
    ##   in_circ_mean_adj in_circ_sd_adj     n in_circ_se pi_est
    ##              <dbl>          <dbl> <int>      <dbl>  <dbl>
    ## 1             3.13           1.65 10000     0.0165   3.13

``` r
pi_est <- df_q2$pi_est
```

# Quantifying Uncertainty

<!-- -------------------------------------------------- -->

You now have an estimate of $\pi$, but how trustworthy is that estimate?
In `e-stat06-clt` we discussed *confidence intervals* as a means to
quantify the uncertainty in an estimate. Now you’ll apply that knowledge
to assess your $\pi$ estimate.

### **q3** Using a CLT approximation, produce a confidence interval for your estimate of $\pi$. Make sure you specify your confidence level. Does your interval include the true value of $\pi$? Was your chosen sample size sufficiently large so as to produce a trustworthy answer?

``` r
q99 <- qnorm( 1 - (1 - 0.99) / 2 )
df_q2 %>%
  summarize(
    lo_pi = in_circ_mean_adj - q99 * in_circ_se,
    hi_pi = in_circ_mean_adj + q99 * in_circ_se
  )
```

    ## # A tibble: 1 × 2
    ##   lo_pi hi_pi
    ##   <dbl> <dbl>
    ## 1  3.08  3.17

**Observations**:

- Does your interval include the true value of $\pi$?
  - My interval includes the true value of $\pi$.
- What confidence level did you choose?
  - I chose a 99% confidence level
- Was your sample size $n$ large enough? Why do you say that?
  - It appears that a sample size of 10,000 was large enough for
    recreational purposes of approximating $\pi$.
  - nobody actually depends on my estimation of the number, and the only
    goals for the estimate are to be within the confidence interval and
    for the confidence interval to be small enough for me to
    qualitatively think, “wowow, this estimate of $\pi$ looks cool”
  - In this case, the confidence interval is less than 0.1, and the true
    value of $\pi$ was within this interval

# References

<!-- -------------------------------------------------- -->

\[1\] Dumoulin and Thouin, “A Ballistic Monte Carlo Approximation of Pi”
(2014) ArXiv, [link](https://arxiv.org/abs/1404.1499)

\[2\] “How Mathematicians Used A Pump-Action Shotgun to Estimate Pi”,
[link](https://medium.com/the-physics-arxiv-blog/how-mathematicians-used-a-pump-action-shotgun-to-estimate-pi-c1eb776193ef)

\[3\] Art Owen “Monte Carlo”,
[link](https://statweb.stanford.edu/~owen/mc/)
