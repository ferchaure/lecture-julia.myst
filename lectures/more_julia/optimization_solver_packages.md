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

(optimization_solver_packages)=
```{raw} html
<div id="qe-notebook-header" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# Solvers, Optimizers, and Automatic Differentiation

```{contents} Contents
:depth: 2
```

## Overview

In this lecture we introduce a few of the Julia libraries that we've found particularly useful for quantitative work in economics.



```{code-cell} julia
---
tags: [hide-output]
---
using LinearAlgebra, Statistics
using ForwardDiff, Optim, JuMP, Ipopt, Roots, NLsolve
using Optim: converged, maximum, maximizer, minimizer, iterations #some extra functions
```

## Introduction to Differentiable Programming

The promise of differentiable programming is that we can move towards taking the derivatives of almost arbitrarily
complicated computer programs, rather than simply thinking about the derivatives of mathematical functions.  Differentiable
programming is the natural evolution of automatic differentiation (AD, sometimes called algorithmic differentiation).

Stepping back, there are three ways to calculate the gradient or Jacobian

* Analytic derivatives / Symbolic differentiation
    * You can sometimes calculate the derivative on pen-and-paper, and potentially simplify the expression.
    * In effect, repeated applications of the chain rule, product rule, etc.
    * It is sometimes, though not always, the most accurate and fastest option if there are algebraic simplifications.
    * Sometimes symbolic integration on the computer a good solution, if the package can handle your functions. Doing algebra by hand is tedious and error-prone, but
      is sometimes invaluable.
* Finite differences
    * Evaluate the function at least $N+1$ times to get the gradient -- Jacobians are even worse.
    * Large $\Delta$ is numerically stable but inaccurate, too small of $\Delta$ is numerically unstable but more accurate.
    * Choosing the $\Delta$ is hard, so use packages such as [DiffEqDiffTools.jl](https://github.com/JuliaDiffEq/DiffEqDiffTools.jl).
    * If a function is $R^N \to R$ for a large $N$, this requires $O(N)$ function evaluations.

$$
\partial_{x_i}f(x_1,\ldots x_N) \approx \frac{f(x_1,\ldots x_i + \Delta,\ldots x_N) - f(x_1,\ldots x_i,\ldots x_N)}{\Delta}
$$

* Automatic Differentiation
    * The same as analytic/symbolic differentiation, but where the **chain rule** is calculated **numerically** rather than symbolically.
    * Just as with analytic derivatives, can establish rules for the derivatives of individual functions (e.g. $d\left(sin(x)\right)$ to $cos(x) dx$) for intrinsic derivatives.

AD has two basic approaches, which are variations on the order of evaluating the chain rule: reverse and forward mode (although mixed mode is possible).

1. If a function is $R^N \to R$, then **reverse-mode** AD can find the gradient in $O(1)$ sweep (where a "sweep" is $O(1)$ function evaluations).
1. If a function is $R \to R^N$, then **forward-mode** AD can find the jacobian in $O(1)$ sweeps.

We will explore two types of automatic differentiation in Julia (and discuss a few packages which implement them).  For both, remember the [chain rule](https://en.wikipedia.org/wiki/Chain_rule)

$$
\frac{dy}{dx} = \frac{dy}{dw} \cdot \frac{dw}{dx}
$$

Forward-mode starts the calculation from the left with $\frac{dy}{dw}$ first, which then calculates the product with $\frac{dw}{dx}$.  On the other hand, reverse mode starts on the right hand side with $\frac{dw}{dx}$ and works backwards.

Take an example a function with fundamental operations and known analytical derivatives

$$
f(x_1, x_2) = x_1 x_2 + \sin(x_1)
$$

And rewrite this as a function which contains a sequence of simple operations and temporaries.

```{code-cell} julia
function f(x_1, x_2)
    w_1 = x_1
    w_2 = x_2
    w_3 = w_1 * w_2
    w_4 = sin(w_1)
    w_5 = w_3 + w_4
    return w_5
end
```

Here we can identify all of the underlying functions (`*, sin, +`), and see if each has an
intrinsic derivative.  While these are obvious, with Julia we could come up with all sorts of differentiation rules for arbitrarily
complicated combinations and compositions of intrinsic operations.  In fact, there is even [a package](https://github.com/JuliaDiff/ChainRules.jl) for registering more.

### Forward-Mode Automatic Differentiation

In forward-mode AD, you first fix the variable you are interested in (called "seeding"), and then evaluate the chain rule in left-to-right order.

For example, with our $f(x_1, x_2)$ example above, if we wanted to calculate the derivative with respect to $x_1$ then
we can seed the setup accordingly.  $\frac{\partial  w_1}{\partial  x_1} = 1$ since we are taking the derivative of it, while $\frac{\partial  w_2}{\partial  x_1} = 0$.

Following through with these, redo all of the calculations for the derivative in parallel with the function itself.

$$
\begin{array}{l|l}
f(x_1, x_2) &
\frac{\partial f(x_1,x_2)}{\partial x_1}
\\
\hline
w_1 = x_1 &
\frac{\partial  w_1}{\partial  x_1} = 1 \text{ (seed)}\\
w_2 = x_2 &
\frac{\partial   w_2}{\partial  x_1} = 0 \text{ (seed)}
\\
w_3 = w_1 \cdot w_2 &
\frac{\partial  w_3}{\partial x_1} = w_2 \cdot \frac{\partial   w_1}{\partial  x_1} + w_1 \cdot \frac{\partial   w_2}{\partial  x_1}
\\
w_4 = \sin w_1 &
\frac{\partial   w_4}{\partial x_1} = \cos w_1 \cdot \frac{\partial  w_1}{\partial x_1}
\\
w_5 = w_3 + w_4 &
\frac{\partial  w_5}{\partial x_1} = \frac{\partial  w_3}{\partial x_1} + \frac{\partial  w_4}{\partial x_1}
\end{array}
$$

Since these two could be done at the same time, we say there is "one pass" required for this calculation.

Generalizing a little, if the function was vector-valued, then that single pass would get the entire row of the Jacobian in that single pass.  Hence for a $R^N \to R^M$ function, requires $N$ passes to get a dense Jacobian using forward-mode AD.

How can you implement forward-mode AD?  It turns out to be fairly easy with a generic programming language to make a simple example (while the devil is in the details for
a high-performance implementation).

### Forward-Mode with Dual Numbers

One way to implement forward-mode AD is to use [dual numbers](https://en.wikipedia.org/wiki/Dual_number).

Instead of working with just a real number, e.g. $x$, we will augment each with an infinitesimal $\epsilon$ and use $x + \epsilon$.

From Taylor's theorem,

$$
f(x + \epsilon) = f(x) + f'(x)\epsilon + O(\epsilon^2)
$$

where we will define the infinitesimal such that $\epsilon^2 = 0$.

With this definition, we can write a general rule for differentiation of $g(x,y)$ as the chain rule for the total derivative

$$
g(x + \epsilon, y + \epsilon) = g(x, y) + (\partial_x g(x,y) + \partial_y g(x,y))\epsilon
$$

But, note that if we keep track of the constant in front of the $\epsilon$ terms (e.g. a $x'$ and $y'$)

$$
g(x + x'\epsilon, y + y'\epsilon) = g(x, y) + (\partial_x g(x,y)x' + \partial_y g(x,y)y')\epsilon
$$

This is simply the chain rule.  A few more examples

$$
\begin{aligned}
        (x + x'\epsilon) + (y + y'\epsilon) &= (x + y) + (x' + y')\epsilon\\
(x + x'\epsilon)\times(y + y'\epsilon) &= (xy) + (x'y + y'x)\epsilon\\
\exp(x + x'\epsilon) &= \exp(x) + (x'\exp(x))\epsilon\\
        \end{aligned}
$$

Using the generic programming in Julia, it is easy to define a new dual number type which can encapsulate the pair $(x, x')$ and provide a definitions for
all of the basic operations.  Each definition then has the chain-rule built into it.

With this approach, the "seed" process is simple the creation of the $\epsilon$ for the underlying variable.

So if we have the function $f(x_1, x_2)$ and we wanted to find the derivative $\partial_{x_1} f(3.8, 6.9)$ then then we would seed them with the dual numbers $x_1 \to (3.8, 1)$ and $x_2 \to (6.9, 0)$.

If you then follow all of the same scalar operations above with a seeded dual number, it will calculate both the function value and the derivative in a single "sweep" and without modifying any of your (generic) code.

### ForwardDiff.jl

Dual-numbers are at the heart of one of the AD packages we have already seen.

```{code-cell} julia
using ForwardDiff
h(x) = sin(x[1]) + x[1] * x[2] + sinh(x[1] * x[2]) # multivariate.
x = [1.4 2.2]
@show ForwardDiff.gradient(h, x) # use AD, seeds from x

#Or, can use complicated functions of many variables
f(x) = sum(sin, x) + prod(tan, x) * sum(sqrt, x)
g = (x) -> ForwardDiff.gradient(f, x); # g() is now the gradient
g(rand(5)) # gradient at a random point
# ForwardDiff.hessian(f,x') # or the hessian
```

We can even auto-differentiate complicated functions with embedded iterations.

```{code-cell} julia
function squareroot(x) #pretending we don't know sqrt()
    z = copy(x) # Initial starting point for Newton’s method
    while abs(z * z - x) > 1e-13
        z = z - (z * z - x) / (2z)
    end
    return z
end
squareroot(2.0)
```

```{code-cell} julia
using ForwardDiff
dsqrt(x) = ForwardDiff.derivative(squareroot, x)
dsqrt(2.0)
```

### Reverse-Mode AD

Unlike forward-mode auto-differentiation, reverse-mode is very difficult to implement efficiently, and there are many variations on the best approach.

Many reverse-mode packages are connected to machine-learning packages, since the efficient gradients of $R^N \to R$ loss functions are necessary for the gradient descent optimization algorithms used in machine learning.

At this point, Julia does not have a single consistently usable reverse-mode AD package without rough edges, but a few key ones to consider are:

- [ReverseDiff.jl](https://github.com/JuliaDiff/ReverseDiff.jl), a relatively dependable but limited package.  Not really intended for standard ML-pipline usage
- [Zygote.jl](https://github.com/FluxML/Zygote.jl), which is flexible but buggy and less reliable.  In a slow process of deprecation, but often the primary alternative.
- [Enzyme.jl](https://enzyme.mit.edu/julia/stable/), which is the most promising (and supports both forward and reverse mode).  However, the usage is more tailored for scientific machine learning and scalar functions rather than fast GPU kernels, and it relies on a innovative (but not fully stable) approach to compilation.

## Optimization

There are a large number of packages intended to be used for optimization in Julia.

Part of the reason for the diversity of options is that Julia makes it possible to efficiently implement a large number of variations on optimization routines.

The other reason is that different types of optimization problems require different algorithms.

### Optim.jl

A good pure-Julia solution for the (unconstrained or box-bounded) optimization of
univariate and multivariate function is the [Optim.jl](https://github.com/JuliaNLSolvers/Optim.jl) package.

By default, the algorithms in `Optim.jl` target minimization rather than
maximization, so if a function is called `optimize` it will mean minimization.

#### Univariate Functions on Bounded Intervals

[Univariate optimization](http://julianlsolvers.github.io/Optim.jl/stable/user/minimization/#minimizing-a-univariate-function-on-a-bounded-interval)
defaults to a robust hybrid optimization routine called [Brent's method](https://en.wikipedia.org/wiki/Brent%27s_method).

```{code-cell} julia
using Optim
using Optim: converged, maximum, maximizer, minimizer, iterations #some extra functions

result = optimize(x -> x^2, -2.0, 1.0)
```

Always check if the results converged, and throw errors otherwise

```{code-cell} julia
converged(result) || error("Failed to converge in $(iterations(result)) iterations")
xmin = result.minimizer
result.minimum
```

The first line is a logical OR between `converged(result)` and `error("...")`.

If the convergence check passes, the logical sentence is true, and it will proceed to the next line; if not, it will throw the error.

Or to maximize

```{code-cell} julia
f(x) = -x^2
result = maximize(f, -2.0, 1.0)
converged(result) || error("Failed to converge in $(iterations(result)) iterations")
xmin = maximizer(result)
fmax = maximum(result)
```

**Note:** Notice that we call `optimize` results using `result.minimizer`, and `maximize` results using `maximizer(result)`.

#### Unconstrained Multivariate Optimization

There are a variety of [algorithms and options](http://julianlsolvers.github.io/Optim.jl/stable/user/minimization/#_top) for multivariate optimization.

From the documentation, the simplest version is

```{code-cell} julia
f(x) = (1.0 - x[1])^2 + 100.0 * (x[2] - x[1]^2)^2
x_iv = [0.0, 0.0]
results = optimize(f, x_iv) # i.e. optimize(f, x_iv, NelderMead())
```

The default algorithm in `NelderMead`, which is derivative-free and hence requires many function evaluations.

To change the algorithm type to [L-BFGS](http://julianlsolvers.github.io/Optim.jl/stable/algo/lbfgs/)

```{code-cell} julia
results = optimize(f, x_iv, LBFGS())
println("minimum = $(results.minimum) with argmin = $(results.minimizer) in " *
        "$(results.iterations) iterations")
```

Note that this has fewer iterations.

As no derivative was given, it used [finite differences](https://en.wikipedia.org/wiki/Finite_difference) to approximate the gradient of `f(x)`.

However, since most of the algorithms require derivatives, you will often want to use auto differentiation or pass analytical gradients if possible.

```{code-cell} julia
f(x) = (1.0 - x[1])^2 + 100.0 * (x[2] - x[1]^2)^2
x_iv = [0.0, 0.0]
results = optimize(f, x_iv, LBFGS(), autodiff = :forward) # i.e. use ForwardDiff.jl
println("minimum = $(results.minimum) with argmin = $(results.minimizer) in " *
        "$(results.iterations) iterations")
```

Note that we did not need to use `ForwardDiff.jl` directly, as long as our `f(x)` function was written to be generic (see the {doc}`generic programming lecture <../more_julia/generic_programming>` ).

Alternatively, with an analytical gradient

```{code-cell} julia
f(x) = (1.0 - x[1])^2 + 100.0 * (x[2] - x[1]^2)^2
x_iv = [0.0, 0.0]
function g!(G, x)
    G[1] = -2.0 * (1.0 - x[1]) - 400.0 * (x[2] - x[1]^2) * x[1]
    G[2] = 200.0 * (x[2] - x[1]^2)
end

results = optimize(f, g!, x_iv, LBFGS()) # or ConjugateGradient()
println("minimum = $(results.minimum) with argmin = $(results.minimizer) in " *
        "$(results.iterations) iterations")
```

For derivative-free methods, you can change the algorithm -- and have no need to provide a gradient

```{code-cell} julia
f(x) = (1.0 - x[1])^2 + 100.0 * (x[2] - x[1]^2)^2
x_iv = [0.0, 0.0]
results = optimize(f, x_iv, SimulatedAnnealing()) # or ParticleSwarm() or NelderMead()
```

However, you will note that this did not converge, as stochastic methods typically require many more iterations as a tradeoff for their global-convergence properties.

See the [maximum likelihood](http://julianlsolvers.github.io/Optim.jl/stable/examples/generated/maxlikenlm/)
example and the accompanying [Jupyter notebook](https://nbviewer.jupyter.org/github/JuliaNLSolvers/Optim.jl/blob/gh-pages/v0.15.3/examples/generated/maxlikenlm.ipynb).

### JuMP.jl

The [JuMP.jl](https://github.com/JuliaOpt/JuMP.jl) package is an ambitious implementation of a modelling language for optimization problems in Julia.

In that sense, it is more like an AMPL (or Pyomo) built on top of the Julia
language with macros, and able to use a variety of different commerical and open source solvers.

If you have a linear, quadratic, conic, mixed-integer linear, etc. problem then this will likely be the ideal "meta-package" for calling various solvers.

For nonlinear problems, the modelling language may make things difficult for complicated functions (as it is not designed to be used as a general-purpose nonlinear optimizer).

See the [quick start guide](http://www.juliaopt.org/JuMP.jl/0.18/quickstart.html) for more details on all of the options.

The following is an example of calling a linear objective with a nonlinear constraint (provided by an external function).

Here `Ipopt` stands for `Interior Point OPTimizer`, a [nonlinear solver](https://github.com/JuliaOpt/Ipopt.jl) in Julia

```{code-cell} julia
using JuMP, Ipopt
# solve
# max( x[1] + x[2] )
# st sqrt(x[1]^2 + x[2]^2) <= 1

function squareroot(x) # pretending we don't know sqrt()
    z = x # Initial starting point for Newton’s method
    while abs(z * z - x) > 1e-13
        z = z - (z * z - x) / (2z)
    end
    return z
end
m = Model(Ipopt.Optimizer)
# need to register user defined functions for AD
JuMP.register(m, :squareroot, 1, squareroot, autodiff = true)

@variable(m, x[1:2], start=0.5) # start is the initial condition
@objective(m, Max, sum(x))
@NLconstraint(m, squareroot(x[1]^2 + x[2]^2)<=1)
@show JuMP.optimize!(m)
```

And this is an example of a quadratic objective

```{code-cell} julia
# solve
# min (1-x)^2 + (100(y-x^2)^2)
# st x + y >= 10

using JuMP, Ipopt
m = Model(Ipopt.Optimizer) # settings for the solver
@variable(m, x, start=0.0)
@variable(m, y, start=0.0)

@NLobjective(m, Min, (1 - x)^2+100(y - x^2)^2)

JuMP.optimize!(m)
println("x = ", value(x), " y = ", value(y))

# adding a (linear) constraint
@constraint(m, x + y==10)
JuMP.optimize!(m)
println("x = ", value(x), " y = ", value(y))
```

### BlackBoxOptim.jl

Another package for doing global optimization without derivatives is [BlackBoxOptim.jl](https://github.com/robertfeldt/BlackBoxOptim.jl).


An example for [parallel execution](https://github.com/robertfeldt/BlackBoxOptim.jl/blob/master/examples/rosenbrock_parallel.jl) of the objective is provided.

## Systems of Equations and Least Squares

### Roots.jl

A root of a real function $f$ on $[a,b]$ is an $x \in [a, b]$ such that $f(x)=0$.

For example, if we plot the function

```{math}
:label: root_f

f(x) = \sin(4 (x - 1/4)) + x + x^{20} - 1
```

with $x \in [0,1]$ we get

```{figure} /_static/figures/sine-screenshot-2.png

```

The unique root is approximately 0.408.

The [Roots.jl](https://github.com/JuliaLang/Roots.jl) package offers `fzero()` to find roots

```{code-cell} julia
using Roots
f(x) = sin(4 * (x - 1 / 4)) + x + x^20 - 1
fzero(f, 0, 1)
```

### NLsolve.jl

The [NLsolve.jl](https://github.com/JuliaNLSolvers/NLsolve.jl/) package provides functions to solve for multivariate systems of equations and fixed points.

From the documentation, to solve for a system of equations without providing a Jacobian

```{code-cell} julia
using NLsolve

f(x) = [(x[1] + 3) * (x[2]^3 - 7) + 18
        sin(x[2] * exp(x[1]) - 1)] # returns an array

results = nlsolve(f, [0.1; 1.2])
```

In the above case, the algorithm used finite differences to calculate the Jacobian.

Alternatively, if `f(x)` is written generically, you can use auto-differentiation with a single setting.

```{code-cell} julia
results = nlsolve(f, [0.1; 1.2], autodiff = :forward)

println("converged=$(NLsolve.converged(results)) at root=$(results.zero) in " *
        "$(results.iterations) iterations and $(results.f_calls) function calls")
```

Providing a function which operates inplace (i.e., modifies an argument) may help performance for large systems of equations (and hurt it for small ones).

```{code-cell} julia
function f!(F, x) # modifies the first argument
    F[1] = (x[1] + 3) * (x[2]^3 - 7) + 18
    F[2] = sin(x[2] * exp(x[1]) - 1)
end

results = nlsolve(f!, [0.1; 1.2], autodiff = :forward)

println("converged=$(NLsolve.converged(results)) at root=$(results.zero) in " *
        "$(results.iterations) iterations and $(results.f_calls) function calls")
```

## LeastSquaresOptim.jl

Many optimization problems can be solved using linear or nonlinear least squares.

Let $x \in R^N$ and $F(x) : R^N \to R^M$ with $M \geq N$, then the nonlinear least squares problem is

$$
\min_x F(x)^T F(x)
$$

While $F(x)^T F(x) \to R$, and hence this problem could technically use any nonlinear optimizer, it is useful to exploit the structure of the problem.

In particular, the Jacobian of $F(x)$, can be used to approximate the Hessian of the objective.

As with most nonlinear optimization problems, the benefits will typically become evident only when analytical or automatic differentiation is possible.

If $M = N$ and we know a root $F(x^*) = 0$ to the system of equations exists, then NLS is the defacto method for solving large **systems of equations**.

An implementation of NLS is given in [LeastSquaresOptim.jl](https://github.com/matthieugomez/LeastSquaresOptim.jl).


## Exercises

### Exercise 1

Doing a simple implementation of forward-mode auto-differentiation is very easy in Julia since it is generic.  In this exercise, you
will fill in a few of the operations required for a simple AD implementation.

First, we need to provide a type to hold the dual.

```{code-cell} julia
struct DualNumber{T} <: Real
    val::T
    ϵ::T
end
```

Here we have made it a subtype of `Real` so that it can pass through functions expecting Reals.

We can add on a variety of chain rule definitions by importing in the appropriate functions and adding DualNumber versions.  For example

```{code-cell} julia
import Base: +, *, -, ^, exp
+(x::DualNumber, y::DualNumber) = DualNumber(x.val + y.val, x.ϵ + y.ϵ)  # dual addition
+(x::DualNumber, a::Number) = DualNumber(x.val + a, x.ϵ)  # i.e. scalar addition, not dual
+(a::Number, x::DualNumber) = DualNumber(x.val + a, x.ϵ)  # i.e. scalar addition, not dual
```

With that, we can seed a dual number and find simple derivatives,

```{code-cell} julia
f(x, y) = 3.0 + x + y

x = DualNumber(2.0, 1.0)  # x -> 2.0 + 1.0\epsilon
y = DualNumber(3.0, 0.0)  # i.e. y = 3.0, no derivative

# seeded calculates both teh function and the d/dx gradient!
f(x, y)
```

For this assignment:

1. Add in AD rules for the other operations: `*, -, ^, exp`.
1. Come up with some examples of univariate and multivariate functions combining those operations and use your AD implementation to find the derivatives.

