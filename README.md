<!-- (Thanks to [Katherine Keith](https://github.com/kakeith) for the inspiration!) -->

## Tidyverse 

### dplyr 

`slice(1L)` for getting the max value of each group

```
grouped_data <- data %>% 
  group_by(variable, group_vars) %>%
  summarize(values = sum(values)) %>%
  mutate(grp = cur_group_id()) %>% 
  arrange(-n) %>% 
  slice(1L)
```

### ggplot2 

### geom_bernie() 

Instantly improves any plot (or something)

Install

```
remotes::install_github("R-CoderDotCom/ggbernie@main")
```

Geom 

```
geom_bernie(aes(x = 1930, y = 20100), bernie = "sitting")
```

### Alt Text Examples 

Ideal to use alt text when posting dataviz on social media to make your posts more accessible :D This package contains examples of AltText from #TidyTuesday posts between 2019 and 2021. 

A future version of this package will include an annotated dataset of alt text + ratings according to feature: https://twitter.com/spcanelon/status/1405488036989870080. Until it is integrated into the package, the data can be found here: https://github.com/spcanelon/csvConf2021/blob/main/data/annotatedRubric1.csv

```
devtools::install_github("spcanelon/TidyTuesdayAltText
```

### Palette from picture with {paletteR}

https://datascienceplus.com/how-to-use-paletter-to-automagically-build-palettes-from-pictures/

```
devtools::install_github("andreacirilloac/paletter")
create_palette(image_path = "~/Desktop/410px-Piero_della_Francesca_046.jpg",
               number_of_colors =20,
               type_of_variable = “categorical")
```

<!-- ## Spatial Data Stuff  -->

<!--Add useful insights from spatial data viz from June 16, 2021 -->

<!-- - Get country from latitude and longitude: https://cran.r-project.org/web/packages/tidygeocoder/tidygeocoder.pdf -->
<!-- - Reverse geocoding (i.e. lat long to country) sans API: https://cran.r-project.org/web/packages/revgeo/revgeo.pdf -->

## Poststratification  

### {survey}: The original Package...probably? 

This was the easiest to port between Stata and R but it's been a while since I've run this code from the CIS: 

```
svy1 <- survey::svydesign(ids = ~psu+ssu+caseid, data = df)
pop.types <- data.frame(type=df$poststrata, Freq=df$postratasize)
pop.types <- pop.types[!duplicated(pop.types), ]
post <- data.frame(type=df$poststrata)
postsvy<- survey::postStratify(design = svy1, strata = post, population = pop.types)
```
### {srvyr}: Survey but make it tidy 

Replicable code illustrated by a [comparison between {survey} and {srvyr} syntax](https://rdrr.io/cran/srvyr/f/vignettes/extending-srvyr.Rmd)

Setup for replicable code vvv 

```
# S3 generic function
survey_gini <- function(
  x, na.rm = FALSE, vartype = c("se", "ci", "var", "cv"), ...
) {
  if (missing(vartype)) vartype <- "se"
  vartype <- match.arg(vartype, several.ok = TRUE)
  .svy <- srvyr::set_survey_vars(srvyr::cur_svy(), x)

  out <- convey::svygini(~`__SRVYR_TEMP_VAR__`, na.rm = na.rm, design = .svy)
  out <- srvyr::get_var_est(out, vartype)
  out
}
```

Full code comparing syntax between {srvyr} and {survey} 

```
# Example from ?convey::svygini
suppressPackageStartupMessages({
  library(srvyr)
  library(survey)
  library(convey)
  library(laeken)
})
data(eusilc) ; names( eusilc ) <- tolower( names( eusilc ) )

# Setup for survey package
des_eusilc <- svydesign(
  ids = ~rb030, 
  strata = ~db040,  
  weights = ~rb050, 
  data = eusilc
)
des_eusilc <- convey_prep(des_eusilc)

# Setup for srvyr package
srvyr_eusilc <- eusilc %>% 
  as_survey(
    ids = rb030,
    strata = db040,
    weights = rb050
  ) %>%
  convey_prep()

## Ungrouped
# Calculate ungrouped for survey package
svygini(~eqincome, design = des_eusilc)

# Use new function from summarize
srvyr_eusilc %>% 
  summarize(eqincome = survey_gini(eqincome))

## Groups
# Calculate by groups for survey
survey::svyby(~eqincome, ~rb090, des_eusilc, convey::svygini)

# Use new function from summarize
srvyr_eusilc %>% 
  group_by(rb090) %>%
  summarize(eqincome = survey_gini(eqincome))
```

### MRP 

Replication code from Lucas Leeman's MRP example here: https://github.com/lleemann/MrP_chapter/blob/master/ExampleCode.R 

Handout in PDF form from Leeman & Wasserfallen 2018: https://github.com/lleemann/MrP_chapter/blob/master/MrP_Illsutration.pdf

### MRSP 

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

### {autoMRP} 

Developers: Broniecki, Leeman & Wüest 

Link to paper on CRAN: https://cran.r-project.org/web/packages/autoMrP/vignettes/autoMrP_vignette.pdf

Baselines and packages used vvv 

> The results in this paper were obtained using R 4.0.2 with the dplyr 1.0.2, foreach 1.5.0, doParallel 1.0.15, doRNG 1.8.2, magittr 1.5, lme4 1.1-23, glmnet 4.0-2, ranger 0.12.1, kknn 1.3.1,
xgboost 1.2.0.1, caret 6.0-86, SRP 0.1.1, BARP 0.0.1.0001 and autoMrP 0.91 packages. R
itself and all packages except SRP, BARP, and autoMrP used are available from the Comprehensive R Archive Network (CRAN) at https://CRAN.R-project.org/. The SRP package
is available on GitHub at https://github.com/joeornstein/SRP, BARP is available on
GitHub at https://github.com/jbisbee1/BARP, and autoMrP is availalbe on GitHub at
https://github.com/retowuest/autoMrP.
