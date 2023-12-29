---
layout: post
title: Upper Confidence Bounds for Multi-Arm Bandits
date: 2023-12-28 11:12:00-0400
description: Deriving Sutton and Barton's UCB Bandit Algoritmhs
tags: math statistics reinforcement-learning
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
An $$\epsilon$$-greedy algorithm makes no distinction between when we have 100000 samples or just 1 sample 
to decide how much or how to explore.

### Vanilla Upper Confidence Bounds
When we get more samples, we learn more about the variance of the different arms.

One way to formalize this is to construct a confidence interval and then pick the
arm which has the highest upper confidence bound.

Lets say that we want to construct a $$95$$% confidence interval for $$\mathbb{E}[R_a] = \mu_a$$. 
Recal the Central Limit Theorem allows us to find a confidence interval for the rewards from an action $$a$$:

$$ 
\overline{x} \pm \Phi^{-1}(.025)\cdot \frac{\sigma_{R_a}}{\sqrt{n}}
$$

This suggests the vanilla confidence bound algorithm:
- Pick the action with the highest $$95$$% upper confidence bound. 
$$
A_{t}\doteq \underset{a}{\arg\max}\left[Q_{t}(a) + \Phi^{-1}(.975)\cdot \sigma_{R_a} \frac{1}{\sqrt{N_t(a)}}\right]
$$
 
Note:
- $$N_t(a) = \sum\limits_{i=1}^{t-1} \mathbf{1}\{a_t = a\}$$ is the number of times we selected action $$a$$ before time $$t$$.
- $$Q_t(a)$$ is our estimate for $$\mathbb{E}[R_a]$$ the mean of arm $$a$$ at time $$t$$.

OK, cool, but there's one glaring problem here: We don't know $$\sigma_{R_a}$$.  We could estimate it using the
sample variance and use a quantile in the $$t$$-distribution instead of the normal, but what if we just, *gasps*, make 
it a **hyper-parameter** and make it our user's problems? Lets call that $$c_a$$

And while we're at it, collapse $$\Phi^{-1}(.975)$$ into the constant too.

$$
\begin{align}
A_{t} &\doteq \underset{a}{\arg\max}\left[Q_{t}(a) + \Phi^{-1}(.975)\cdot \sigma_{R_a} \frac{1}{\sqrt{N_t(a)}}\right] \\
      &=\underset{a}{\arg\max}\left[Q_{t}(a) + c_a \frac{1}{\sqrt{N_t(a)}}\right]
\end{align}
$$

### Agumenting Upper Confidence Bounds for Exploration
Now, a problem with this is that you might get extraordinarily unlucky and pull a couple really low rewards for the
optimal arm. If that happens, our estimate $$Q_t(a)$$ of $$\mathbb{E}[R_a] = \mu_*$$ will be way too low and we'll never recover
because our upper confidence bound will be far too low (recall that $$95$$% of confidence intervals will contain 
the true parameter, so that means $$5$$% of them don't!).

To deal with that, one elegant way is to artificially increase the variance for each estimate of $$\mathbb{E}[R_a] = \mu_a$$.
We can do that with a variance multipler which we'll call $$\lambda_a(t)^2$$.

$$
A_{t} =\underset{a}{\arg\max}\left[Q_{t}(a) + c_a \cdot \sqrt{\lambda_a(t)^2} \cdot \frac{1}{\sqrt{N_t(a)}}\right]
$$

Well, we want $$\lambda_a(t)$$ to decrease over time, right? But, we want to make sure that it's always unbounded
so that it gives any estimate the ability to recover from a really unlucky streak. What if we set it to $$\lambda_a(t)^2 = \ln t$$.
This would make it so that if we have $$\ln t$$ samples for action $$a$$, then the variance multipler to $$c$$ in our confidence
bound estimate will be 1

$$\sqrt{\lambda_a(t)^2} \cdot \frac{1}{\sqrt{N_t(a)}} = \sqrt{\frac{\lambda_a(t)^2}{N_t(a)}} = \sqrt{\frac{\ln t}{\ln t}} = \sqrt{1} = 1$$

If all of the means are of the same magnitude, 
this will encourage us to have $$O(\ln t)$$ samples for each action $$a$$ at any given point in time. 
If we have less than $$\ln t$$
samples for a point in time $$t$$, then the upper confidence bound for $$a$$ will be more than $$c$$ away from the estimated mean.

Then, we get

$$
\begin{align}
A_{t} &=\underset{a}{\arg\max}\left[Q_{t}(a) + c_a \cdot \sqrt{\lambda_a(t)^2} \cdot \frac{1}{\sqrt{N_t(a)}}\right] \\
&= \underset{a}{\arg\max}\left[Q_{t}(a) + c_a \cdot \sqrt{\ln t} \cdot \frac{1}{\sqrt{N_t(a)}}\right]  \\
&= \underset{a}{\arg\max}\left[Q_{t}(a) + c_a \cdot \sqrt{\frac{\ln t}{N_t(a)}}\right] 
\end{align}
$$

This is the final formulation in the book by Sutton and Barto.