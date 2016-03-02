---
layout: post
title: A function to build hierarchical regression models
---

Simon A Jackson  
3 March 2016  

Understanding hierarchical regression is essential for any good data modeller, particularly those working in scientific disciplines like psychology (my background). But building them in R can get fiddly, leaving you with the difficult task of tracking multiple objects that might contain errors. In this post, I'll demonstrate how I construct hierarchical regression models using strings and lists to reduce this complexity and uncertainty.

# Data and modelling question

We'll use the `mtcars` data provided with R for this post. This data set provides us with 32 observations scored on eleven continous variables:


```r
str(mtcars)
```

```
## 'data.frame':	32 obs. of  11 variables:
##  $ mpg : num  21 21 22.8 21.4 18.7 18.1 14.3 24.4 22.8 19.2 ...
##  $ cyl : num  6 6 4 6 8 6 8 4 4 6 ...
##  $ disp: num  160 160 108 258 360 ...
##  $ hp  : num  110 110 93 110 175 105 245 62 95 123 ...
##  $ drat: num  3.9 3.9 3.85 3.08 3.15 2.76 3.21 3.69 3.92 3.92 ...
##  $ wt  : num  2.62 2.88 2.32 3.21 3.44 ...
##  $ qsec: num  16.5 17 18.6 19.4 17 ...
##  $ vs  : num  0 0 1 1 0 1 0 1 1 1 ...
##  $ am  : num  1 1 1 0 0 0 0 0 0 0 ...
##  $ gear: num  4 4 4 3 3 3 3 4 4 4 ...
##  $ carb: num  4 4 1 1 2 1 4 2 2 4 ...
```

For this example, we'll predict the `mpg` (Miles per gallon) for cars with their `cyl`	(Number of cylinders), `hp`	(Gross horsepower), and `wt` (Weight [lb/1000]), and their interaction terms. A convention when investigating interaction terms in regression is to determine whether they significantly add to the predictive power of the model over and above the main effect terms. This is achieved by comparing nested/hierarchical regression models.

As we plan on looking at interaction terms, we should start by mean centering our variables. Let's create a copy of the `mtcars` data set with all numeric variables centered just to be safe.


```r
dat <- lapply(mtcars, scale, scale = FALSE)  # Centre but no need to standardise
dat <- as.data.frame(dat)
summary(dat)
```

```
##       mpg               cyl               disp               hp        
##  Min.   :-9.6906   Min.   :-2.1875   Min.   :-159.62   Min.   :-94.69  
##  1st Qu.:-4.6656   1st Qu.:-2.1875   1st Qu.:-109.90   1st Qu.:-50.19  
##  Median :-0.8906   Median :-0.1875   Median : -34.42   Median :-23.69  
##  Mean   : 0.0000   Mean   : 0.0000   Mean   :   0.00   Mean   :  0.00  
##  3rd Qu.: 2.7094   3rd Qu.: 1.8125   3rd Qu.:  95.28   3rd Qu.: 33.31  
##  Max.   :13.8094   Max.   : 1.8125   Max.   : 241.28   Max.   :188.31  
##       drat                wt               qsec               vs         
##  Min.   :-0.83656   Min.   :-1.7043   Min.   :-3.3487   Min.   :-0.4375  
##  1st Qu.:-0.51656   1st Qu.:-0.6360   1st Qu.:-0.9563   1st Qu.:-0.4375  
##  Median : 0.09844   Median : 0.1077   Median :-0.1388   Median :-0.4375  
##  Mean   : 0.00000   Mean   : 0.0000   Mean   : 0.0000   Mean   : 0.0000  
##  3rd Qu.: 0.32344   3rd Qu.: 0.3927   3rd Qu.: 1.0513   3rd Qu.: 0.5625  
##  Max.   : 1.33344   Max.   : 2.2067   Max.   : 5.0512   Max.   : 0.5625  
##        am               gear              carb        
##  Min.   :-0.4062   Min.   :-0.6875   Min.   :-1.8125  
##  1st Qu.:-0.4062   1st Qu.:-0.6875   1st Qu.:-0.8125  
##  Median :-0.4062   Median : 0.3125   Median :-0.8125  
##  Mean   : 0.0000   Mean   : 0.0000   Mean   : 0.0000  
##  3rd Qu.: 0.5938   3rd Qu.: 0.3125   3rd Qu.: 1.1875  
##  Max.   : 0.5938   Max.   : 1.3125   Max.   : 5.1875
```

# The first approach

We want to know if adding the interaction terms to the regression provides a statistically significant improvement in model fit (measured by $R^{2}$). A typical approach is to write out spearate models like this:


```r
# Model 1: main effects only
fit.1 <- lm(mpg ~ cyl + hp + wt, dat)

# Model 2: + 2-way interaction terms
fit.2 <- lm(mpg ~ cyl + hp + wt +
                    cyl:hp + cyl:wt + hp:wt, dat)

# Model 3: + 3-way interaction term
fit.3 <- lm(mpg ~ cyl + hp + wt +
                    cyl:hp + cyl:wt + hp:wt +
                    cyl:hp:wt, dat)
```

We can examine their results by calling summary on each model.


```r
summary(fit.1)
```

```
## 
## Call:
## lm(formula = mpg ~ cyl + hp + wt, data = dat)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.9290 -1.5598 -0.5311  1.1850  5.8986 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  7.456e-16  4.440e-01   0.000 1.000000    
## cyl         -9.416e-01  5.509e-01  -1.709 0.098480 .  
## hp          -1.804e-02  1.188e-02  -1.519 0.140015    
## wt          -3.167e+00  7.406e-01  -4.276 0.000199 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 2.512 on 28 degrees of freedom
## Multiple R-squared:  0.8431,	Adjusted R-squared:  0.8263 
## F-statistic: 50.17 on 3 and 28 DF,  p-value: 2.184e-11
```

```r
summary(fit.2)
```

```
## 
## Call:
## lm(formula = mpg ~ cyl + hp + wt + cyl:hp + cyl:wt + hp:wt, data = dat)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.3526 -1.4828 -0.3622  1.1684  4.0131 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept) -2.050300   0.880758  -2.328   0.0283 *  
## cyl          0.050766   0.591932   0.086   0.9323    
## hp          -0.039760   0.014965  -2.657   0.0135 *  
## wt          -3.550470   0.706439  -5.026 3.49e-05 ***
## cyl:hp       0.012123   0.008767   1.383   0.1789    
## cyl:wt       0.438212   0.780261   0.562   0.5794    
## hp:wt        0.006370   0.023493   0.271   0.7885    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 2.175 on 25 degrees of freedom
## Multiple R-squared:  0.895,	Adjusted R-squared:  0.8697 
## F-statistic:  35.5 on 6 and 25 DF,  p-value: 4.665e-11
```

```r
summary(fit.3)
```

```
## 
## Call:
## lm(formula = mpg ~ cyl + hp + wt + cyl:hp + cyl:wt + hp:wt + 
##     cyl:hp:wt, data = dat)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.3520 -1.4641 -0.1695  1.3445  4.0009 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)  
## (Intercept) -2.333496   1.074865  -2.171   0.0400 *
## cyl          0.346183   0.864684   0.400   0.6924  
## hp          -0.048850   0.024427  -2.000   0.0570 .
## wt          -4.358913   1.845681  -2.362   0.0266 *
## cyl:hp       0.015347   0.011193   1.371   0.1830  
## cyl:wt       0.529070   0.815341   0.649   0.5226  
## hp:wt        0.003792   0.024473   0.155   0.8781  
## cyl:hp:wt    0.006538   0.013751   0.475   0.6388  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 2.21 on 24 degrees of freedom
## Multiple R-squared:  0.8959,	Adjusted R-squared:  0.8656 
## F-statistic: 29.52 on 7 and 24 DF,  p-value: 2.635e-10
```

We then determine whether each more complex model has a significantly different $R^{2}$ to the less complex model using `anova()`.


```r
anova(fit.1, fit.2, fit.3)
```

```
## Analysis of Variance Table
## 
## Model 1: mpg ~ cyl + hp + wt
## Model 2: mpg ~ cyl + hp + wt + cyl:hp + cyl:wt + hp:wt
## Model 3: mpg ~ cyl + hp + wt + cyl:hp + cyl:wt + hp:wt + cyl:hp:wt
##   Res.Df    RSS Df Sum of Sq      F  Pr(>F)  
## 1     28 176.62                              
## 2     25 118.28  3    58.339 3.9829 0.01954 *
## 3     24 117.18  1     1.104 0.2260 0.63878  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

It appears that the two-way interactions add, but the three-way interactions do not. Still, this is not important right now. What is important is the process by which we achieved this result.

So far it seems straight forward. But it doesn't seem to scale very well. What if you have a large data set with tens of predictors, and want four, five, or many more nested models to compare against each other? The first immediate problem is that you have a lot of R code to type out! Every model has to be a perfect copy of the last, plus the new variables. Then you end up with objects for every model, which need to be named, summarised and compared separately. Put simply, things can get messy fast!

# Using strings

The regression function `lm()` accepts strings. Let's check:


```r
model.1 <- "mpg ~ cyl + hp + wt"
fit.1   <- lm(model.1, dat)
summary(fit.1)
```

```
## 
## Call:
## lm(formula = model.1, data = dat)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.9290 -1.5598 -0.5311  1.1850  5.8986 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  7.456e-16  4.440e-01   0.000 1.000000    
## cyl         -9.416e-01  5.509e-01  -1.709 0.098480 .  
## hp          -1.804e-02  1.188e-02  -1.519 0.140015    
## wt          -3.167e+00  7.406e-01  -4.276 0.000199 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 2.512 on 28 degrees of freedom
## Multiple R-squared:  0.8431,	Adjusted R-squared:  0.8263 
## F-statistic: 50.17 on 3 and 28 DF,  p-value: 2.184e-11
```

OK, this is cool, but why bother? The main reason is that we can create our models as string, and iteratively concantenate on them to ensure that we have nested models! Let's try.


```r
model.2 <- paste(model.1, "cyl:hp", "cyl:wt", "hp:wt", sep = " + ")  # Paste on to model 1
model.3 <- paste(model.2, "cyl:hp:wt", sep = " + ")  # Paste on to model 2
print(model.2)
```

```
## [1] "mpg ~ cyl + hp + wt + cyl:hp + cyl:wt + hp:wt"
```

```r
print(model.3)
```

```
## [1] "mpg ~ cyl + hp + wt + cyl:hp + cyl:wt + hp:wt + cyl:hp:wt"
```

```r
fit.2   <- lm(model.2, dat)
fit.3   <- lm(model.3, dat)

summary(fit.2)
```

```
## 
## Call:
## lm(formula = model.2, data = dat)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.3526 -1.4828 -0.3622  1.1684  4.0131 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept) -2.050300   0.880758  -2.328   0.0283 *  
## cyl          0.050766   0.591932   0.086   0.9323    
## hp          -0.039760   0.014965  -2.657   0.0135 *  
## wt          -3.550470   0.706439  -5.026 3.49e-05 ***
## cyl:hp       0.012123   0.008767   1.383   0.1789    
## cyl:wt       0.438212   0.780261   0.562   0.5794    
## hp:wt        0.006370   0.023493   0.271   0.7885    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 2.175 on 25 degrees of freedom
## Multiple R-squared:  0.895,	Adjusted R-squared:  0.8697 
## F-statistic:  35.5 on 6 and 25 DF,  p-value: 4.665e-11
```

```r
summary(fit.3)
```

```
## 
## Call:
## lm(formula = model.3, data = dat)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.3520 -1.4641 -0.1695  1.3445  4.0009 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)  
## (Intercept) -2.333496   1.074865  -2.171   0.0400 *
## cyl          0.346183   0.864684   0.400   0.6924  
## hp          -0.048850   0.024427  -2.000   0.0570 .
## wt          -4.358913   1.845681  -2.362   0.0266 *
## cyl:hp       0.015347   0.011193   1.371   0.1830  
## cyl:wt       0.529070   0.815341   0.649   0.5226  
## hp:wt        0.003792   0.024473   0.155   0.8781  
## cyl:hp:wt    0.006538   0.013751   0.475   0.6388  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 2.21 on 24 degrees of freedom
## Multiple R-squared:  0.8959,	Adjusted R-squared:  0.8656 
## F-statistic: 29.52 on 7 and 24 DF,  p-value: 2.635e-10
```

```r
anova(fit.1, fit.2, fit.3)
```

```
## Analysis of Variance Table
## 
## Model 1: mpg ~ cyl + hp + wt
## Model 2: mpg ~ cyl + hp + wt + cyl:hp + cyl:wt + hp:wt
## Model 3: mpg ~ cyl + hp + wt + cyl:hp + cyl:wt + hp:wt + cyl:hp:wt
##   Res.Df    RSS Df Sum of Sq      F  Pr(>F)  
## 1     28 176.62                              
## 2     25 118.28  3    58.339 3.9829 0.01954 *
## 3     24 117.18  1     1.104 0.2260 0.63878  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

Seems to work and look a bit neater, but there is still room for error. It's a fine point, but it's easy for us to accidentally forget to change the model and fit numbers as we move along. E.g., Model 2 has to paste on Model 1, Model 3 has to paste on Model 2, and so on. If we have lots of models, it's easy for us to get this wrong. Also, we still have lots of objects. In fact, we've got even more objects this time because we have 'model.X' for the each model string, and then 'fit.X' for each model fit.

# Using lists

To manage these issues, I like to construct hierarchical models as strings in a list. The main idea is that we grow a list with the simplest model as the first object, next as the second, and so on. This way, we can always paste on to the last model in the list by referencing its index with `length(our.model.list)`. So we can always reuse the same code, regardless of what model number we're up to. Let's give it a shot.


```r
models <- list()
models[[1]] <- "mpg ~ cyl + hp + wt"  # Start with model 1

# To add a new model, we get the number of models in our list with
# length(models) and add 1. We then paste new variables onto the last model,
# again with length(models).
models[[length(models) + 1]] <- paste(models[[length(models)]], "cyl:hp", "cyl:wt", "hp:wt", sep = " + ")
models[[length(models) + 1]] <- paste(models[[length(models)]], "cyl:hp:wt", sep = " + ")

print(models)
```

```
## [[1]]
## [1] "mpg ~ cyl + hp + wt"
## 
## [[2]]
## [1] "mpg ~ cyl + hp + wt + cyl:hp + cyl:wt + hp:wt"
## 
## [[3]]
## [1] "mpg ~ cyl + hp + wt + cyl:hp + cyl:wt + hp:wt + cyl:hp:wt"
```

We can create a list of corresponding model fits using `lapply()` to iterate through each model and run the regression using our data frame (`dat`) as the data.


```r
fits <- lapply(models, lm, dat)  # Fit all models
```

We can print out a model summary for the *n*th model:

```r
summary(fits[[2]])  # Summary for model 2
```

```
## 
## Call:
## FUN(formula = X[[i]], data = ..1)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.3526 -1.4828 -0.3622  1.1684  4.0131 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept) -2.050300   0.880758  -2.328   0.0283 *  
## cyl          0.050766   0.591932   0.086   0.9323    
## hp          -0.039760   0.014965  -2.657   0.0135 *  
## wt          -3.550470   0.706439  -5.026 3.49e-05 ***
## cyl:hp       0.012123   0.008767   1.383   0.1789    
## cyl:wt       0.438212   0.780261   0.562   0.5794    
## hp:wt        0.006370   0.023493   0.271   0.7885    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 2.175 on 25 degrees of freedom
## Multiple R-squared:  0.895,	Adjusted R-squared:  0.8697 
## F-statistic:  35.5 on 6 and 25 DF,  p-value: 4.665e-11
```

We can also use `lapply()` to print out all of the model summaries at once.


```r
lapply(fits, summary)
```

```
## [[1]]
## 
## Call:
## FUN(formula = X[[i]], data = ..1)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.9290 -1.5598 -0.5311  1.1850  5.8986 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  7.456e-16  4.440e-01   0.000 1.000000    
## cyl         -9.416e-01  5.509e-01  -1.709 0.098480 .  
## hp          -1.804e-02  1.188e-02  -1.519 0.140015    
## wt          -3.167e+00  7.406e-01  -4.276 0.000199 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 2.512 on 28 degrees of freedom
## Multiple R-squared:  0.8431,	Adjusted R-squared:  0.8263 
## F-statistic: 50.17 on 3 and 28 DF,  p-value: 2.184e-11
## 
## 
## [[2]]
## 
## Call:
## FUN(formula = X[[i]], data = ..1)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.3526 -1.4828 -0.3622  1.1684  4.0131 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept) -2.050300   0.880758  -2.328   0.0283 *  
## cyl          0.050766   0.591932   0.086   0.9323    
## hp          -0.039760   0.014965  -2.657   0.0135 *  
## wt          -3.550470   0.706439  -5.026 3.49e-05 ***
## cyl:hp       0.012123   0.008767   1.383   0.1789    
## cyl:wt       0.438212   0.780261   0.562   0.5794    
## hp:wt        0.006370   0.023493   0.271   0.7885    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 2.175 on 25 degrees of freedom
## Multiple R-squared:  0.895,	Adjusted R-squared:  0.8697 
## F-statistic:  35.5 on 6 and 25 DF,  p-value: 4.665e-11
## 
## 
## [[3]]
## 
## Call:
## FUN(formula = X[[i]], data = ..1)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.3520 -1.4641 -0.1695  1.3445  4.0009 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)  
## (Intercept) -2.333496   1.074865  -2.171   0.0400 *
## cyl          0.346183   0.864684   0.400   0.6924  
## hp          -0.048850   0.024427  -2.000   0.0570 .
## wt          -4.358913   1.845681  -2.362   0.0266 *
## cyl:hp       0.015347   0.011193   1.371   0.1830  
## cyl:wt       0.529070   0.815341   0.649   0.5226  
## hp:wt        0.003792   0.024473   0.155   0.8781  
## cyl:hp:wt    0.006538   0.013751   0.475   0.6388  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 2.21 on 24 degrees of freedom
## Multiple R-squared:  0.8959,	Adjusted R-squared:  0.8656 
## F-statistic: 29.52 on 7 and 24 DF,  p-value: 2.635e-10
```

To iteratively compare all our model fits, we can use `do.call()` and `anova()` together.


```r
do.call(anova, fits)
```

```
## Analysis of Variance Table
## 
## Model 1: mpg ~ cyl + hp + wt
## Model 2: mpg ~ cyl + hp + wt + cyl:hp + cyl:wt + hp:wt
## Model 3: mpg ~ cyl + hp + wt + cyl:hp + cyl:wt + hp:wt + cyl:hp:wt
##   Res.Df    RSS Df Sum of Sq      F  Pr(>F)  
## 1     28 176.62                              
## 2     25 118.28  3    58.339 3.9829 0.01954 *
## 3     24 117.18  1     1.104 0.2260 0.63878  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

Great! So we now have two objects: `models` and `fits`. We created `models` as a list and in a way that we couldn't make mistakes.
Once we have `models`, we only need one line of code to fit all of the regression models (`fits <- lapply(models, lm, dat)`). After this, we only need one line of code to see all of their summaries (`lapply(fits, summary)`) or compare all of the models together (`do.call(anova, fits)`).

# As a function

This looks like a improvement over our first approach, but there's one final thing I'd like to do. Right now, building the models seems a little convoluted. It works, but it's a lot of repeated code. So let's write a function to handle the model building. There are a few ways we can do this. For clarity, I'll write a function that builds the whole string list of nested regression models at once. As arguements, we take the dependent/outcome variable (dv), and then vectors of the independent variables (ivs) to be included in each new model. We then paste these together as needed.


```r
#' Create a list of hierarchical regression models
createHierarchical <- function(dv, ivs.1, ...) {
  
  ivs <- list(ivs.1, ...)  # ivs for each model as a list
  nModels <- length(ivs)  # Number of models (foruse later)
  models <- vector("list", nModels)  # empty list of required length

  # Create base model
  models[[1]] <- paste(dv, paste(ivs[[1]], collapse = " + "),
                       sep = " ~ ")
  
  # Create all other models
  if (nModels >= 2) {
    for (i in 2:nModels) {
      models[[i]] <- paste(models[[i - 1]],
                           paste(ivs[[i]], collapse = " + "),
                           sep = " + ")
    }
  }

  return (models)
}
```

Let's give it a whirl.


```r
models <- createHierarchical("mpg",  # DV
                       c("cyl", "hp", "wt"),  # IVs in model 1
                       c("cyl:hp", "cyl:wt", "hp:wt"),  # IVs to add in model 2
                       c("cyl:hp:wt"))  # IVs to add in model 3
print(models)
```

```
## [[1]]
## [1] "mpg ~ cyl + hp + wt"
## 
## [[2]]
## [1] "mpg ~ cyl + hp + wt + cyl:hp + cyl:wt + hp:wt"
## 
## [[3]]
## [1] "mpg ~ cyl + hp + wt + cyl:hp + cyl:wt + hp:wt + cyl:hp:wt"
```

```r
fits <- lapply(models, lm, dat)
do.call(anova, fits)
```

```
## Analysis of Variance Table
## 
## Model 1: mpg ~ cyl + hp + wt
## Model 2: mpg ~ cyl + hp + wt + cyl:hp + cyl:wt + hp:wt
## Model 3: mpg ~ cyl + hp + wt + cyl:hp + cyl:wt + hp:wt + cyl:hp:wt
##   Res.Df    RSS Df Sum of Sq      F  Pr(>F)  
## 1     28 176.62                              
## 2     25 118.28  3    58.339 3.9829 0.01954 *
## 3     24 117.18  1     1.104 0.2260 0.63878  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

Thanks to our utility function `createHierarchical()`, we can build, fit, and analyse our hierarchical regression in a transparent and simple way. If you wanted, you could go as far as to write another function to fit the models for us, thus removing the need for a `models` object. However, I'll leave this up to you if you're interested. I still like having the ability to quickly look at the models if I need to.

Let's try something a little more convulated by adding only one variable at a time to make sure that this function scales up.


```r
models <- createHierarchical("mpg", "cyl", "hp", "wt", "cyl:hp", "cyl:wt", "hp:wt", "cyl:hp:wt")
fits <- lapply(models, lm, dat)
do.call(anova, fits)
```

```
## Analysis of Variance Table
## 
## Model 1: mpg ~ cyl
## Model 2: mpg ~ cyl + hp
## Model 3: mpg ~ cyl + hp + wt
## Model 4: mpg ~ cyl + hp + wt + cyl:hp
## Model 5: mpg ~ cyl + hp + wt + cyl:hp + cyl:wt
## Model 6: mpg ~ cyl + hp + wt + cyl:hp + cyl:wt + hp:wt
## Model 7: mpg ~ cyl + hp + wt + cyl:hp + cyl:wt + hp:wt + cyl:hp:wt
##   Res.Df    RSS Df Sum of Sq       F    Pr(>F)    
## 1     30 308.33                                   
## 2     29 291.98  1    16.360  3.3507   0.07962 .  
## 3     28 176.62  1   115.354 23.6264 5.919e-05 ***
## 4     27 135.73  1    40.895  8.3759   0.00797 ** 
## 5     26 118.63  1    17.096  3.5015   0.07355 .  
## 6     25 118.28  1     0.348  0.0712   0.79182    
## 7     24 117.18  1     1.104  0.2260   0.63878    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

We just constructed, fit, and compared seven nested regression models in three lines of code! As before, you're able to look at the individual model summaries with `summary(fits[[model.number]])` or look at them all at once with `lapply(fits, summary)`.

# Sign-off 

As always, what's presented here is just my approach. It's not the only approach, and it's unlikley to be the best approach. I hope that you're able to glean something useful from it. Please comment, email me at <drsimonjackson@gmail.com>, or tweet @drsimonj to chat!
