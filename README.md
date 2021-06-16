# helpful-codestuff

## Tidyverse 

### dplyr 

`slice(1L)` for getting the max value of each group (MIGHT HAVE TO ADD SOMETHING TO ARRANGE) with

```
grouped_data <- data %>% 
  group_by(variable, group_vars) %>%
  
  arrange(-n) %>% 
  slice(1L)
```

## Poststratification with survey data 

### [{BARP}](https://github.com/jbisbee1/BARP/): MRP + Bayesian Additive Regression Trees (+ other ML algorithms via `SuperLearner`)

The main parameters used by the `barp` function include:

1. the survey data `dat`
1. the census data `census`
1. variable names for the outcome `y` and covariates `x`
1. variable name of the geographic unit of interest `geo.unit`

> The user should also specify the name of the column in the census data that lists the proportions or shares that fall into each covariate category (`proportion`). If left to the default "None", `barp` assumes that the census data is raw and calculates the proportions by counting the number of rows for each covariate bin over the total rows per geographic unit.

```
data("gaymar")
census06 <- census06 %>% merge(svy %>% dplyr::select(state,stateid) %>% distinct())
factors <- c("region","gXr","state")
census06[factors] <- lapply(census06[factors],factor)
svy[c(factors)] <- lapply(svy[c(factors)],factor)
barp.obj <- barp(y = "supp_gaymar",
                 x = c("age","educ","gXr",
                       "pvote","religcon",
                       "state","region"),
                 dat = svy,census = census06,
                 geo.unit = "state",
                 proportion = "n",
                 seed = 1021,
                 serialize = TRUE)
```

> The resulting `barp` object summarizes the predicted opinions and bounds as a `data.frame`. Plotting the `barp` object will return either a simple plot of the predicted values and credible intervals (`evaluate_model = FALSE`, the default), or a set of convergence diagnostic plots (`evaluate_model = TRUE`). The latter plot should exhibit relative stability across the post-burn-in Markov Chain Monte Carlo (MCMC) simulations in terms of percent acceptance, number of leafs and terminal nodes, and tree depth (and $\sigma^2$ when $y$ is not a factor).

### {srvyr}: Survey but make it tidy 

### {survey}: The survey stuff 
