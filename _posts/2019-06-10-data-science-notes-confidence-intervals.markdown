---
layout: post
title:  "Data Science Notes: Confidence Intervals"
date:   2019-06-10 16:18:39 +0200
---

This year I resolved to start learning Data Science on my free time with the expectation of finding a way to use it in my every day work.

To help me in the process I have been writing some notes in the format of a private blog (which is a recurring [tip that experts give to begginners](http://datastori.es/120-data-science-with-david-robinson/#t=31:46.163)). I'll start publishing my private notes some other day, but today I wanted to write about the lastest item I have read about.

**Disclaimer:** The following are more _my notes_ and less _a tutorial_.

## Confidence Intervals

According to [Harvey Motulsky](https://stats.stackexchange.com/users/25/harvey-motulsky), in [Essential Biostatistics](https://www.goodreads.com/en/book/show/25923604-essential-biostatistics):

CIs express precission or margin of errors and so let you **make a general conclusion from limited data**. But this only works under the following assumptions:

* random sample / representative sample
* independent observations
* correctly tabulated data / free of bias

### Meaning of 95% CI

If you calculate the 95% CI of a given observation, you would expect the real population to be encompassed by your CI 95% of the times. But there is no way for you to know whether the real population lies within your CI or not.

We **do not** say _there is a 95% chance that the true population is in my CI_, that's flipping things around. The true population is fixed, and there is 95% chance that your CI contains it.

To make this more clear: let's say you forgot your keys at the kitchen table (but you don't remember where). If I ask you to guess where your keys are you may say something like _"I am 95% sure that I forgot them at home"_, which is different from _"There is a 95% chance that the keys are at home"_. Your keys are 100% at home, whether you know it or not.

### A simulation

Harvey proposes the following exercise to understand better. I added some code to help.

Imagine you have a bowl with 100 balls, 25 of them are red and 75 are black. Pretend you are a researcher who doesn't know the real distribution.

{% highlight python %}
red_balls = ['red' for i in range(25)]
black_balls = ['black' for i in range(75)]
bowl = red_balls + black_balls
{% endhighlight %}

Next we mix the balls, choose one randomly and put it back again in the bowl. We repeat this process 15 times and calculate the 95% CI for this proportion:


{% highlight python %}
import random
from statsmodels.stats.proportion import proportion_confint

def simulation():
    number_of_red_balls_observed = 0
    NUMBER_OF_TRIALS = 15
    CONFIDENCE_LEVEL = 0.95
    for i in range(NUMBER_OF_TRIALS):
        random.shuffle(bowl)
        ball = random.choice(bowl)
        if ball == 'red':
            number_of_red_balls_observed += 1
    ci_low, ci_up = proportion_confint(number_of_red_balls_observed, NUMBER_OF_TRIALS, alpha=1.0 - CONFIDENCE_LEVEL)
    observed_proportion = number_of_red_balls_observed/NUMBER_OF_TRIALS
    return (ci_low, ci_up, observed_proportion)
{% endhighlight %}

If we repeat this exercise 20 times we should see that:

* about half of the times the observed proportion is above the real population
* the other half of the times it would be lower
* 5% of the times the calculate CI will not encompass the real population

The following figure shows the 20 confidence intervals in the form of bars, with a line in the middle indicating the observed proportion. A horizontal line show the true proportion (25% of the balls are red).

<p style="text-align: center;">
    <img src="{{ site.url }}/assets/data-sci-notes/confidence_intervals.png" alt="Figure 1: Confidence intervals of 20 samples from a binomial distribution B(15,0.25)">
    <small>Figure 1: Confidence intervals of 20 samples from a binomial distribution B(15,0.25)</small>
</p>

#### The code
You can find and run the full code at: [github.com/ariera/essential-biostatistics](https://github.com/ariera/essential-biostatistics)



{% include comments.html %}