---
title:  "Data Science Interview Study Guide Part I: Statistics"
category: posts
date: 2019-07-04
---

# TODO

- Finish writing section on LR
- Make notation on "other questions" consistent; bold questions, include brief
  answers, link more in-depth discussions.
- Link to cross-validated, and other good stats resources (Gelman?)
- Section on Bayesian inference?
- Discuss chi-squared distribution and/or tests?

As I have gone through a couple rounds of interviews for data scientist
positions, I've been compiling notes on what I consider to be the essential
areas of knowledge. I want to make these notes available to the general public;
although there are many blog posts out there that are supposed to help one
prepare for data science interviews, I haven't found any of them to be very
high-quality. 

From my perspective, there are four key subject areas that a data scientist
should feel comfortable with when going into an interview:

1. Statistics (including experimental design)
2. Machine Learning
3. Software Engineering (including SQL)
4. "Soft" Questions

I'm going to go through each of these individually. This first post will focus
on statistics. We will go over a number of topics in statistics in no particular
order. Note that **this post will not teach you statistics; it will remind you
of what you should already know.**

If you're utterly unfamiliar with the concepts I'm mentioning, I'd recommend
[this excellent MIT course on probability & statistics][1] as a good starting
point. When I began interviewing, I had never taken a statistics class before; I
worked through the notes, homeworks, and exams for this course, and at the end
had a solid foundation to learn the specific things that you need to know for
these interviews.

Of course, this should all be taken with a grain of salt; I've been in the
industry a relatively short time, and haven't done very many interviews. I hope
to improve this guide over time; please let me know in the comments if there's
something you think should be added, removed, or changed!

# The Central Limit Theorem

**Example question: What is the central limit theorem? Why is it useful?**

The Central Limit Theorem is a fundamental tool in statistical analysis. It
state (roughly) that when you add up a bunch of random variables with finite
variance, then their sum will converge to a Gaussian distribution.[^fnote1]

What does this mean to a data scientist? Well, one place where we see a sum of
random variables is in a _sample mean_. One consequence of the central limit
theorem is that the sample mean of a variable with mean $$\mu$$ and variance
$$\sigma^2$$ will itself have mean $$\mu$$ and variance $$\sigma^2/n$$, where
$$n$$ is the number of saples.

This is cool! We don't need to know much of anything about the distribution of
we're sampling from, besides its mean and variance. This simplification will
allow us to do hypothesis testing that compares two means with relative ease.

# Hypothesis Testing

**Example question: 47 out of 100 people clicked the blue button. 26 out of 50
  clicked the green button. How confident are you that the green button is
  better than the blue button?**

Hypothesis testing is a huge subject, both in scope and importance. We use
statistics to quantitatively answer questions based on data, and hypothesis
testing is the method by which we construct these answers. Here we'll just lay
out one possible case of a hypothesis test. It's simple, but it comes up all the
time in practice, so it's essential to know.

## An Example

Suppose we have two buttons, one green and one blue. We put them in front
of two different samples of users. For simplicity, let's say that each sample
has size $$n=100$$. We observe that $$k_\text{green}$$ 57 users click the green button, and only
$$k_\text{blue} = 48$$ click the blue button.

Seems like the green button is better, right? Well, we want to be able to say
how _confident_ we are of this fact. We'll do this in the language of null
hypothesis significance testing. I'm going to lay out a table of all the
important factors here, and then discuss

| Description | Value |
|------------|-------|
| Null Hypothesis| $$p_{blue} - p_{green} = 0$$ |
| Test Statistic | $$ \frac{k_\text{blue}}{n} - \frac{k_\text{green}}{n} $$ |
| Test Statistic's Distribution | $$N(0, (p_b(1-p_b) + p_g(1-p_g)) / n)$$ |
| Test Statistic's Observed Value | -0.09 | 
| $$p$$-value | 0.1003 |

There are a few noteworthy things here. First, we really want to know whether
$$p_g > p_b$$, but that's equivalent to $$p_b-p_g < 0$$. Second, we assume that
$$n$$ is large enough so that $$k/n$$ is approximately normally distributed,
with mean $$\mu = p$$ and variance $$\sigma^2 = p(1-p)/n$$. Third, since the
differences of two normals is itself a normal, the test statistics distribution
is (under the null hypothesis) a normal with mean zero and the variance given
(which is the sum of the two variances of $$k_b/n$$ and $$k_g/n$$). 

Finally, we don't actually know $$p_b$$ or $$p_g$$, so we can't really compute
the $$p$$-value; what we do is we say that $$k_b/n$$ is "close enough"" to
$$p_b$$ and use it as an approximation. That gives us our final $$p$$-value.

The $$p$$-value was calculated in Python, as follows:

{% highlight python %}
from scipy.stats import norm
pb = 0.48
pg = 0.57
n = 100
sigma = np.sqrt((pb*(1-pb) + pg*(1-pg))/n)
norm.cdf(-0.09, loc = 0, scale = sigma) # 0.10034431272089045
{% endhighlight %}

Phew! I went through that pretty quick, but if you can't follow the gist of what
I was doing there, I'd recommend you think through it until it is clear to
you. You will be faced with more complicated situations in practice; it's
important that you begin by understanding the most simple situation inside out.

## Other Topics in Hypothesis Testing

We've briefly covered a simple case here. But there are many follow-up
questions you might get after outlining such a test. Here are some important ones:

- You should know about Type I & II error. What are they? What is a situation
  where you would be more concerned with Type I error? Vice versa?

- You should know what the _power_ of a test is, and how to calculate it. To
  calculate the power, you need an alternative hypothesis; in the example
  above, this would look like $$p_b-p_g = -0.1$$. Although these alternative
  hypothesis are often somewhat ad-hoc, the power analysis depends critically
  upon them.

- You should know what the _significance_ of a test is. This is the same as the
  $$p$$-value threshold below which we reject the null
  hypothesis. (In)famously, 0.05 has become the de-facto standard throughout
  many sciences for significance levels worthy of publication.

- Speaking of $$p$$-values, you should understand them thoroughly. A very
  common question is **how would you explain a p-value to a lay person**? For
  what it's worth, I'm not convinced there's a great answer to that one; it's
  an inherently technical quantity that is frequently misrepresented and abused
  by people trying to (falsely) simplify its meaning.

- **Bonus example question:** If you measure 14 different test statistics, and
  get a $$p$$-value for each (all based on the same null hypothesis), how do
  you combine them to get an aggregate $$p$$-value?
  
# Confidence Intervals

Confidence intervals allow us to say how certain we are of a result. If we
count that 150 out of 400 people sample randomly from a city identify
themselves as male, then our best estimate of the fraction of women in the city
is 250/400, or 5/8. How certain are we of this fact?[^fnotea]

Suppose that we want to find a 95% confidence inverval on the female fraction
in the city discussed above. This corresponds to a significance level of
$$\alpha/2$$. There are a few different ways to generate confidence intervals.

## The Exact Method

To get the **exact confidence inverval**, we'd need to invert the CDF to find
where it hits $$\alpha/2$$ and $$1-\alpha/2$$. That is, we need to find the
value $$p_l$$ that solves the equation

$$CDF\left(n, p_l\right) = \alpha/2$$

and the value $$p_u$$ that solves the equation

$$CDF\left(n, p_u\right) = 1 - \alpha/2.$$

In these, $$CDF(n,p)$$ is the cumulative distribution function of a binomial
random variable with parameters $$n$$ and $$p.$$ Solving the two equations
above would give us our confidence inverval $$[p_l, p_u]$$.

Although it is useful for theoretical analysis, I rarely use this method in
practice, because I often do not actually know the true CDF of the statistic
I am measuring. Sometimes I do know the true CDF, but even in such cases, the
next (approximate) method is generally sufficient.

## The Approximate Method

If your statistic can be phrased as a sum, then its distribution approaches a
normal distribution.[^fnote2] This means that you can solve the above equations
for a normal CDF rather than (in the case above) a binomial CDF. Since a
binomial $$B(n,p)$$ approaches a normal with mean $$\mu=np$$ and variance
$$\sigma^2=np(1-p)$$, we just need to know when the CDF of a $$N(np,
np(1-p))$$ is equal to $$\alpha/2$$ (for the lower bound; getting the upper
bound is similar).

How does this help? It helps because, for a normal distribution, we've already
solved the above equations to find lower and upper bounds. In particular, the inverval
$$[\mu-\sigma,\mu+\sigma]$$, also called a $$1\sigma$$-interval, covers about
68% of the mass (probability) of the normal PDF. A table below indicates the
probability mass contained in various symmetric intervals on a normal
distribution:

| Inverval | Width[^fnote3] | Coverage |
|-------|----| ----|
| $$[\mu-\sigma,\mu+\sigma]$$ | $$1\sigma$$ | 0.683 |
| $$[\mu-2\sigma,\mu+2\sigma]$$ | $$2\sigma$$ | 0.954 |
| $$[\mu-3\sigma,\mu+3\sigma]$$ | $$3\sigma$$ | 0.997 |

Therefore, if we want an (approximate) 95% confidence interval on the
percentage of women in the population of our city, we can just do a two-sigma
interval. The distribution of the parameter $$p$$ is calculated by dividing the
binomial variable by $$n$$, so the mean is $$\mu= p$$ and the variance is
$$\sigma^2 = p(1-p)/n$$.[^fnote4] In our case, $$p=5/8$$, so our confidence
interval is $$5/8 \pm 15/1280 \approx 0.625 \pm 0.0117$$.

## The Bootstrap Method

The previous approach relies on the accuracy of approximating our statistic's
distribution by a normal distribution. Bootstrapping is a pragmatic, flexible
approach to calculating confidence intervals, which makes no assumptions on the
underlying statistics we are calculating. We'll go into more detail on
bootstrapping in general below, so we'll be pretty brief here.

The basic idea is that, once you have bootstrapped an empirical distribution
for your statistic of interest (in the example above, this is the percentage of
the population that is women), then you can simply find the $$\alpha/2$$ and
$$1-\alpha/2$$ percentiles, which then become your confidence interval. Of
course, in the case above, we would expect our empirical bootstrapped
distribution to converge to a normal with mean $$\mu= p$$ and variance
$$\sigma^2 = p(1-p)/n$$. However, we can reasonably calculate percentiles
_regardless_ of what the empirical distribution is; this is why bootstrapping
confidence intervals are so flexible.

As you'll see below, the downside of bootstrapping confidence intervals is that
it requires some computation. The amount of computation required can be
anywhere from trivial to daunting, depending on how many samples you want in
your empirical distribution.

**Overall, I would recommend using the approximate method, or bootstrapping if
you aren't confident that your statistic is normally distributed.** Of course,
the central limit theorem can provide some guarantees about the asympototic
distribution of certain statistics, so it's worth thinking through whether that
applies to your situation, or not.

## Other Topics in Confidence Intervals

- What is the definition of a confidence interval? This is a bit more
  technical, but it's essential to know that it is **not** "there is a 95%
  probability that the true parameter is in this range."
  
- How would this change if you wanted a **one-sided** confidence interval?

# Bootstrapping

As a data scientist, you have to have a good understanding of bootstrapping. It
is a technique that allows you to get insight into the quality of your
estimates, based only on the data you have. To understand it, let's look
through an example.

In the last section, we samples 400 people in an effort to understand what
percentage of a city's population identified as female. We got an estimate that
was $$5/8$$. This estimate it itself a random variable; if we had sampled
different people, we might have ended up with a different number. What if we
want to know the distribution of this estimate? How would we go about getting
that?

Well, the obvious way is to go out and sample 400 more people, and repeat this
over and over again, until we have many such fractional estimates. But what if
we don't have access to sampling more people? The natural thing is to think
that we're out of luck - without the ability to sample further, we can't
actually understand more about the distribution of our parameter (ignoring, for
the moment, that we have lots of theoretical knowledge about it via the CLT).

The idea behind bootstrapping is simple. Sample from the data you already have,
with replacement, a new sample of 400 people. This will give you an estimate of
the female fraction that is distinct from your original estimate, due to the
replacement in your sampling. You can repeat this process as many times as
you like; you will then get an empirical distribution whic approaches the true
distribution of the statistic.[^fnote4]

Bootstrapping has the advantage of belig flexible, although it does have its
limitations. Rather than get too far into the weeds, I'll just point you to the
[Wikipedia article on bootstrapping][2]. There are also tons of resources about
this subject online. Try coding it up for yourself! By the time you're
interviewing, you should be able to write a bootstrapping algorithm quite
easily.

**TODO add code snippet showing an example of bootstrapping**

## Other Topics in Bootstrapping



# Linear Regression

Regression is the study of the relationship between variables; for example, we
might wish to know how the weight of a person relates to their height. _Linear_
regression assumes that your input (height, or $$h$$) and output (weight, or
$$w$$) variables are _linearly related_, with slope $$\beta_1$$, intercept
$$\beta_0$$, and noise $$\epsilon$$.

$$w = \beta_1\cdot h + \beta_0 + \epsilon.$$

A linear regression analysis helps the user discover the $$\beta$$s in the
above equation. This is just the simplest application of LR; in reality, it is
quite flexible and can be used in a number of scenarios. 

Linear regression is another large topic that I can't really do justice to in
this article. Instead, I'll just go through some of the common topics, and
introduce the questions you should be able to address. As is the case with most
of these topics, you can look at the [MIT Statistics & Probability course][1]
for a solid academic introduction to the subject. You can also dig through [the
Wikipedia article][3] to get a more in-depth picture.

## Basic Questions on LR

- **How do you calculate the $$\beta$$s?** Generally you solve the so-called
  "normal equations", but you can use gradient descent to approximate the
  optimal solution when the design matrix is too large to invert.
  
- **How do you decide if you should use linear regression?** The best case is
  when the data is 2- or 3-dimensional; then you can just plot the data and see
  if it looks like "linear plus noise". However, if you have lots of
  independent variables, this isn't really an option. In such a case, you
  should look perform a linear regression analysis, and then look at the errors
  to verify that they look normally distributed and homoskedastic (constant
  variance).
  
- **What are the assumptions you make when doing a linear regression?** The
  Wikipedia article [addresses this point][4] quite thoroughly. This is worth
  knowing, because you don't just want to jump in and blindly do LR; you want
  to be sure it's actually a reasonable approach.
  
- **When is it a bad idea to do LR?** TODO. See Anscombe's Quartet.
  
- **Can you do linear regression on a nonlinear relationship?** In many cases,
  yes. What we need is for the model to be linear in the parameters $$\beta$$;
  if, for example, you are comparing distance and time for a constantly
  accelerating object $$d = 1/2at^2$$, and you want to do regression to
  discover the acceleration $$a$$, then you can just use $$t^2$$ as your
  independent variable. The model relating $$d$$ and $$t^2$$ is linear in the
  acceleration $$a$$, as required.
  
- **What does the "linear" in linear regression refer to?** TODO

- **What if your errors do not look heteroskedastic? What can you do in such a
  case?** TODO

- **What does the $$r^2$$ value of a regression indicate?** TODO

## Handling Overfitting

TODO

## Validaing a Linear Regression

TODO

## Logistic Regression

TODO



<!-------------------------------- FOOTER ----------------------------> 


[1]: https://ocw.mit.edu/courses/mathematics/18-05-introduction-to-probability-and-statistics-spring-2014/index.htm

[2]: https://en.wikipedia.org/wiki/Bootstrapping_(statistics)

[3]: https://en.wikipedia.org/wiki/Linear_regression

[4]: https://en.wikipedia.org/wiki/Linear_regression#Assumptions

[^fnote1]: Of course, the actual statement is careful about the mode of
    convergence, and the fact that it is actually an appropriately-normalized
    version of the distribution that converges, and so on.
    
[^fnotea]: If this feels familiar, it's because it's (statistically speaking)
    the same problem we worked with in the section on hypothesis testing.
    
[^fnote2]: Again, we're being loose here - it has to have finite variance, and
    the convergence is only in a specific sense.
    
[^fnote3]: I'm being a little loose with definitions here - the width of a
    $$2\sigma$$ inverval is actually $$4\sigma$$, but I think most would still
    describe it using the phrase "two-sigma".


<!-- Wish we could put this in _includes/scripts.html. But it doesn't run from -->
<!-- there. It needs to be run at the bottom of the file, rather than at the   -->
<!-- top; perhaps that has something to do with it. Anyways, I'll just include -->
<!-- this chunk of HTML at the footer of all my posts, even though its fugly.  -->

<div id="disqus_thread"></div>
<script>

/**
*  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
*  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables*/
/*
var disqus_config = function () {
this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
};
*/
(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');
s.src = 'https://pwills-com.disqus.com/embed.js';
s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
