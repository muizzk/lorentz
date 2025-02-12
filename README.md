
<!-- README.md is generated from README.Rmd. Please edit that file -->

The lorentz R package: special relativity

# Introduction

\`\`The nonassociativity of Einstein’s velocity addition is not widely
known’’– Ungar 2006.

The `lorentz` package furnishes some R-centric functionality for special
relativity. Lorentz transformations of four-vectors are handled and some
functionality for the stress energy tensor is given.

The package also works with three-velocities in the context of their
being a gyrogroup. Natural R idiom may be used to manipulate vectors of
three-velocities, although one must be careful with brackets.

# Installation

To install the most recent stable version on CRAN, use
`install.packages()` at the R prompt:

    R> install.packages("lorentz")

To install the current development version use `devtools`:

    R> devtools::install_github("RobinHankin/lorentz")

And then to load the package use `library()`:

``` r
library("lorentz")
#> Loading required package: magrittr
```

# Features

Natural R idiom can be used to define three-velocities, four-velocities,
and Lorentz transformations as four-by-four matrices.

``` r
 u <- as.3vel(c(0.6,0,0))  # define a three-velocity, 0.6c to the right
 u
#>        x y z
#> [1,] 0.6 0 0

as.4vel(u)    # convert to a four-velocity:
#>         t    x y z
#> [1,] 1.25 0.75 0 0
gam(u)  # calculate the gamma term
#> [1] 1.25

B <- boost(as.3vel(c(0.6,0,0))) # give the Lorentz transformation
B
#>       t     x y z
#> t  1.25 -0.75 0 0
#> x -0.75  1.25 0 0
#> y  0.00  0.00 1 0
#> z  0.00  0.00 0 1

 B %*% (1:4)  # Lorentz transform of an arbitrary four-vector
#>    [,1]
#> t -0.25
#> x  1.75
#> y  3.00
#> z  4.00
```

The package is fully vectorized and includes functionality to convert
between three-velocities and four-velocities:

``` r
 set.seed(0)
 options(digits=3)
 # generate 5 random three-velocities:
 (u <- r3vel(5))
#>           x       y      z
#> [1,]  0.230  0.0719  0.314
#> [2,] -0.311  0.4189 -0.277
#> [3,] -0.185  0.5099 -0.143
#> [4,] -0.739 -0.4641  0.129
#> [5,] -0.304 -0.2890  0.593
 # calculate the gamma correction term:
 gam(u)
#> [1] 1.09 1.24 1.21 2.13 1.46

 # add a velocity of 0.9c in the x-direction:
 v <- as.3vel(c(0.9,0,0))
 v+u
#>          x      y      z
#> [1,] 0.936  0.026  0.113
#> [2,] 0.818  0.253 -0.168
#> [3,] 0.858  0.267 -0.075
#> [4,] 0.480 -0.605  0.168
#> [5,] 0.820 -0.174  0.356


 # convert u to a four-velocity:
 as.4vel(u)
#>         t      x       y      z
#> [1,] 1.09  0.250  0.0783  0.341
#> [2,] 1.24 -0.385  0.5190 -0.343
#> [3,] 1.21 -0.223  0.6160 -0.173
#> [4,] 2.13 -1.571 -0.9862  0.273
#> [5,] 1.46 -0.443 -0.4209  0.864

 # use four-velocities to effect the same transformation:
 w <- as.4vel(u) %*% boost(-v)
 as.3vel(w)
#>          x      y      z
#> [1,] 0.936  0.026  0.113
#> [2,] 0.818  0.253 -0.168
#> [3,] 0.858  0.267 -0.075
#> [4,] 0.480 -0.605  0.168
#> [5,] 0.820 -0.174  0.356
```

### Creation of three velocities:

Three-velocites behave in interesting and counter-intuitive ways.

``` r
 u <- as.3vel(c(0.2,0.4,0.1))   # single three-velocity
 v <- r3vel(4,0.9)              # 4 random three-velocities with speed 0.9
 w <- as.3vel(c(-0.5,0.1,0.3))  # single three-velocity
```

The three-velocity addition law is given by Ungar.

Then we can see that velocity addition is not commutative:

``` r
 u+v
#>           x      y     z
#> [1,]  0.702 -0.113 0.567
#> [2,] -0.679  0.580 0.102
#> [3,] -0.046  0.879 0.364
#> [4,]  0.312  0.407 0.788
 v+u
#>           x      y     z
#> [1,]  0.624 -0.378 0.543
#> [2,] -0.823  0.358 0.045
#> [3,] -0.234  0.832 0.401
#> [4,]  0.228  0.190 0.892
 (u+v)-(v+u)
#>          x     y       z
#> [1,] 0.243 0.506  0.1190
#> [2,] 0.201 0.490  0.1206
#> [3,] 0.503 0.245 -0.0519
#> [4,] 0.242 0.564 -0.1105
```

Observe that the difference between `u+v` and `v+u` is not “small” in
any sense. Commutativity is replaced with gyrocommutatitivity:

``` r
# Compare two different ways of calculating the same thing:
 (u+v) - gyr(u,v,v+u)  
#>              x         y         z
#> [1,]  3.53e-15 -1.20e-15  2.89e-15
#> [2,]  2.89e-16 -3.18e-15 -1.08e-16
#> [3,] -4.26e-15  1.09e-13  4.67e-14
#> [4,]  1.67e-15  4.76e-16  1.91e-15

# The other way round:
 (v+u) - gyr(v,u,u+v)
#>              x         y         z
#> [1,]  3.21e-15 -6.42e-16  2.89e-15
#> [2,] -1.45e-15  1.73e-15  1.08e-16
#> [3,]  1.47e-14 -4.07e-14 -2.03e-14
#> [4,]  9.05e-15  6.43e-15  3.24e-14
```

(that is, zero to numerical accuracy)

### Nonassociativity

It would be reasonable to expect that `u+(v+w)==(u+v)+w`. However, this
is not the case:

``` r
 ((u+v)+w) - (u+(v+w))
#>             x       y         z
#> [1,]  0.00613  0.0794 -0.001467
#> [2,] -0.11096 -0.1508 -0.031226
#> [3,] -0.10748 -0.1022  0.000795
#> [4,] -0.05772 -0.0631 -0.007364
```

(that is, significant departure from associativity). Associativity is
replaced with gyroassociativity:

``` r
 (u+(v+w)) - ((u+v)+gyr(u,v,w))
#>             x        y         z
#> [1,]  0.0e+00 8.16e-17 -6.53e-16
#> [2,] -3.8e-15 2.85e-15  9.49e-16
#> [3,]  0.0e+00 3.21e-15  1.60e-15
#> [4,]  0.0e+00 0.00e+00  0.00e+00
 ((u+v)+w) - (u+(v+gyr(v,u,w)))
#>              x         y         z
#> [1,]  0.00e+00  4.03e-17 -1.29e-15
#> [2,] -1.81e-15  9.07e-16  0.00e+00
#> [3,]  0.00e+00  1.37e-14  5.48e-15
#> [4,]  0.00e+00 -1.84e-15 -1.84e-15
```

(zero to numerical accuracy).

# References

The most concise reference is

  - A. A. Ungar 2006. *Thomas precession: a kinematic effect of the
    algebra of Einstein’s velocity addition law. Comments on "Deriving
    relativistic momentum and energy: II, Three-dimensional case*.
    European Journal of Physics, 27:L17-L20

# Further information

For more detail, see the package vignette

    vignette("lorentz")
