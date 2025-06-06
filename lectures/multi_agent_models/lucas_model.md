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

(lucas_asset)=
```{raw} html
<div id="qe-notebook-header" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# Asset Pricing II: The Lucas Asset Pricing Model

```{index} single: Models; Lucas Asset Pricing
```

```{contents} Contents
:depth: 2
```

## Overview

As stated in an {doc}`earlier lecture <../multi_agent_models/markov_asset>`, an asset is a claim on a stream of prospective payments.

What is the correct price to pay for such a claim?

The elegant asset pricing model of Lucas {cite}`Lucas1978` attempts to answer this question in an equilibrium setting with risk averse agents.

While we mentioned some consequences of Lucas' model {ref}`earlier <mass_pra>`, it is now time to work through the model more carefully, and try to understand where the fundamental asset pricing equation comes from.

A side benefit of studying Lucas' model is that it provides a beautiful illustration of model building in general and equilibrium pricing in competitive models in particular.

Another difference to our {doc}`first asset pricing lecture <../multi_agent_models/markov_asset>` is that the state space and shock will be continous rather than discrete.

## The Lucas Model

```{index} single: Lucas Model
```

Lucas studied a pure exchange economy with a representative consumer (or household), where

* *Pure exchange* means that all endowments are exogenous.
* *Representative* consumer means that either
    * there is a single consumer (sometimes also referred to as a household), or
    * all consumers have identical endowments and preferences

Either way, the assumption of a representative agent means that prices adjust to eradicate desires to trade.

This makes it very easy to compute competitive equilibrium prices.

### Basic Setup

Let's review the set up.

#### Assets

```{index} single: Lucas Model; Assets
```

There is a single "productive unit" that costlessly generates a sequence of consumption goods $\{y_t\}_{t=0}^{\infty}$.

Another way to view $\{y_t\}_{t=0}^{\infty}$ is as a *consumption endowment* for this economy.

We will assume that this endowment is Markovian, following the exogenous process

$$
y_{t+1} = G(y_t, \xi_{t+1})
$$

Here $\{ \xi_t \}$ is an iid shock sequence with known distribution $\phi$ and $y_t \geq 0$.

An asset is a claim on all or part of this endowment stream.

The consumption goods $\{y_t\}_{t=0}^{\infty}$ are nonstorable, so holding assets is the only way to transfer wealth into the future.

For the purposes of intuition, it's common to think of the productive unit as a "tree" that produces fruit.

Based on this idea, a "Lucas tree" is a claim on the consumption endowment.

#### Consumers

```{index} single: Lucas Model; Consumers
```

A representative consumer ranks consumption streams $\{c_t\}$ according to the time separable utility functional

```{math}
:label: lt_uf

\mathbb{E} \sum_{t=0}^\infty \beta^t u(c_t)
```

Here

* $\beta \in (0,1)$ is a fixed discount factor
* $u$ is a strictly increasing, strictly concave, continuously differentiable period utility function
* $\mathbb{E}$ is a mathematical expectation

### Pricing a Lucas Tree

```{index} single: Lucas Model; Pricing
```

What is an appropriate price for a claim on the consumption endowment?

We'll price an *ex dividend* claim, meaning that

* the seller retains this period's dividend
* the buyer pays $p_t$ today to purchase a claim on
    * $y_{t+1}$ and
    * the right to sell the claim tomorrow at price $p_{t+1}$

Since this is a competitive model, the first step is to pin down consumer
behavior, taking prices as given.

Next we'll impose equilibrium constraints and try to back out prices.

In the consumer problem, the consumer's control variable is the share $\pi_t$ of the claim held in each period.

Thus, the consumer problem is to maximize {eq}`lt_uf` subject to

$$
c_t + \pi_{t+1} p_t \leq \pi_t y_t + \pi_t p_t
$$

along with $c_t \geq 0$ and $0 \leq \pi_t \leq 1$ at each $t$.

The decision to hold share $\pi_t$ is actually made at time $t-1$.

But this value is inherited as a state variable at time $t$, which explains the choice of subscript.

#### The dynamic program

```{index} single: Lucas Model; Dynamic Program
```

We can write the consumer problem as a dynamic programming problem.

Our first observation is that prices depend on current information, and current information is really just the endowment process up until the current period.

In fact the endowment process is Markovian, so that the only relevant
information is the current state $y \in \mathbb R_+$ (dropping the time subscript).

This leads us to guess an equilibrium where price is a function $p$ of $y$.

Remarks on the solution method

* Since this is a competitive (read: price taking) model, the consumer will take this function $p$ as .
* In this way we determine consumer behavior given $p$ and then use equilibrium conditions to recover $p$.
* This is the standard way to solve competitive equilibrum models.

Using the assumption that price is a given function $p$ of $y$, we write the value function and constraint as

$$
v(\pi, y) = \max_{c, \pi'}
    \left\{
        u(c) + \beta \int v(\pi', G(y, z)) \phi(dz)
    \right\}
$$

subject to

```{math}
:label: preltbe

c + \pi' p(y) \leq \pi y + \pi p(y)
```

We can invoke the fact that utility is increasing to claim equality in {eq}`preltbe` and hence eliminate the constraint, obtaining

```{math}
:label: ltbe

v(\pi, y) = \max_{\pi'}
    \left\{
        u[\pi (y + p(y)) - \pi' p(y) ] + \beta \int v(\pi', G(y, z)) \phi(dz)
    \right\}
```

The solution to this dynamic programming problem is an optimal policy expressing either $\pi'$ or $c$ as a function of the state $(\pi, y)$.

* Each one determines the other, since $c(\pi, y) = \pi (y + p(y))- \pi' (\pi, y) p(y)$.

#### Next steps

What we need to do now is determine equilibrium prices.

It seems that to obtain these, we will have to

1. Solve this two dimensional dynamic programming problem for the optimal policy.
1. Impose equilibrium constraints.
1. Solve out for the price function $p(y)$ directly.

However, as Lucas showed, there is a related but more straightforward way to do this.

#### Equilibrium constraints

```{index} single: Lucas Model; Equilibrium Constraints
```

Since the consumption good is not storable, in equilibrium we must have $c_t = y_t$ for all $t$.

In addition, since there is one representative consumer (alternatively, since
all consumers are identical), there should be no trade in equilibrium.

In particular, the representative consumer owns the whole tree in every period, so $\pi_t = 1$ for all $t$.

Prices must adjust to satisfy these two constraints.

#### The equilibrium price function

```{index} single: Lucas Model; Equilibrium Price Function
```

Now observe that the first order condition for {eq}`ltbe` can be written as

$$
u'(c)  p(y) = \beta \int v_1'(\pi', G(y, z)) \phi(dz)
$$

where $v'_1$ is the derivative of $v$ with respect to its first argument.

To obtain $v'_1$ we can simply differentiate the right hand side of
{eq}`ltbe` with respect to $\pi$, yielding

$$
v'_1(\pi, y) = u'(c) (y + p(y))
$$

Next we impose the equilibrium constraints while combining the last two
equations to get

```{math}
:label: lteeq

p(y)  = \beta \int \frac{u'[G(y, z)]}{u'(y)} [G(y, z) + p(G(y, z))]  \phi(dz)
```

In sequential rather than functional notation, we can also write this as

```{math}
:label: lteeqs

p_t = \mathbb{E}_t \left[ \beta \frac{u'(c_{t+1})}{u'(c_t)} ( y_{t+1} + p_{t+1} ) \right]
```

This is the famous consumption-based asset pricing equation.

Before discussing it further we want to solve out for prices.

### Solving the Model

```{index} single: Lucas Model; Solving
```

Equation {eq}`lteeq` is a *functional equation* in the unknown function $p$.

The solution is an equilibrium price function $p^*$.

Let's look at how to obtain it.

#### Setting up the problem

Instead of solving for it directly we'll follow Lucas' indirect approach, first setting

```{math}
:label: ltffp

f(y) := u'(y) p(y)
```

so that {eq}`lteeq` becomes

```{math}
:label: lteeq2

f(y) = h(y) + \beta \int f[G(y, z)] \phi(dz)
```

Here $h(y) := \beta \int u'[G(y, z)] G(y, z)  \phi(dz)$ is a function that
depends only on the primitives.

Equation {eq}`lteeq2` is a functional equation in $f$.

The plan is to solve out for $f$ and convert back to $p$ via {eq}`ltffp`.

To solve {eq}`lteeq2` we'll use a standard method: convert it to a fixed point problem.

First we introduce the operator $T$ mapping $f$ into $Tf$ as defined by

```{math}
:label: lteeqT

(Tf)(y) = h(y) + \beta \int f[G(y, z)] \phi(dz)
```

The reason we do this is that a solution to {eq}`lteeq2` now corresponds to a
function $f^*$ satisfying $(Tf^*)(y) = f^*(y)$ for all $y$.

In other words, a solution is a *fixed point* of $T$.

This means that we can use fixed point theory to obtain and compute the solution.

#### A little fixed point theory

```{index} single: Fixed Point Theory
```

Let $cb\mathbb{R}_+$ be the set of continuous bounded functions $f \colon \mathbb{R}_+ \to \mathbb{R}_+$.

We now show that

1. $T$ has exactly one fixed point $f^*$ in $cb\mathbb{R}_+$.
1. For any $f \in cb\mathbb{R}_+$, the sequence $T^k f$ converges
   uniformly to $f^*$.

(Note: If you find the mathematics heavy going you can take 1--2 as given and skip to the {ref}`next section <lt_comp_eg>`)

Recall the [Banach contraction mapping theorem](https://en.wikipedia.org/wiki/Banach_fixed-point_theorem).

It tells us that the previous statements will be true if we can find an
$\alpha < 1$ such that

```{math}
:label: ltbc

\| Tf - Tg \| \leq \alpha \| f - g \|,
\qquad \forall \, f, g \in cb\mathbb{R}_+
```

Here $\|h\| := \sup_{x \in \mathbb{R}_+} |h(x)|$.

To see that {eq}`ltbc` is valid, pick any $f,g \in cb\mathbb{R}_+$ and any $y \in \mathbb{R}_+$.

Observe that, since integrals get larger when absolute values are moved to the
inside,

$$
\begin{aligned}
    |Tf(y) - Tg(y)|
    & = \left| \beta \int f[G(y, z)] \phi(dz)
        -  \beta \int g[G(y, z)] \phi(dz) \right|
    \\
    & \leq \beta \int \left| f[G(y, z)] -  g[G(y, z)] \right| \phi(dz)
    \\
    & \leq \beta \int \| f -  g \| \phi(dz)
    \\
    & = \beta  \| f -  g \|
\end{aligned}
$$

Since the right hand side is an upper bound, taking the sup over all $y$
on the left hand side gives {eq}`ltbc` with $\alpha := \beta$.

(lt_comp_eg)=
### Computation -- An Example

```{index} single: Lucas Model; Computation
```

The preceding discussion tells that we can compute $f^*$ by picking any arbitrary $f \in cb\mathbb{R}_+$ and then iterating with $T$.

The equilibrium price function $p^*$ can then be recovered by $p^*(y) = f^*(y) / u'(y)$.

Let's try this when $\ln y_{t+1} = \alpha \ln y_t + \sigma \epsilon_{t+1}$ where $\{\epsilon_t\}$ is iid and standard normal.

Utility will take the isoelastic form $u(c) = c^{1-\gamma}/(1-\gamma)$, where $\gamma > 0$ is the coefficient of relative risk aversion.

Some code to implement the iterative computational procedure can be found below:


```{code-cell} julia
---
tags: [remove-cell]
---
using Test
```

```{code-cell} julia
using LinearAlgebra, Statistics, Random
using Distributions, Interpolations, LaTeXStrings, Plots, NLsolve

```

```{code-cell} julia
function LucasTree(; gamma = 2.0,
                   beta = 0.95,
                   alpha = 0.9,
                   sigma = 0.1,
                   grid_size = 100,
                   num_z = 500)
    phi = LogNormal(0.0, sigma)
    z = rand(phi, num_z)

    # build a grid with mass around stationary distribution
    ssd = sigma / sqrt(1 - alpha^2)
    grid_min = exp(-4 * ssd)
    grid_max = exp(4 * ssd)
    grid = range(grid_min, grid_max, length = grid_size)

    # set h(y) = beta * int u'(G(y,z)) G(y,z) phi(dz)
    h = similar(grid)
    for (i, y) in enumerate(grid)
        h[i] = beta * mean((y^alpha .* z) .^ (1 - gamma))
    end

    return (; gamma, beta, alpha, sigma, phi, grid, z, h)
end

# get equilibrium price for Lucas tree
function solve_lucas_model(lt; ftol = 1e-8, iterations = 500)
    (; grid, gamma, alpha, beta, h, z) = lt

    # approximate Lucas operator, which returns the updated function Tf on the grid
    function T(f)
        Af = linear_interpolation(grid, f, extrapolation_bc = Line())
        # Using z for monte-carlo integration
        Tf = [h[i] + beta * mean(Af.(grid[i]^alpha .* z))
              for i in 1:length(grid)]
        return Tf
    end

    sol = fixedpoint(T, zero(grid); ftol, iterations)
    converged(sol) || error("Failed to converge in $(sol.iterations) iter")
    f = sol.zero

    price = f .* grid .^ gamma # f(y)*y^gamma

    return price
end
```

An example of usage is given in the docstring and repeated here

```{code-cell} julia
Random.seed!(42) # For reproducible results.

tree = LucasTree(; gamma = 2.0, beta = 0.95, alpha = 0.90, sigma = 0.1)
price_vals = solve_lucas_model(tree);
```

```{code-cell} julia
---
tags: [remove-cell]
---
@testset begin
    @test price_vals[57] ≈ 44.53930835369383
    @test price_vals[78] ≈ 68.48080295548888
    @test price_vals[13] ≈ 9.886241027004147
end
```

Here's the resulting price function

```{code-cell} julia
plot(tree.grid, price_vals, lw = 2, label = L"p^*(y)")
plot!(xlabel = L"y", ylabel = "price", legend = :topleft)
```

The price is increasing, even if we remove all serial correlation from the endowment process.

The reason is that a larger current endowment reduces current marginal
utility.

The price must therefore rise to induce the household to consume the entire endowment (and hence satisfy the resource constraint).

What happens with a more patient consumer?

Here the orange line corresponds to the previous parameters and the green line is price when $\beta = 0.98$.

(mass_lt_cb)=
```{figure} /_static/figures/solution_mass_ex2.png
:width: 80%
```

We see that when consumers are more patient the asset becomes more valuable, and the price of the Lucas tree shifts up.

Exercise 1 asks you to replicate this figure.

## Exercises

(lucas_asset_ex1)=
### Exercise 1

Replicate {ref}`the figure <mass_lt_cb>` to show how discount rates affect prices.

## Solutions

```{code-cell} julia
---
tags: [remove-cell]
---
Random.seed!(42);
```

```{code-cell} julia
plot()
for beta in (0.95, 0.98)
    tree = LucasTree(; beta)
    grid = tree.grid
    price_vals = solve_lucas_model(tree)
    plot!(grid, price_vals, lw = 2, label = L"\beta = %$beta")
end

plot!(xlabel = L"y", ylabel = "price", legend = :topleft)
```

```{code-cell} julia
---
tags: [remove-cell]
---
@testset begin # For the 0.98, since the other one is overwritten.
    Random.seed!(42)
    price_vals = solve_lucas_model(LucasTree(beta = 0.98))
    @test price_vals[20] ≈ 35.03700398163009
    @test price_vals[57] ≈ 124.46814606174088
end
```

