
<!-- README.md is generated from README.Rmd. Please edit that file -->

# MaximinInfer

<!-- badges: start -->

[![CRAN
status](https://www.r-pkg.org/badges/version/MaximinInfer)](https://CRAN.R-project.org/package=MaximinInfer)
<!-- badges: end -->

MaximinInfer is a package that implements the sampling and aggregation
method for the covariate shift maximin effect, which was proposed in
&lt;arXiv:2011.07568&gt;. It constructs the confidence interval for any
linear combination of the high-dimensional maximin effect.

## Installation

You can install the released version of MaximinInfer from
[CRAN](https://CRAN.R-project.org) with:

``` r
install.packages("MaximinInfer")
```

And the development version from [GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("zywang0701/MaximinInfer")
```

## Example

This is a basic example which shows you how to solve a common problem:

``` r
library(MaximinInfer)
```

The data is heterogeneous and covariates shift between source and target
data

``` r
set.seed(0)

## number of groups
L=2
## dimension
p=100

## mean vector for source
mean.source = rep(0, p)
## covariance matrix for source
A1gen <- function(rho,p){
  A1=matrix(0,p,p)
  for(i in 1:p){
    for(j in 1:p){
      A1[i,j]<-rho^(abs(i-j))
    }
  }
  return(A1)
}
cov.source = A1gen(0.6, p)

## 1st group's source data
n1 = 100
X1 = MASS::mvrnorm(n1, mu=mean.source, Sigma=cov.source)
# true coef for 1st group
b1 = rep(0, p)
b1[1:5] = seq(1,5)/20
b1[98:100] = c(0.5, -0.5, -0.5)
Y1 = X1%*%b1 + rnorm(n1)

## 2nd group's source data
n2 = 100
X2 = MASS::mvrnorm(n2, mu=mean.source, Sigma=cov.source)
# true coef for 2nd group
b2 = rep(0, p)
b2[6:10] = seq(1,5)/20
b2[98:100] = 0.5*c(0.5, -0.5, -0.5)
Y2 = X2%*%b2 + rnorm(n2)

## Target Data, covariate shift
n.target = 100
mean.target = rep(0, p)
cov.target = cov.source
for(i in 1:p) cov.target[i, i] = 1.5
for(i in 1:5){
  for(j in 1:5){
    if(i!=j) cov.target[i, j] = 0.9
  }
}
for(i in 99:100){
  for(j in 99:100){
    if(i!=j) cov.target[i, j] = 0.9
  }
}
X.target = MASS::mvrnorm(n.target, mu=mean.target, Sigma=cov.target)
```

``` r
set.seed(0)
## loading
loading = rep(0, 100) # dimension p=100
loading[98:100] = 1

## call - use wrapper function
# mmInfer <- MaximinInfer(list(X1, X2), list(Y1, Y2), loading, X.target, covariate.shift = TRUE)

## call - separate steps
mm <- Maximin(list(X1, X2), list(Y1, Y2), loading, X.target, covariate.shift = TRUE)
mmInfer <- infer(mm)
```

Weights for groups

``` r
mmInfer$weight
#> [1] 0.4729686 0.5270314
```

Point estimator for the linear contrast

``` r
mmInfer$point
#> [1] -0.3883289
```

Confidence Interval for point estimator

``` r
mmInfer$CI
#>           lower      upper
#> [1,] -0.9004034 0.09923036
```

The default ridge penalty used is 0, if you want to make sure the
estimator is more stable, we recommend adding a data-dependent penalty.
The function below will help you tell whether zero penalty suffices to
yield a stable estimator, if not, it will return a suggested penalty
level.

``` r
out <- decide_delta(mm)
out$delta
#> [1] 1.2
out$reward.ratio
#> [1] 0.9503952
```

We can measure instability for specific ridge penalty

``` r
out2 <- measure_instability(mm, delta=out$delta)
# if measure < 0.5, it's stable enough; 
out2$measure
#> [1] 0.04840878
```
