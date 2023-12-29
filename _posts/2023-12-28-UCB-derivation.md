---
layout: post
title: Upper Confidence Bounds for Multi-Arm Bandits
date: 2023-12-28 11:12:00-0400
description: Deriving Sutton and Barton's UCB Bandit Algoritmhs
tags: math statistics 
categories: 
related_posts: false
---
[Sutton and Barto](http://incompleteideas.net/book/the-book-2nd.html) do a great job of explaining many things but 
the derivation of Upper Confidence Bounds is not great.

Let's get to whether they got differently.

### Problems with $$\epsilon$$-greedy algorithms
An $$\epsilon$$-greedy solution selects a random action with probability $$\epsilon$$ and selects the *greedy* action  
the the rest of the time.

If we assume that the means for the different "arms" are stationary, i.e. not changing over time,
$$\epsilon$$-greedy algorithms will *always*, even after we've been training for eons, select every action with probability 
$$\frac{\epsilon}{k}$$.

Ideally, we want to explore a lot at the beginning and exploit more as we sample more from the different arms. 
An $\epsilon$-greedy algorithm makes no distinction between when we have more information and
when we have less information. 

### Vanilla Upper Confidence Bounds
When we get more samples, we learn more about the variance of the different arms.

One way to formalize this is to construct a confidence interval and then pick the
arm which has the highest upper confidence bound.

Lets say that we're using a $$95$$% confidence interval. Lets use the Central Limit Theorem to get a
confidence interval for the rewards from an action $$a$$:

$$ 
\overline{x} \pm \Phi(.025)\cdot \frac{\sigma_{R_a}}{\sqrt{n}}
$$

This suggests our vanilla Upper Confidence Bound Algorithm, which will be to pick the action with the highest
$$95$$% upper confidence bound. $$N_t(a)$$ here will be the number of times we selected action $$a$$ before time $$t$$.
$$Q_t(a)$$ will be the estimate we have for the mean of arm $$a$$ at time $$t$$.

$$
A_{t}\doteq \underset{a}{\arg\max}\left[Q_{t}(a) + \Phi(.975)\cdot \sigma_{R_a} \frac{1}{\sqrt{N_t(a)}}\right]
$$

OK, cool, but there's one glaring problem here: We don't know $$\sigma_{R_a}$$.  We could estimate it using the
sample variance and use a quantile in the $$t$$-distribution, but what if we just *gasps* make it a **hyper-parameter**
and make setting it our user's problems? Lets call that $$c_a$$

And while we're at it, collapse $$\Phi(.975)$$ into the constant too.

$$
\begin{align}
A_{t} &\doteq \underset{a}{\arg\max}\left[Q_{t}(a) + \Phi(.975)\cdot \sigma_{R_a} \frac{1}{\sqrt{N_t(a)}}\right] \\
      &=\underset{a}{\arg\max}\left[Q_{t}(a) + c_a \frac{1}{\sqrt{N_t(a)}}\right]
\end{align}
$$

### Agumenting Upper Confidence Bounds for Exploration
Now, a problem with this is that you might get extraordinarily unlucky and pull a couple really low rewards for the
optimal arm.

To deal with that, one elegant way is to artificially increase the standard deviations we use in the confidence 
bounds as we increase time. We can do this with a multipler $$\lambda_a(t)$$.

$$
A_{t} =\underset{a}{\arg\max}\left[Q_{t}(a) + c_a \cdot \lambda_a(t) \cdot \frac{1}{\sqrt{N_t(a)}}\right]
$$

Well, we want $$\lambda_a(t)$$ to decrease over time, right? But, we want to make sure that it's always unbounded
so that it gives any estimate the ability to recover from a really unlucky streak. What if we set it to $$\lambda_a(t) = \sqrt{\ln t}$$.

Then, we get

$$
\begin{align}
A_{t} &=\underset{a}{\arg\max}\left[Q_{t}(a) + c_a \cdot \lambda_a(t) \cdot \frac{1}{\sqrt{N_t(a)}}\right] \\
&= \underset{a}{\arg\max}\left[Q_{t}(a) + c_a \cdot \sqrt{\ln t} \cdot \frac{1}{\sqrt{N_t(a)}}\right]  \\
&= \underset{a}{\arg\max}\left[Q_{t}(a) + c_a \cdot \sqrt{\frac{\ln t}{N_t(a)}}\right] 
\end{align}
$$

This is the final formulation in the book by Sutton and Barto.