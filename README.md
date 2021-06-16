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

Packages (from favorite to the other ones): 

- {BARP}: MRP + Bayesian Additive Regression Trees (+ other ML algorithms via `SuperLearner`
- {srvyr}: 
