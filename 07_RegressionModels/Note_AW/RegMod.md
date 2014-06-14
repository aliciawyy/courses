Regression Models
=================
*This note is a reorganization of Dr. Brian Caffo's lecture notes for the Coursera course [Regression Models](https://class.coursera.org/regmods-002).*

## Introduction

Questions concerning *regression models*:
  * To use the parents' heights to predict children' heights.
  * To try to find a parsimonious, easily described mean relationship between parent and children's heights.
  * To investigate the variation in childrens' heights that appears 
  unrelated to parents' heights (residual variation).
  * To quantify what impact genotype information has beyond parental height in explaining child height.
  * To figure out how/whether and what assumptions are needed to generalize findings beyond the data in question.  
  * Why do children of very tall parents tend to be tall, but a little shorter than their parents and why children of very short parents tend to be short, but a little taller than their parents? (This is a famous question called *Regression to the mean*.)
  
---
## Galton's Data

  * Galton's Data was used by Francis Galton in 1885. 
  * Galton was a statistician who invented the term and concepts of regression and correlation, founded the journal Biometrika, and was the cousin of Charles Darwin.
  * Let's look at the marginal (parents disregarding children and children disregarding parents) distributions first. 
  * Parent distribution is all heterosexual couples.
  * Correction for gender via multiplying female heights by 1.08.
  * Overplotting is an issue from discretization.
  

```r
require(MASS)
```

```
## Loading required package: MASS
```

```r
library(UsingR)
str(galton)
```

```
## 'data.frame':	928 obs. of  2 variables:
##  $ child : num  61.7 61.7 61.7 61.7 61.7 62.2 62.2 62.2 62.2 62.2 ...
##  $ parent: num  70.5 68.5 65.5 64.5 64 67.5 67.5 67.5 66.5 66.5 ...
```

```r
par(mfrow = c(1, 2))
hist(galton$child, col = "blue", breaks = 100, xlab = "Child", ylab = "Count")
hist(galton$parent, col = "red", breaks = 100, xlab = "Parent", ylab = "Count")
```

![plot of chunk galton](figure/galton.png) 


---
### Finding the *middle* via least squares

Consider only the children's heights. 
  * How could one describe the "middle"?
  * One definition, let $Y_i$ be the height of child $i$ for $i = 1, \ldots, n = 928$, then define the middle as the value of $\mu$ that minimizes $$\sum_{i=1}^n (Y_i - \mu)^2$$
  * This is physical center of mass of the histrogram.
  * We can prove that $\mu = \bar Y$.

**Proof.**

$$ 
\begin{align} 
\sum_{i=1}^n (Y_i - \mu)^2 & = \
\sum_{i=1}^n (Y_i - \bar Y + \bar Y - \mu)^2 \\ 
& = \sum_{i=1}^n (Y_i - \bar Y)^2 + \
2 \sum_{i=1}^n (Y_i - \bar Y)  (\bar Y - \mu) +\
\sum_{i=1}^n (\bar Y - \mu)^2 \\
& = \sum_{i=1}^n (Y_i - \bar Y)^2 + \
2 (\bar Y - \mu) \sum_{i=1}^n (Y_i - \bar Y)  +\
\sum_{i=1}^n (\bar Y - \mu)^2 \\
& = \sum_{i=1}^n (Y_i - \bar Y)^2 + \
2 (\bar Y - \mu)  (\sum_{i=1}^n Y_i - n \bar Y) +\
\sum_{i=1}^n (\bar Y - \mu)^2 \\
& = \sum_{i=1}^n (Y_i - \bar Y)^2 + \sum_{i=1}^n (\bar Y - \mu)^2\\ 
& \geq \sum_{i=1}^n (Y_i - \bar Y)^2 \
\end{align} 
$$

So that $\sum_{i=1}^n (Y_i - \mu)^2$ gets the minimum iff $\mu = \bar Y$.

---
### Regression through the origin
First, let's compare the children's heights and their parents' heights.

On the right figure, the size of point represents number of points at that (X, Y) combination.

```r
par(mfrow = c(1, 2))
plot(x = galton$parent, y = galton$child, pch = 19, col = "blue", xlab = "Parent", 
    ylab = "Child")

freqData <- as.data.frame(table(galton$child, galton$parent))
names(freqData) <- c("child", "parent", "freq")
plot(as.numeric(as.vector(freqData$parent)), as.numeric(as.vector(freqData$child)), 
    pch = 21, col = "black", bg = "lightblue", cex = 0.15 * freqData$freq, xlab = "Parent", 
    ylab = "Child")
```

![plot of chunk unnamed-chunk-1](figure/unnamed-chunk-1.png) 


Suppose that $X_i$ are the parents' heights as the **predictor** or explanatory variable and $Y_i$ the response, consider picking the slope $\beta$ that minimizes 
$$\sum_{i=1}^n (Y_i - X_i \beta)^2$$
This is exactly *using the origin as a pivot point* to pick the line that minimizes the sum of the squared vertical distances of the points to the line.

If we subtract the means then the origin is the mean of the parent and children's heights. In such case, the solution is

```r
lm(I(child - mean(child)) ~ I(parent - mean(parent)) - 1, data = galton)
```

```
## 
## Call:
## lm(formula = I(child - mean(child)) ~ I(parent - mean(parent)) - 
##     1, data = galton)
## 
## Coefficients:
## I(parent - mean(parent))  
##                    0.646
```

The best fit line is

```r
freqData <- as.data.frame(table(galton$child, galton$parent))
names(freqData) <- c("child", "parent", "freq")
plot(as.numeric(as.vector(freqData$parent)), as.numeric(as.vector(freqData$child)), 
    pch = 21, col = "black", bg = "lightblue", cex = 0.05 * freqData$freq, xlab = "Parent", 
    ylab = "Child")
lm1 <- lm(galton$child ~ galton$parent)
lines(galton$parent, lm1$fitted, col = "red", lwd = 3)
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 


---
### General least square regression

This time, let's consider finding the best line Child's Height = $\beta_0$ + Parent's Height $\beta_1$ i.e. 
$$Y = \beta_0 + \beta_1 X$$
to minimize the least square residuals
$$
  \dagger := \sum_{i=1}^n [ Y_i - (\beta_0 + \beta_1 X_i)]^2
$$

**Analysis.**

Let $\mu_i := \beta_0 + \beta_1 X_i$ and our estimates be $\hat \mu_i := \hat \beta_0 + \hat \beta_1 X_i$.

Then we want to minimize
$$ \dagger = \sum_{i=1}^n (Y_i - \mu_i)^2 = \sum_{i=1}^n (Y_i - \hat \mu_i) ^ 2 + 2 \sum_{i=1}^n (Y_i - \hat \mu_i) (\hat \mu_i - \mu_i) + \sum_{i=1}^n (\hat \mu_i - \mu_i)^2$$
Suppose that $\sum_{i=1}^n (Y_i - \hat \mu_i) (\hat \mu_i - \mu_i) = 0$ then
$$ \dagger =\sum_{i=1}^n (Y_i - \hat \mu_i) ^ 2  + \sum_{i=1}^n (\hat \mu_i - \mu_i)^2\geq \sum_{i=1}^n (Y_i - \hat \mu_i) ^ 2$$

So that if $\sum_{i=1}^n (Y_i - \hat \mu_i) (\hat \mu_i - \mu_i) = 0$, then the line 
$$Y = \hat \beta_0 + \hat \beta_1 X$$
is the least squares line.

* Now consider forcing $\beta_1 = 0$ and thus $\hat \beta_1=0$ to only consider horizontal lines. Because
$$
\sum_{i=1}^n (Y_i - \hat \mu_i) (\hat \mu_i - \mu_i)  =  \sum_{i=1}^n (Y_i - \hat \beta_0) (\hat \beta_0 - \beta_0)
= (\hat \beta_0 - \beta_0) \sum_{i=1}^n (Y_i   - \hat \beta_0) 
$$
This will equal 0 if $\sum_{i=1}^n (Y_i  - \hat \beta_0) = n\bar Y - n \hat \beta_0=0$

* Thus $\hat \beta_0 = \bar Y.$
* Recall that if we force $\beta_0 = 0$ and thus $\hat \beta_0 = 0$ it is to only consider the regression through the origin. In this case
$$
\sum_{i=1}^n (Y_i - \hat \mu_i) (\hat \mu_i - \mu_i)  =  \sum_{i=1}^n (Y_i - \hat \beta_1 X_i) (\hat \beta_1 - \beta_1)X_i
= (\hat \beta_1 - \beta_1) \sum_{i=1}^n (Y_i X_i   - \hat \beta_1 X_i X_i) 
$$
* Thus $\hat \beta_1 = \frac{\sum_{i=1^n} Y_i X_i}{\sum_{i=1}^n X_i^2}.$







