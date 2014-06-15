Regression Models
=================
*This note is a reorganization of Dr. Brian Caffo's lecture notes for the Coursera course [Regression Models](https://class.coursera.org/regmods-002).*

# Module III : Generalized Linear Models

## Introduction

The Generalized Linear Model (GLM) was introduced in a 1972 RSSB paper by Nelder and Wedderburn. It involves three components

* An *exponential family* model for the response.
* A systematic component via a linear predictor.
* A link function that connects the means of the response to the linear predictor.

> GLMs can be thought of as a two-stage modeling approach. We first model the response variable using a probability distribution, such as the binomial or Poisson distribution. Second, we model the parameter of the distribution using a collection of prediction and a special form of multi regression. - [OpenIntro](http://www.openintro.org)

Mathematically, first we assume that $Y_i \sim N(\mu_i, \sigma^2)$ (the Gaussian distribution is an exponential family distribution) and define the linear predictor to be $\eta_i = \sum_{k=1}^p X_{ik} \beta_k$. The link function is $g(\mu) = \eta$. 

For linear models $g(\mu) = \mu$ so that $\mu_i = \eta_i$. This yields the same likelihood model as our additive error Gaussian linear model $Y_i = \sum_{k=1}^p X_{ik} \beta_k + \epsilon_{i}$ where $\epsilon_i \stackrel{iid}{\sim} N(0, \sigma^2)$.

### Logistic Regression
logistic regression is a type of GLM where regular multiple regression does not work well. Assume that $Y_i \sim Bernoulli(\mu_i)$ so that $E[Y_i] = \mu_i$ where $0\leq \mu_i \leq 1$.
* Linear predictor $\eta_i = \sum_{k=1}^p X_{ik} \beta_k$
* Link function 
$g(\mu) = \eta = \log\left( \frac{\mu}{1 - \mu}\right)$
$g$ is the (natural) log odds, referred to as the **logit**.
* Note then we can invert the logit function as
$$
\mu_i = \frac{\exp(\eta_i)}{1 + \exp(\eta_i)} ~~~\mbox{and}~~~
1 - \mu_i = \frac{1}{1 + \exp(\eta_i)}
$$
Thus the likelihood is
$$
\prod_{i=1}^n \mu_i^{y_i} (1 - \mu_i)^{1-y_i}
= \exp\left(\sum_{i=1}^n y_i \eta_i \right)
\prod_{i=1}^n (1 + \eta_i)^{-1}
$$

### Poisson Regression
Assume that $Y_i \sim Poisson(\mu_i)$ so that $E[Y_i] = \mu_i$ where $0\leq \mu_i$
* Linear predictor $\eta_i = \sum_{k=1}^p X_{ik} \beta_k$
* Link function 
$g(\mu) = \eta = \log(\mu)$
* Recall that $e^x$ is the inverse of $\log(x)$ so that 
$$
\mu_i = e^{\eta_i}
$$
Thus, the likelihood is
$$
\prod_{i=1}^n (y_i !)^{-1} \mu_i^{y_i}e^{-\mu_i}
\propto \exp\left(\sum_{i=1}^n y_i \eta_i - \sum_{i=1}^n \mu_i\right)
$$

### Summary
In each case, the only way in which the likelihood depends on the data is through 
$$\sum_{i=1}^n y_i \eta_i =
\sum_{i=1}^n y_i\sum_{k=1}^p X_{ik} \beta_k = 
\sum_{k=1}^p \beta_k\sum_{i=1}^n X_{ik} y_i
$$
Thus if we don't need the full data, only $\sum_{i=1}^n X_{ik} y_i$. This simplification is a consequence of chosing so-called **canonical** link functions.

All models acheive their maximum at the root of the so called normal equations
$$
0=\sum_{i=1}^n \frac{(Y_i - \mu_i)}{Var(Y_i)}W_i
$$
where $W_i$ are the derivative of the inverse of the link function.

The variances in the GLMs are

* For the linear model $Var(Y_i) = \sigma^2$ is constant.
* For Bernoulli case $Var(Y_i) = \mu_i (1 - \mu_i)$
* For the Poisson case $Var(Y_i) = \mu_i$. 
* In the latter cases, it is often relevant to have a more flexible variance model, even if it doesn't correspond to an actual likelihood
$$
0=\sum_{i=1}^n \frac{(Y_i - \mu_i)}{\phi \mu_i (1 - \mu_i ) } W_i ~~~\mbox{and}~~~
0=\sum_{i=1}^n \frac{(Y_i - \mu_i)}{\phi \mu_i} W_i
$$
* These are called **quasi-likelihood** normal equations 

### Odds and ends
* The normal equations have to be solved iteratively. Resulting in $\hat \beta_k$ and, if included, $\hat \phi$.
* Predicted linear predictor responses can be obtained as $\hat \eta = \sum_{k=1}^p X_k \hat \beta_k$
* Predicted mean responses as $\hat \mu = g^{-1}(\hat \eta)$
* Coefficients are interpretted as 
$$
g(E[Y | X_k = x_k + 1, X_{\sim k} = x_{\sim k}]) - g(E[Y | X_k = x_k, X_{\sim k}=x_{\sim k}]) = \beta_k
$$
or the change in the link function of the expected response per unit change in $X_k$ holding other regressors constant.
* Variations on Newon/Raphson's algorithm are used to do it.
* Asymptotics are used for inference usually. 
* Many of the ideas from linear models can be brought over to GLMs.


---
Previous Module. [Module II : Multivariable Regression](http://rpubs.com/sialy/regmod-mod-2)
