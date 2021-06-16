# helpful-codestuff

## Tidyverse 

### dplyr 

`slice(1L)` for getting the max value of each group (MIGHT HAVE TO ADD SOMETHING TO ARRANGE) with

```
grouped_data <- data %>% 
  count(variable) %>% 
  arrange(-n) %>% 
  slice(1L)
```
