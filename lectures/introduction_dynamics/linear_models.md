---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  display_name: Julia
  language: julia
  name: julia-1.11
---

(lssm)=
```{raw} html
<div id="qe-notebook-header" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# Linear State Space Models

```{index} single: Linear State Space Models
```

```{contents} Contents
:depth: 2
```

```{epigraph}
"We may regard the present state of the universe as the effect of its past and the cause of its future" -- Marquis de Laplace
```

## Overview

This lecture introduces the **linear state space** dynamic system.

This model is a workhorse that carries a powerful theory of prediction.

Its many applications include:

* representing dynamics of higher-order linear systems
* predicting the position of a system $j$ steps into the future
* predicting a geometric sum of future values of a variable like
    * non financial income
    * dividends on a stock
    * the money supply
    * a government deficit or surplus, etc.
* key ingredient of useful models
    * Friedman's permanent income model of consumption smoothing
    * Barro's model of smoothing total tax collections
    * Rational expectations version of Cagan's model of hyperinflation
    * Sargent and Wallace's "unpleasant monetarist arithmetic," etc.



```{code-cell} julia
using LinearAlgebra, Statistics
```

## The Linear State Space Model

```{index} single: Models; Linear State Space
```

The objects in play are:

* An $n \times 1$ vector $x_t$ denoting the **state** at time $t = 0, 1, 2, \ldots$.
* An iid sequence of $m \times 1$ random vectors $w_t \sim N(0,I)$.
* A $k \times 1$ vector $y_t$ of **observations** at time $t = 0, 1, 2, \ldots$.
* An $n \times n$ matrix $A$  called the **transition matrix**.
* An $n \times m$ matrix $C$  called the **volatility matrix**.
* A $k \times n$ matrix $G$ sometimes called the **output matrix**.

Here is the linear state-space system

```{math}
:label: st_space_rep

\begin{aligned}
    x_{t+1} & =  A x_t + C w_{t+1}   \\
    y_t &  =  G x_t \nonumber \\
    x_0 & \sim N(\mu_0, \Sigma_0) \nonumber
\end{aligned}
```

(lss_pgs)=
### Primitives

The primitives of the model are

1. the matrices $A, C, G$
1. shock distribution, which we have specialized to $N(0,I)$
1. the distribution of the initial condition $x_0$, which we have set to $N(\mu_0, \Sigma_0)$

Given $A, C, G$ and draws of $x_0$ and $w_1, w_2, \ldots$, the
model {eq}`st_space_rep` pins down the values of the sequences $\{x_t\}$ and $\{y_t\}$.

Even without these draws, the primitives 1--3 pin down the *probability distributions* of $\{x_t\}$ and $\{y_t\}$.

Later we'll see how to compute these distributions and their moments.

#### Martingale difference shocks

```{index} single: Linear State Space Models; Martingale Difference Shocks
```

We've made the common assumption that the shocks are independent standardized normal vectors.

But some of what we say will be valid under the assumption that $\{w_{t+1}\}$ is a **martingale difference sequence**.

A martingale difference sequence is a sequence that is zero mean when conditioned on past information.

In the present case, since $\{x_t\}$ is our state sequence, this means that it satisfies

$$
\mathbb{E} [w_{t+1} | x_t, x_{t-1}, \ldots ] = 0
$$

This is a weaker condition than that $\{w_t\}$ is iid with $w_{t+1} \sim N(0,I)$.

### Examples

By appropriate choice of the primitives, a variety of dynamics can be represented in terms of the linear state space model.

The following examples help to highlight this point.

They also illustrate the wise dictum *finding the state is an art*.

(lss_sode)=
#### Second-order difference equation

Let $\{y_t\}$ be a deterministic sequence that satisfies

```{math}
:label: st_ex_1

y_{t+1} =  \phi_0 + \phi_1 y_t + \phi_2 y_{t-1}
\quad \text{s.t.} \quad
y_0, y_{-1} \text{ given}
```

To map {eq}`st_ex_1` into our state space system {eq}`st_space_rep`, we set

$$
x_t=
\begin{bmatrix}
    1 \\
    y_t \\
    y_{t-1}
\end{bmatrix}
\qquad
A = \begin{bmatrix}
          1 & 0 & 0 \\
          \phi_0 & \phi_1 & \phi_2  \\
          0 & 1 & 0
    \end{bmatrix}
\qquad
C= \begin{bmatrix}
    0 \\
    0 \\
    0
    \end{bmatrix}
\qquad
G = \begin{bmatrix} 0 & 1 & 0 \end{bmatrix}
$$

You can confirm that under these definitions, {eq}`st_space_rep` and {eq}`st_ex_1` agree.

The next figure shows dynamics of this process when $\phi_0 = 1.1, \phi_1=0.8, \phi_2 = -0.8, y_0 = y_{-1} = 1$

(lss_sode_fig)=
```{figure} /_static/figures/solution_lss_ex1.png
:width: 80%
```

Later you'll be asked to recreate this figure.

#### Univariate Autoregressive Processes

```{index} single: Linear State Space Models; Univariate Autoregressive Processes
```

We can use {eq}`st_space_rep` to represent the model

```{math}
:label: eq_ar_rep

y_{t+1} = \phi_1 y_{t} + \phi_2 y_{t-1} + \phi_3 y_{t-2} + \phi_4  y_{t-3} + \sigma w_{t+1}
```

where $\{w_t\}$ is iid and standard normal.

To put this in the linear state space format we take $x_t = \begin{bmatrix} y_t & y_{t-1} &  y_{t-2} &  y_{t-3} \end{bmatrix}'$ and

$$
A =
\begin{bmatrix}
    \phi_1 & \phi_2 & \phi_3 & \phi_4 \\
    1 & 0 & 0 & 0 \\
    0 & 1 & 0 & 0 \\
    0 & 0 & 1 & 0
\end{bmatrix}
\qquad
C = \begin{bmatrix}
        \sigma \\
        0 \\
        0 \\
        0
    \end{bmatrix}
\qquad
 G = \begin{bmatrix}
         1 & 0  & 0 & 0
     \end{bmatrix}
$$

The matrix $A$ has the form of the *companion matrix* to the vector
$\begin{bmatrix}\phi_1 &  \phi_2 & \phi_3 & \phi_4 \end{bmatrix}$.

The next figure shows dynamics of this process when

$$
\phi_1 = 0.5, \phi_2 = -0.2, \phi_3 = 0, \phi_4 = 0.5, \sigma = 0.2, y_0 = y_{-1} = y_{-2} =
y_{-3} = 1
$$

(lss_uap_fig)=
```{figure} /_static/figures/solution_lss_ex2.png
:width: 80%
```

#### Vector Autoregressions

```{index} single: Linear State Space Models; Vector Autoregressions
```

Now suppose that

* $y_t$ is a $k \times 1$ vector
* $\phi_j$ is a $k \times k$ matrix and
* $w_t$ is $k \times 1$

Then {eq}`eq_ar_rep` is termed a *vector autoregression*.

To map this into {eq}`st_space_rep`, we set

$$
x_t =
\begin{bmatrix}
     y_t \\
     y_{t-1} \\
     y_{t-2} \\
     y_{t-3}
 \end{bmatrix}
\quad
A =
\begin{bmatrix}
\phi_1 & \phi_2 & \phi_3 & \phi_4 \\
I & 0        & 0        & 0       \\
0 & I        & 0        & 0       \\
0 & 0        & I        & 0
\end{bmatrix}
\quad
C =
\begin{bmatrix}
     \sigma \\
     0 \\
     0 \\
     0
 \end{bmatrix}
\quad
G =
\begin{bmatrix}
     I & 0 & 0 & 0
 \end{bmatrix}
$$

where $I$ is the $k \times k$ identity matrix and $\sigma$ is a $k \times k$ matrix.

#### Seasonals

```{index} single: Linear State Space Models; Seasonals
```

We can use {eq}`st_space_rep` to represent

1. the *deterministic seasonal* $y_t = y_{t-4}$
1. the *indeterministic seasonal* $y_t = \phi_4 y_{t-4} + w_t$

In fact both are special cases of {eq}`eq_ar_rep`.

With the deterministic seasonal, the transition matrix becomes

$$
A = \begin{bmatrix}
        0 & 0 & 0 & 1 \\
        1 & 0 & 0 & 0 \\
        0 & 1 & 0 & 0 \\
        0 & 0 & 1 & 0
    \end{bmatrix}
$$

It is easy to check that $A^4 = I$, which implies that $x_t$ is strictly periodic with period 4:[^foot1]

$$
x_{t+4} = x_t
$$

Such an $x_t$ process can be used to model deterministic seasonals in quarterly time series.

The *indeterministic* seasonal produces recurrent, but aperiodic, seasonal fluctuations.

#### Time Trends

```{index} single: Linear State Space Models; Time Trends
```

The model $y_t = a t + b$ is known as a *linear time trend*.

We can represent this model in the linear state space form by taking

```{math}
:label: lss_ltt

A
= \begin{bmatrix}
    1 & 1  \\
    0 & 1
  \end{bmatrix}
\qquad
C
= \begin{bmatrix}
        0 \\
        0
  \end{bmatrix}
\qquad
G
= \begin{bmatrix}
        a & b
  \end{bmatrix}
```

and starting at initial condition $x_0 = \begin{bmatrix} 0 & 1\end{bmatrix}'$.

In fact it's possible to use the state-space system to represent polynomial trends of any order.

For instance, let

$$
x_0
= \begin{bmatrix}
        0 \\
        0 \\
        1
  \end{bmatrix}
\qquad
A
= \begin{bmatrix}
    1 & 1 & 0 \\
    0 & 1 & 1 \\
    0 & 0 & 1
  \end{bmatrix}
\qquad
C
= \begin{bmatrix}
        0 \\
        0 \\
        0
  \end{bmatrix}
$$

It follows that

$$
A^t =
\begin{bmatrix}
 1 & t & t(t-1)/2 \\
 0 & 1 & t \\
 0 & 0 & 1
\end{bmatrix}
$$

Then $x_t^\prime = \begin{bmatrix} t(t-1)/2 &t & 1 \end{bmatrix}$, so that $x_t$ contains
linear and quadratic time trends.

### Moving Average Representations

```{index} single: Linear State Space Models; Moving Average Representations
```

A nonrecursive expression for $x_t$ as a function of
$x_0, w_1, w_2, \ldots,  w_t$ can be found by using {eq}`st_space_rep` repeatedly to obtain

```{math}
:label: eqob5

\begin{aligned}
   x_t & = Ax_{t-1} + Cw_t \\
       & = A^2 x_{t-2} + ACw_{t-1} + Cw_t \nonumber \\
       & \qquad \vdots \nonumber \\
       & = \sum_{j=0}^{t-1} A^j Cw_{t-j}  + A^t x_0 \nonumber
\end{aligned}
```

Representation {eq}`eqob5` is a  *moving average* representation.

It expresses $\{x_t\}$ as a linear function of

1. current and past values of the  process $\{w_t\}$ and
1. the initial condition $x_0$

As an example of a moving average representation, let the model be

$$
A
= \begin{bmatrix}
    1 & 1  \\
    0 & 1
  \end{bmatrix}
\qquad
C
= \begin{bmatrix}
        1 \\
        0
  \end{bmatrix}
$$

You will be able to show that $A^t = \begin{bmatrix} 1 & t \cr 0 & 1 \end{bmatrix}$ and  $A^j C = \begin{bmatrix} 1 & 0 \end{bmatrix}'$.

Substituting into the moving average representation {eq}`eqob5`, we obtain

$$
x_{1t} = \sum_{j=0}^{t-1} w_{t-j} +
\begin{bmatrix}
 1 & t
\end{bmatrix}
x_0
$$

where $x_{1t}$ is the first entry of $x_t$.

The first term on the right is a cumulated sum of martingale differences, and is therefore a [martingale](https://en.wikipedia.org/wiki/Martingale_%28probability_theory%29).

The second term is a translated linear function of time.

For this reason, $x_{1t}$ is called a *martingale with drift*.

## Distributions and Moments

```{index} single: Linear State Space Models; Distributions
```

```{index} single: Linear State Space Models; Moments
```

### Unconditional Moments

Using {eq}`st_space_rep`, it's easy to obtain expressions for the
(unconditional) means of $x_t$ and $y_t$.

We'll explain what *unconditional* and *conditional* mean soon.

Letting $\mu_t := \mathbb{E} [x_t]$ and using linearity of expectations, we
find that

```{math}
:label: lss_mut_linear_models

\mu_{t+1} = A \mu_t
\quad \text{with} \quad \mu_0 \text{ given}
```

Here $\mu_0$ is a primitive given in {eq}`st_space_rep`.

The variance-covariance matrix of $x_t$ is $\Sigma_t := \mathbb{E} [ (x_t - \mu_t) (x_t - \mu_t)']$.

Using $x_{t+1} - \mu_{t+1} = A (x_t - \mu_t) + C w_{t+1}$, we can
determine this matrix recursively via

```{math}
:label: eqsigmalaw_linear_models

\Sigma_{t+1}  = A \Sigma_t A' + C C'
\quad \text{with} \quad \Sigma_0 \text{ given}
```

As with $\mu_0$, the matrix $\Sigma_0$ is a primitive given in {eq}`st_space_rep`.

As a matter of terminology, we will sometimes call

* $\mu_t$ the *unconditional mean*  of $x_t$
* $\Sigma_t$ the *unconditional variance-convariance matrix*  of $x_t$

This is to distinguish $\mu_t$ and $\Sigma_t$ from related objects that use conditioning
information, to be defined below.

However, you should be aware that these "unconditional" moments do depend on
the initial distribution $N(\mu_0, \Sigma_0)$.

#### Moments of the Observations

Using linearity of expectations again we have

```{math}
:label: lss_umy

\mathbb{E} [y_t] = \mathbb{E} [G x_t] = G \mu_t
```

The variance-covariance matrix of $y_t$ is easily shown to be

```{math}
:label: lss_uvy

\textrm{Var} [y_t] = \textrm{Var} [G x_t] = G \Sigma_t G'
```

### Distributions

```{index} single: Linear State Space Models; Distributions
```

In general, knowing the mean and variance-covariance matrix of a random vector
is not quite as good as knowing the full distribution.

However, there are some situations where these moments alone tell us all we
need to know.

These are situations in which the mean vector and covariance matrix are **sufficient statistics** for the population distribution.

(Sufficient statistics form a list of objects that characterize a population distribution)

One such situation is when the vector in question is Gaussian (i.e., normally
distributed).

This is the case here, given

1. our Gaussian assumptions on the primitives
1. the fact that normality is preserved under linear operations

In fact, it's [well-known](https://en.wikipedia.org/wiki/Multivariate_normal_distribution#Affine_transformation) that

```{math}
:label: lss_glig

u \sim N(\bar u, S)
\quad \text{and} \quad
v = a + B u
\implies
v \sim N(a + B \bar u, B S B')
```

In particular, given our Gaussian assumptions on the primitives and the
linearity of {eq}`st_space_rep` we can see immediately that  both $x_t$ and
$y_t$ are  Gaussian for all $t \geq 0$ [^fn_ag].

Since $x_t$ is Gaussian, to find the distribution, all we need to do is
find its mean and variance-covariance matrix.

But in fact we've already done this, in {eq}`lss_mut_linear_models` and {eq}`eqsigmalaw_linear_models`.

Letting $\mu_t$ and $\Sigma_t$ be as defined by these equations,
we have

```{math}
:label: lss_mgs_x

x_t \sim N(\mu_t, \Sigma_t)
```

By similar reasoning combined with {eq}`lss_umy` and {eq}`lss_uvy`,

```{math}
:label: lss_mgs_y

y_t \sim N(G \mu_t, G \Sigma_t G')
```

### Ensemble Interpretations

How should we interpret the distributions defined by {eq}`lss_mgs_x`--{eq}`lss_mgs_y`?

Intuitively, the probabilities in a distribution correspond to relative frequencies in a large population drawn from that distribution.

Let's apply this idea to our setting, focusing on the distribution of $y_T$ for fixed $T$.

We can generate independent draws of $y_T$ by repeatedly simulating the
evolution of the system up to time $T$, using an independent set of
shocks each time.

The next figure shows 20 simulations, producing 20 time series for $\{y_t\}$, and hence 20 draws of $y_T$.

The system in question is the univariate autoregressive model {eq}`eq_ar_rep`.

The values of $y_T$ are represented by black dots in the left-hand figure

```{figure} /_static/figures/tsh0.png

```

In the right-hand figure, these values are converted into a rotated histogram
that shows relative frequencies from our sample of 20 $y_T$'s.

(The parameters and source code for the figures can be found in file [linear_models/paths_and_hist.jl](https://github.com/QuantEcon/lecture-julia.myst/blob/main/lectures/_static/code/linear_models/paths_and_hist.jl))

Here is another figure, this time with 100 observations

```{figure} /_static/figures/tsh.png

```

Let's now try with 500,000 observations, showing only the histogram (without rotation)

```{figure} /_static/figures/tsh_hg.png
:width: 100%
```

The black line is the population density of $y_T$ calculated from {eq}`lss_mgs_y`.

The histogram and population distribution are close, as expected.

By looking at the figures and experimenting with parameters, you will gain a
feel for how the population distribution depends on the model primitives {ref}`listed above <lss_pgs>`, as intermediated by
the distribution's sufficient statistics.

#### Ensemble means

In the preceding figure we approximated the population distribution of $y_T$ by

1. generating $I$ sample paths (i.e., time series) where $I$ is a large number
1. recording each observation $y^i_T$
1. histogramming this sample

Just as the histogram approximates the population distribution, the *ensemble* or
*cross-sectional average*

$$
\bar y_T := \frac{1}{I} \sum_{i=1}^I y_T^i
$$

approximates the expectation $\mathbb{E} [y_T] = G \mu_T$ (as implied by the law of large numbers).

Here's a simulation comparing the ensemble averages and population means at time points $t=0,\ldots,50$.

The parameters are the same as for the preceding figures,
and the sample size is relatively small ($I=20$).

(lss_em_fig)=
```{figure} /_static/figures/ensemble_mean.png
:width: 70%
```

The ensemble mean for $x_t$ is

$$
\bar x_T := \frac{1}{I} \sum_{i=1}^I x_T^i \to \mu_T
\qquad (I \to \infty)
$$

The limit $\mu_T$ is a  "long-run average".

(By *long-run average* we mean the average for an infinite ($I = \infty$)  number of sample $x_T$'s)

Another application of the law of large numbers assures us that

$$
\frac{1}{I} \sum_{i=1}^I (x_T^i - \bar x_T) (x_T^i - \bar x_T)' \to \Sigma_T
\qquad (I \to \infty)
$$

### Joint Distributions

In the preceding discussion we looked at the distributions of $x_t$ and
$y_t$ in isolation.

This gives us useful information, but doesn't allow us to answer questions like

* what's the probability that $x_t \geq 0$ for all $t$?
* what's the probability that the process $\{y_t\}$ exceeds some value $a$ before falling below $b$?
* etc., etc.

Such questions concern the *joint distributions* of these sequences.

To compute the joint distribution of $x_0, x_1, \ldots, x_T$, recall
that joint and conditional densities are linked by the rule

$$
p(x, y) = p(y \, | \, x) p(x)
\qquad \text{(joint }=\text{ conditional }\times\text{ marginal)}
$$

From this rule we get $p(x_0, x_1) = p(x_1 \,|\, x_0) p(x_0)$.

The Markov property $p(x_t \,|\, x_{t-1}, \ldots, x_0) =  p(x_t \,|\, x_{t-1})$ and repeated applications of the preceding rule lead us to

$$
p(x_0, x_1, \ldots, x_T) =  p(x_0) \prod_{t=0}^{T-1} p(x_{t+1} \,|\, x_t)
$$

The marginal $p(x_0)$ is just the primitive $N(\mu_0, \Sigma_0)$.

In view of {eq}`st_space_rep`, the conditional densities are

$$
p(x_{t+1} \,|\, x_t) = N(Ax_t, C C')
$$

#### Autocovariance functions

An important object related to the joint distribution is the *autocovariance function*

```{math}
:label: eqnautodeff

\Sigma_{t+j, t} := \mathbb{E} [ (x_{t+j} - \mu_{t+j})(x_t - \mu_t)' ]
```

Elementary calculations show that

```{math}
:label: eqnautocov

\Sigma_{t+j,t} = A^j \Sigma_t
```

Notice that $\Sigma_{t+j,t}$ in general depends on both $j$, the gap between the two dates, and $t$, the earlier date.

## Stationarity and Ergodicity

```{index} single: Linear State Space Models; Stationarity
```

```{index} single: Linear State Space Models; Ergodicity
```

Stationarity and ergodicity are two properties  that, when they hold,  greatly aid analysis of linear state space models.

Let's start with the intuition.

### Visualizing Stability

Let's look at some more time series from the same model that we analyzed above.

This picture shows cross-sectional distributions for $y$ at times
$T, T', T''$

```{figure} /_static/figures/paths_and_stationarity.png
:width: 100%
```

Note how the time series "settle down" in the sense that the distributions at
$T'$ and $T''$ are relatively similar to each other --- but unlike
the distribution at $T$.

Apparently, the distributions of $y_t$  converge to a fixed long-run
distribution as $t \to \infty$.

When such a distribution exists it is called a *stationary distribution*.

### Stationary Distributions

In our setting, a distribution $\psi_{\infty}$ is said to be *stationary* for $x_t$ if

$$
x_t \sim \psi_{\infty}
\quad \text{and} \quad
x_{t+1} = A x_t + C w_{t+1}
\quad \implies \quad
x_{t+1} \sim \psi_{\infty}
$$

Since

1. in the present case all distributions are Gaussian
1. a Gaussian distribution is pinned down by its mean and variance-covariance matrix

we can restate the definition as follows: $\psi_{\infty}$ is stationary for $x_t$ if

$$
\psi_{\infty}
= N(\mu_{\infty}, \Sigma_{\infty})
$$

where $\mu_{\infty}$ and $\Sigma_{\infty}$ are fixed points of {eq}`lss_mut_linear_models` and {eq}`eqsigmalaw_linear_models` respectively.

### Covariance Stationary Processes

Let's see what happens to the preceding figure if we start $x_0$ at the stationary distribution.

(lss_s_fig)=
```{figure} /_static/figures/covariance_stationary.png
:width: 100%
```

Now the  differences in the observed distributions at $T, T'$ and $T''$ come entirely from random fluctuations due to the finite sample size.

By

* our choosing $x_0 \sim N(\mu_{\infty}, \Sigma_{\infty})$
* the definitions of $\mu_{\infty}$ and $\Sigma_{\infty}$ as fixed points of {eq}`lss_mut_linear_models` and {eq}`eqsigmalaw_linear_models` respectively

we've ensured that

$$
\mu_t = \mu_{\infty}
\quad \text{and} \quad
\Sigma_t = \Sigma_{\infty}
\quad \text{for all } t
$$

Moreover, in view of {eq}`eqnautocov`, the autocovariance function takes the form $\Sigma_{t+j,t} = A^j \Sigma_\infty$, which depends on $j$ but not on $t$.

This motivates the following definition.

A  process $\{x_t\}$ is said to be *covariance stationary* if

* both $\mu_t$ and $\Sigma_t$ are constant in $t$
* $\Sigma_{t+j,t}$ depends on the time gap $j$ but not on time $t$

In our setting, $\{x_t\}$ will be covariance stationary if $\mu_0, \Sigma_0, A, C$  assume values that  imply that none of $\mu_t, \Sigma_t, \Sigma_{t+j,t}$ depends on $t$.

### Conditions for Stationarity

#### The globally stable case

The difference equation $\mu_{t+1} = A \mu_t$ is known to have *unique*
fixed point $\mu_{\infty} = 0$ if all eigenvalues of $A$ have moduli strictly less than unity.

That is, if  `all(abs(eigvals(A)) .< 1) == true`.

The difference equation {eq}`eqsigmalaw_linear_models` also has a unique fixed point in this case, and, moreover

$$
\mu_t \to \mu_{\infty} = 0
\quad \text{and} \quad
\Sigma_t \to \Sigma_{\infty}
\quad \text{as} \quad t \to \infty
$$

regardless of the initial conditions $\mu_0$ and $\Sigma_0$.

```{only} html
This is the *globally stable case* --- see <a href=/_static/pdfs/iteration_notes.pdf download>these notes</a> for more a theoretical treatment
```

```{only} latex
This is the *globally stable case* --- see [these notes](https://lectures.quantecon.org/_downloads/iteration_notes.pdf) for more a theoretical treatment
```

However, global stability is more than we need for stationary solutions, and often more than we want.

To illustrate, consider {ref}`our second order difference equation example <lss_sode>`.

Here the state is $x_t = \begin{bmatrix} 1 & y_t & y_{t-1} \end{bmatrix}'$.

Because of the constant first component in the state vector, we will never have $\mu_t \to 0$.

How can we find stationary solutions that respect a constant state component?

#### Processes with a constant state component

To investigate such a process, suppose that $A$ and $C$ take the
form

$$
A
 = \begin{bmatrix}
    A_1 & a \\
    0 & 1
\end{bmatrix}
 \qquad
 C  = \begin{bmatrix}
     C_1 \\
     0
\end{bmatrix}
$$

where

* $A_1$ is an $(n-1) \times (n-1)$ matrix
* $a$ is an $(n-1) \times 1$ column vector

Let $x_t = \begin{bmatrix} x_{1t}' & 1 \end{bmatrix}'$ where $x_{1t}$ is $(n-1) \times 1$.

It follows  that

$$
\begin{aligned}
x_{1,t+1} & = A_1 x_{1t} + a + C_1 w_{t+1} \\
\end{aligned}
$$

Let $\mu_{1t} = \mathbb{E} [x_{1t}]$ and take expectations on both sides of this expression to get

```{math}
:label: eqob29

\mu_{1,t+1} = A_1 \mu_{1,t} + a
```

Assume now that the moduli of the eigenvalues of $A_1$ are all strictly less than one.

Then {eq}`eqob29` has a unique stationary solution, namely,

$$
\mu_{1\infty} = (I-A_1)^{-1} a
$$

The stationary value of $\mu_t$ itself is then $\mu_\infty := \begin{bmatrix}
\mu_{1\infty}' & 1 \end{bmatrix}'$.

The stationary values of $\Sigma_t$ and $\Sigma_{t+j,t}$ satisfy

```{math}
:label: eqnSigmainf

\begin{aligned}
\Sigma_\infty  & = A \Sigma_\infty A' + C C' \\
\Sigma_{t+j,t} & = A^j \Sigma_\infty \nonumber
\end{aligned}
```

Notice that here $\Sigma_{t+j,t}$ depends on the time gap $j$ but not on calendar time $t$.

In conclusion, if

* $x_0 \sim N(\mu_{\infty}, \Sigma_{\infty})$ and
* the moduli of the eigenvalues of $A_1$ are all strictly less than unity

then the $\{x_t\}$ process is covariance stationary, with constant state
component

```{note}
If the eigenvalues of $A_1$ are less than unity in modulus, then
(a) starting from any initial value, the mean and variance-covariance
matrix both converge to their stationary values; and (b)
iterations on {eq}`eqsigmalaw_linear_models` converge to the fixed point of the *discrete
Lyapunov equation* in the first line of {eq}`eqnSigmainf`.
```

### Ergodicity

Let's suppose that we're working with a covariance stationary process.

In this case we know that the ensemble mean will converge to $\mu_{\infty}$ as the sample size $I$ approaches infinity.

#### Averages over time

Ensemble averages across simulations are interesting theoretically, but in real life we usually observe only a *single* realization $\{x_t, y_t\}_{t=0}^T$.

So now let's take a single realization and form the time series averages

$$
\bar x := \frac{1}{T} \sum_{t=1}^T x_t
\quad \text{and} \quad
\bar y := \frac{1}{T} \sum_{t=1}^T y_t
$$

Do these time series averages converge to something interpretable in terms of our basic state-space representation?

The answer depends on something called *ergodicity*.

Ergodicity is the property that time series and ensemble averages coincide.

More formally, ergodicity implies that time series sample averages converge to their
expectation under the stationary distribution.

In particular,

* $\frac{1}{T} \sum_{t=1}^T x_t \to \mu_{\infty}$
* $\frac{1}{T} \sum_{t=1}^T (x_t -\bar x_T) (x_t - \bar x_T)' \to \Sigma_\infty$
* $\frac{1}{T} \sum_{t=1}^T (x_{t+j} -\bar x_T) (x_t - \bar x_T)' \to A^j \Sigma_\infty$

In our linear Gaussian setting, any covariance stationary process is also ergodic.

## Noisy Observations

In some settings the observation equation $y_t = Gx_t$ is modified to
include an error term.

Often this error term represents the idea that the true state can only be
observed imperfectly.

To include an error term in the observation we introduce

* An iid sequence of $\ell \times 1$ random vectors $v_t \sim N(0,I)$
* A $k \times \ell$ matrix $H$

and extend the linear state-space system to

```{math}
:label: st_space_rep_noisy

\begin{aligned}
    x_{t+1} & =  A x_t + C w_{t+1}   \\
    y_t &  =  G x_t + H v_t \nonumber \\
    x_0 & \sim N(\mu_0, \Sigma_0) \nonumber
\end{aligned}
```

The sequence $\{v_t\}$ is assumed to be independent of $\{w_t\}$.

The process $\{x_t\}$ is not modified by noise in the observation
equation and its moments, distributions and stability properties remain the same.

The unconditional moments of $y_t$ from {eq}`lss_umy` and {eq}`lss_uvy`
now become

```{math}
:label: lss_umy_2

\mathbb{E} [y_t] = \mathbb{E} [G x_t + H v_t] = G \mu_t
```

The variance-covariance matrix of $y_t$ is easily shown to be

```{math}
:label: lss_uvy_2

\textrm{Var} [y_t] = \textrm{Var} [G x_t + H v_t] = G \Sigma_t G' + HH'
```

The distribution of $y_t$ is therefore

$$
y_t \sim N(G \mu_t, G \Sigma_t G' + HH')
$$

## Prediction

```{index} single: Linear State Space Models; Prediction
```

The theory of prediction for linear state space systems is elegant and
simple.

(ff_cm)=
### Forecasting Formulas -- Conditional Means

The natural way to predict variables is to use conditional distributions.

For example, the optimal forecast of $x_{t+1}$ given information known at time $t$ is

$$
\mathbb{E}_t [x_{t+1}] := \mathbb{E} [x_{t+1} \mid x_t, x_{t-1}, \ldots, x_0 ] = Ax_t
$$

The right-hand side follows from $x_{t+1} = A x_t + C w_{t+1}$ and the
fact that $w_{t+1}$ is zero mean and independent of $x_t, x_{t-1}, \ldots, x_0$.

That $\mathbb{E}_t [x_{t+1}] = \mathbb{E}[x_{t+1} \mid x_t]$ is an implication of $\{x_t\}$ having the *Markov property*.

The one-step-ahead forecast error is

$$
x_{t+1} - \mathbb{E}_t [x_{t+1}] = Cw_{t+1}
$$

The covariance matrix of the forecast error is

$$
\mathbb{E} [ (x_{t+1} - \mathbb{E}_t [ x_{t+1}] ) (x_{t+1} - \mathbb{E}_t [ x_{t+1}])'] = CC'
$$

More generally, we'd like to compute the $j$-step ahead forecasts $\mathbb{E}_t [x_{t+j}]$ and $\mathbb{E}_t [y_{t+j}]$.

With a bit of algebra we obtain

$$
x_{t+j} = A^j x_t + A^{j-1} C w_{t+1} + A^{j-2} C w_{t+2}
+ \cdots + A^0 C w_{t+j}
$$

In view of the iid property, current and past state values provide no information about future values of the shock.

Hence $\mathbb{E}_t[w_{t+k}] = \mathbb{E}[w_{t+k}] = 0$.

It now follows from linearity of expectations that the $j$-step ahead forecast of $x$ is

$$
\mathbb{E}_t [x_{t+j}] = A^j x_t
$$

The $j$-step ahead forecast of $y$ is therefore

$$
\mathbb{E}_t [y_{t+j}]
= \mathbb{E}_t [G x_{t+j} + H v_{t+j}]
= G A^j x_t
$$

### Covariance of Prediction Errors

It is useful to obtain the covariance matrix of the vector of  $j$-step-ahead prediction errors

```{math}
:label: eqob8

x_{t+j} - \mathbb{E}_t [ x_{t+j}] = \sum^{j-1}_{s=0} A^s C w_{t-s+j}
```

Evidently,

```{math}
:label: eqob9a

V_j := \mathbb{E}_t [ (x_{t+j} - \mathbb{E}_t [x_{t+j}] ) (x_{t+j} - \mathbb{E}_t [x_{t+j}] )^\prime ] =   \sum^{j-1}_{k=0} A^k C C^\prime A^{k^\prime}
```

$V_j$ defined in {eq}`eqob9a` can be calculated recursively via $V_1 = CC'$ and

```{math}
:label: eqob9b

V_j = CC^\prime + A V_{j-1} A^\prime, \quad j \geq 2
```

$V_j$ is the *conditional covariance matrix* of the errors in forecasting
$x_{t+j}$, conditioned on time $t$ information $x_t$.

Under particular conditions, $V_j$ converges to

```{math}
:label: eqob10

V_\infty = CC' + A V_\infty A'
```

Equation {eq}`eqob10` is an example of a *discrete Lyapunov* equation in the covariance matrix $V_\infty$.

A sufficient condition for $V_j$ to converge is that the eigenvalues of $A$ be strictly less than one in modulus.

Weaker sufficient conditions for convergence  associate eigenvalues equaling or exceeding one in modulus with elements of $C$ that equal $0$.

(lm_fgs)=
### Forecasts of Geometric Sums

In several contexts, we want to compute forecasts of  geometric sums of future random variables governed by the linear state-space system {eq}`st_space_rep`.

We want the following objects

* Forecast of a geometric sum of future $x$'s, or $\mathbb{E}_t \left[ \sum_{j=0}^\infty \beta^j x_{t+j} \right]$.
* Forecast of a geometric sum of future $y$'s, or $\mathbb{E}_t \left[\sum_{j=0}^\infty \beta^j y_{t+j} \right]$.

These objects are important components of some famous and  interesting dynamic models.

For example,

* if $\{y_t\}$ is a stream of dividends, then $\mathbb{E} \left[\sum_{j=0}^\infty \beta^j y_{t+j} | x_t \right]$ is a model of a stock price
* if $\{y_t\}$ is  the money supply, then $\mathbb{E} \left[\sum_{j=0}^\infty \beta^j y_{t+j} | x_t \right]$ is a  model of the price level

#### Formulas

Fortunately, it is easy to use a little matrix algebra to compute these objects.

Suppose that every eigenvalue of $A$ has modulus strictly less than $\frac{1}{\beta}$.

It {ref}`then follows <la_neumann_remarks>` that $I + \beta A + \beta^2 A^2 + \cdots = \left[I - \beta A \right]^{-1}$.

This leads to our formulas:

* Forecast of a geometric sum of future $x$'s

$$
\mathbb{E}_t \left[\sum_{j=0}^\infty \beta^j x_{t+j} \right]
= [I + \beta A + \beta^2 A^2 + \cdots \ ] x_t = [I - \beta A]^{-1} x_t
$$

* Forecast of a geometric sum of future $y$'s

$$
\mathbb{E}_t \left[\sum_{j=0}^\infty \beta^j y_{t+j} \right]
= G [I + \beta A + \beta^2 A^2 + \cdots \ ] x_t
= G[I - \beta A]^{-1} x_t
$$

## Code

Our preceding simulations and calculations are based on code in
the file [lss.jl](https://github.com/QuantEcon/QuantEcon.jl/blob/master/src/lss.jl) from the [QuantEcon.jl](http://quantecon.org/quantecon-jl) package.

The code implements a type which the linear state space models can act on directly through specific methods (for simulations, calculating moments, etc.).

Examples of usage are given in the solutions to the exercises.

## Exercises

(lss_ex1)=
### Exercise 1

Replicate {ref}`this figure <lss_sode_fig>` using the `LSS` type from `lss.jl`.

(lss_ex2)=
### Exercise 2

Replicate {ref}`this figure <lss_uap_fig>` modulo randomness using the same type.

(lss_ex3)=
### Exercise 3

Replicate {ref}`this figure <lss_em_fig>` modulo randomness using the same type.

The state space model and parameters are the same as for the preceding exercise.

(lss_ex4)=
### Exercise 4

Replicate {ref}`this figure <lss_s_fig>` modulo randomness using the same type.

The state space model and parameters are the same as for the preceding exercise, except that the initial condition is the stationary distribution.

Hint: You can use the `stationary_distributions` method to get the initial conditions.

The number of sample paths is 80, and the time horizon in the figure is 100.

Producing the vertical bars and dots is optional, but if you wish to try,
the bars are at dates 10, 50 and 75.

## Solutions

```{code-cell} julia
---
tags: [remove-cell]
---
using Test
```

```{code-cell} julia
using LaTeXStrings, QuantEcon, Plots

```

### Exercise 1

```{code-cell} julia
phi0, phi1, phi2 = 1.1, 0.8, -0.8

A = [1.0 0.0 0
     phi0 phi1 phi2
     0.0 1.0 0.0]
C = zeros(3, 1)
G = [0.0 1.0 0.0]
mu_0 = ones(3)

lss = LSS(A, C, G; mu_0)

x, y = simulate(lss, 50)
plot(dropdims(y, dims = 1), color = :blue, linewidth = 2, alpha = 0.7)
plot!(xlabel = "time", ylabel = L"y_t", legend = :none)
```

```{code-cell} julia
---
tags: [remove-cell]
---
@testset "Testing Exercise 1" begin
    @test x[10] ≈ 1.
    #test y[13] ≈ 1.0889186488319997 atol = 1e-6
end
```

### Exercise 2

```{code-cell} julia
using Random
Random.seed!(42) # For deterministic results.

phi1, phi2, phi3, phi4 = 0.5, -0.2, 0, 0.5
sigma = 0.2

A = [phi1 phi2 phi3 phi4
     1.0 0.0 0.0 0.0
     0.0 1.0 0.0 0.0
     0.0 0.0 1.0 0.0]
C = [sigma
     0.0
     0.0
     0.0]
G = [1.0 0.0 0.0 0.0]

ar = LSS(A, C, G; mu_0 = ones(4))
x, y = simulate(ar, 200)

plot(dropdims(y, dims = 1), color = :blue, linewidth = 2, alpha = 0.7)
plot!(xlabel = "time", ylabel = L"y_t", legend = :none)
```

```{code-cell} julia
---
tags: [remove-cell]
---
@testset "Exercise 2 tests" begin
    #test x[1, 2] ≈ 1.155572219611465 atol = 1e-10
    #test x[1, :]' == y
end
```

### Exercise 3

```{code-cell} julia
---
tags: [remove-cell]
---
Random.seed!(42);
```

```{code-cell} julia
phi1, phi2, phi3, phi4 = 0.5, -0.2, 0, 0.5
sigma = 0.1

A = [phi1 phi2 phi3 phi4
     1.0 0.0 0.0 0.0
     0.0 1.0 0.0 0.0
     0.0 0.0 1.0 0.0]
C = [sigma
     0.0
     0.0
     0.0]
G = [1.0 0.0 0.0 0.0]
I = 20
T = 50
ar = LSS(A, C, G; mu_0 = ones(4))
ymin, ymax = -0.5, 1.15

ensemble_mean = zeros(T)
ys = []
for i in 1:I
    x, y = simulate(ar, T)
    y = dropdims(y, dims = 1)
    push!(ys, y)
    ensemble_mean .+= y
end

ensemble_mean = ensemble_mean ./ I
plot(ys, color = :blue, alpha = 0.2, linewidth = 0.8, label = "")
plot!(ensemble_mean, color = :blue, linewidth = 2, label = L"\overline{y_t}")
m = moment_sequence(ar)
pop_means = zeros(0)
for (i, t) in enumerate(m)
    (mu_x, mu_y, Sigma_x, Sigma_y) = t
    push!(pop_means, mu_y[1])
    i == 50 && break
end
plot!(pop_means, color = :green, linewidth = 2, label = L"G \mu_t")
plot!(ylims = (ymin, ymax), xlabel = "time", ylabel = L"y_t", legendfont = font(12))
```

```{code-cell} julia
---
tags: [remove-cell]
---
@testset "Exercise 3 tests" begin
    #test pop_means[3] ≈ 0.7
    #test m.lss.A[1, 2] ≈ -0.2
    #test pop_means[end] ≈ 0.01665357418047883
end
```

### Exercise 4

```{code-cell} julia
---
tags: [remove-cell]
---
Random.seed!(42);
```

```{code-cell} julia
phi1, phi2, phi3, phi4 = 0.5, -0.2, 0, 0.5
sigma = 0.1

A = [phi1 phi2 phi3 phi4
     1.0 0.0 0.0 0.0
     0.0 1.0 0.0 0.0
     0.0 0.0 1.0 0.0]
C = [sigma
     0.0
     0.0
     0.0]
G = [1.0 0.0 0.0 0.0]

T0 = 10
T1 = 50
T2 = 75
T4 = 100

ar = LSS(A, C, G; mu_0 = ones(4))
ymin, ymax = -0.6, 0.6

mu_x, mu_y, Sigma_x, Sigma_y = stationary_distributions(ar)
ar = LSS(A, C, G; mu_0 = mu_x, Sigma_0 = Sigma_x)
colors = ["c", "g", "b"]

ys = []
x_scatter = []
y_scatter = []
for i in 1:80
    rcolor = colors[rand(1:3)]
    x, y = simulate(ar, T4)
    y = dropdims(y, dims = 1)
    push!(ys, y)
    x_scatter = [x_scatter; T0; T1; T2]
    y_scatter = [y_scatter; y[T0]; y[T1]; y[T2]]
end

plot(ys, linewidth = 0.8, alpha = 0.5)
plot!([T0 T1 T2; T0 T1 T2], [-1 -1 -1; 1 1 1], color = :black, legend = :none)
scatter!(x_scatter, y_scatter, color = :black, alpha = 0.5)
plot!(ylims = (ymin, ymax), ylabel = L"y_t", xticks = [], yticks = ymin:0.2:ymax)
plot!(annotations = [(T0 + 1, -0.55, L"T"); (T1 + 1, -0.55, L"T^\prime");
                     (T2 + 1, -0.55, L"T^{\prime\prime}")])
```

```{code-cell} julia
---
tags: [remove-cell]
---
@testset "Exercise 4 Tests" begin
    # #test ys[4][7] ≈ 0.011484225879516554 atol = 1e-3
    #test ar.A[1, 4] ≈ 0.5 atol = 1e-3
    #test Sigma_y[1] ≈ 0.020833333252863163 atol = 1e-3
end
```

[^foot1]: The eigenvalues of $A$ are $(1,-1, i,-i)$.

[^fn_ag]: The correct way to argue this is by induction.  Suppose that
$x_t$ is Gaussian.  Then {eq}`st_space_rep` and
{eq}`lss_glig` imply that $x_{t+1}$ is Gaussian.  Since $x_0$
is assumed to be Gaussian, it follows that every $x_t$ is Gaussian.
Evidently this implies that each $y_t$ is Gaussian.

