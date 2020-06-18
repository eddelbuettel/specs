specs: Single-equation Penalized Error-Correction Selector
==========================================================

The R package `specs` implements the single-equation penalized
error-correction selector proposed in Smeekes and Wijler (2018a) as an
automated approach towards sparse single-equation cointegration
modelling. In addition, the package contains the dataset used in the
empirical application in Smeekes and Wijler (2018a) to predict Dutch
unemployment rates based on Google Trends.

Installation and Loading
------------------------

### Installation

The development version of the `specs` package can be installed from
GitHub using

    # install.package("devtools")
    devtools::install_github("wijler/specs")

### Load Package

After installation, the package can be loaded in the standard way:

    library(specs)

Data
----

Loading `specs` provides access to the dataset `Unempl_GT` which
contains the monthly unemployment rates (x1000) in the Netherlands and a
set of 87 related Google Trends aggregated to a monthly frequency. The
data covers the period from 1 January 2014 to December 2017 and is used
in the empirical application in Smeekes and Wijler (2018a).

Sparse single-equation cointegration modelling
----------------------------------------------

The package `specs` enables sparse and automated estimation of
single-equation cointegration modelling. To estimate a conditional
error-correction model (CECM) on the untransformed levels of a
collection of time series, use the function `specs`. Conversion of the
data matrix in levels to a CECM representation is performed
automatically within the function. The user is free to set the number of
lagged differences to include through the input `p`. In addition, the
inclusion of deterministic (unpenalized) terms is governed via the input
`deterministics = c("constant","trend","both","none")`. When it is not
desired to explicitly model cointegration, the lagged levels can be
omitted from the model by setting `ADL = TRUE`. This estimates a
penalized autoregressive distributed lag (ADL) model on the differenced
data.

Alternatively, the user may choose to pre-transform the data into the
form of a CECM/ADL, for example to save on computation time in rolling
window forecast exercises, and use the function `specs_tr` instead. This
function operates entirely analogous to `specs`, with the exception of
requiring a differenced dependent variable (`y_d`), the lagged levels of
the time series (`z_l`) and the required difference of the data (`w`) as
inputs. Since `w` is directly provided to the function, the option to
set the lag length `p` is omitted. When `ADL=TRUE`, the user may omit
`z_l` as an input.

Note that within this package the coefficients corresponding to `z_l`
are referred to as `delta`, those corresponding to `w` as `pi`, and the
numeric object that stacks both `delta` and `pi` is referred to as
`gamma`. The deterministic terms are passed to the function outputs as
`d`, with there coefficients being referred to as `theta`. When
`ADL=TRUE`, `delta` is omitted from the numeric object `gamma` in the
output. The naming of objects here is congruent with Smeekes and Wijler
(2018a), which may serve as a helpful guideline for implementation.

**Penalty**

The functions `specs`, or equivalently `specs_tr`, estimate the model by
variants of penalized regression, customized to the error-correction
framework. In its most general form, specs penalizes each individual
coefficient via `lambda_i`, and adds a group penalty on `delta` via
`lambda_g`. Unless sequences of positive numbers for `lambda_i` and/or
`lambda_g` are supplied, sequences are generated automatically. The
largest value in each respective penalty sequence is chosen as the
smallest value that sets all the coefficients that it penalizes equal to
zero. The smallest penalty in the sequence is chosen as 1e-4 times the
largest value in the penalty sequence. As an important special case, the
user may set `lambda_g = 0`, in which case the function will estimate
the model by (weighted) *L*<sub>1</sub>-penalized regression, i.e. the
(adaptive) lasso. `specs` estimates a unique solution for each
combination of penalty parameters and, among others, provides a matrix
containing the corresponding coefficients called `gamma` as output.

In practice, one typically requires a single choice of penalties that
provides the optimal solution for the model building exercise at hand.
To facilitate selection of such an optimal penalty, the functions
`specs_opt` and `specs_tr_opt`, being the equivalents of `specs` and
`specs_tr`, respectively, come with the added functionality of automated
selection of optimal values for `lambda_i` and `lambda_g`. The selection
criteria can be set via the input `rule`, with the possible choices
being `BIC`, `AIC` or time series cross-validation (`TSCV`). The first
two are information criteria in which the degrees of freedom is
approximated by the number of non-zero coefficients for a particular
solution, whereas the latter is a form of cross-validation that respects
the time series structure of the data. The implementation details for
TSCV can be found in Smeekes and Wijler (2018b, p. 411). Both the
optimal penalty values, as well as the corresponding estimated
coefficients are included in the output.

**Weights**

Finally, specs can be estimated with the use of adaptively weighted
penalization. Automatically generated weighting schemes are available
via the input `weights = c("ridge","ols","none")`. The default option,
`"ridge"` constructs the weights via the use of initial estimates
obtained by ridge regression. In detail, the weights for
*δ*<sub>*i*</sub> and *π*<sub>*j*</sub> are constructed as
|*δ̂*<sub>*i*</sub>|<sup> − *k*<sub>*δ*</sub></sup> and
|*π̂*<sub>*j*</sub>|<sup> − *k*<sub>*π*</sub></sup>, respectively. The
penalty parameter for the ridge regression is automatically chosen by
TSCV. Alternative options are to automatically generate weights via
initial ols estimates (`weights = "ols"`) or to refrain from adaptive
weighting altogether (`weights = "none"`). Alternatively, it is also
possible to supply a sequence of positive weights directly. Finally, the
values for *k*<sub>*δ*</sub> and *k*<sub>*π*</sub> can be chosen by the
user via the equivalently named input options `k_delta` and `k_pi`. The
optimal values for these parameters are case-dependent, although some
theoretical guidance is provided in table 1 of Smeekes and Wijler
(2018a).

**Algorithm and Implementation**

The package `specs` combines accelerated generalized gradient descent
for the estimation of *δ* with coordinatewise descent for the estimation
of *π*. Since *δ* is penalized by both an *L*<sub>1</sub>- and
*L*<sub>2</sub>-penalty, its estimation fits into the framework of the
so-called sparse group lasso, for which numerous computational
procedures have been proposed. This package adopts the algorithm of
Simon et al. (2013), as the use of accelerated gradient descent via
Nesterov updates greatly improves computational time. However, since *π*
is regularized via an *L*<sub>1</sub>-penalty, and is separable from the
penalty on *δ*, the optimal solution for *π* is calculated via the
coordinate-wise descent procedure proposed in Friedman et al. (2009).
Essentially, `specs` iterates between optimizing for *δ* and *π*, where
within each iteration one of the aforementioned two algorithms is
repeated until numerical convergence. All calculation are performed in
C++, with the help of the Rcpp and Armadillo packages.

References
----------

-   Friedman, J., Hastie, T., and Tibshirani, R. (2009). glmnet: Lasso
    and elastic-net regularized generalized linear models. R package
    version, 1(4).
-   Simon, N., Friedman, J., Hastie, T., and Tibshirani, R. (2013). A
    sparse-group lasso. Journal of computational and graphical
    statistics, 22(2), 231-245.
-   Smeekes, S., and Wijler, E. (2018a). An Automated Approach Towards
    Sparse Single-Equation Cointegration Modelling. arXiv preprint
    arXiv:1809.08889.
-   Smeekes, S., & Wijler, E. (2018b). Macroeconomic forecasting using
    penalized regression methods. International journal of forecasting,
    34(3), 408-430.
