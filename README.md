<!-- (Thanks to [Katherine Keith](https://github.com/kakeith) for the inspiration!) -->

# `R`

## package management 

My preferred package for package management is `pacman`. Before loading in dependencies, put this at the top of the script 

```
if(!require("pacman")) install.packages("pacman")
```

## dplyr

`slice(1L)` for getting the max value of each group

```
grouped_data <- data %>%
  group_by(variable, group_vars) %>%
  summarize(values = sum(values)) %>%
  mutate(grp = cur_group_id()) %>%
  arrange(-n) %>%
  slice(1L)
```

`recode()` values in variables

`replace_na()` for recoding NA values in variables

Do you want counts of variables in groups without deleting all the other variables? Use `mutate()` after `group_by()` instead of summarize. Then subset accordingly. e.g.:

```
df %>%
  group_by(country_person) %>%
  mutate(
    n_articles_total = n(),
    n_articles_before = sum(before_appoint==1),
    n_articles_after = n_articles_total - n_articles_before,
    n_lang_en = sum(lang_en==1),
    n_lang_other = n_articles_total - n_lang_en,
    av_text_length = mean(length)
  )

df_tidy_subset <- df %>%
  select(
    id, country_person, n_articles_total, n_articles_before,
    n_articles_after, n_lang_en, n_lang_other, av_text_length
  ) %>%
  unique() # rm duplicates
```

__id numbers within groups__

```
df %>% group_by(cat) %>% mutate(id = row_number())
```

## String/character vector manipulation

My general philosophy: avoid pure regex whenever possible 😅 

Remove all characters that are non-numeric: `STRING <- str_remove_all(STRING, "\\D+")`

Extract substring between two strings: `qdapRegex::ex_between()`

## purrr

### map

On reading in multiple files and combining result of a function into a data frame:  https://clauswilke.com/blog/2016/06/13/reading-and-combining-many-tidy-data-files-in-r/

## lubridate (/working with dates in general)

Create date object from year and month columns with `ym()` function (goes for a bunch of different ymd combinations as well). e.g.: 

```
df %>%
  mutate(
    date = ym(paste(Year, Month))
  )
```

There's also a vector of

## ggplot2

If you're a Python user, stick to the Grammar of Graphics and use the [plotnine](https://plotnine.readthedocs.io/en/stable/) library for visualization :D

### Themes

This is the `theme_set()` that I might use for now.

```
# add fonts (this might not be a necessary step)
font_add_google(name = "Fira Sans", family = "fira")
font_add_google(name = "Roboto", family = "roboto")

# themes and text defaults
theme_set(
  theme_minimal() +
    theme(
      legend.position = "bottom",
      plot.title = element_text(family = "fira"),
      text = element_text(family = "roboto")
    )
)
```

### Labels

Use `str_wrap()` around different graphic elements to automatically wrap captions/text/legend labels. Sample code below:

```
top_df %>%
  ggplot(aes(x = date, y = as.numeric(rank), color = str_wrap(game, 20))) +
  geom_point() +
  geom_bump() +
  scale_y_reverse(limits = c(10, 1), n.breaks = 10) +
  labs(
    title = "Top Games Streamed on Twitch",
    subtitle = str_wrap("Games shown are a subset of data with the top 200 ranked games over time. Each of these games have consistently ranked in the top 200, but not necessarily top 10 throughout the years.")
  ) +
  guides(col = guide_legend(ncol = 3))
```

If you want to wrap legend labels but keep factor levels, use the following helper function (thanks Hadley Wickham!)

```
# for wrapping legend labels while keeping original factor levels
# https://github.com/tidyverse/stringr/issues/107
str_wrap_factor <- function(x, ...) {
  levels(x) <- str_wrap(levels(x), ...)
  x
}
```

How to customize which legends are shown based on aesthetic: `guides()`. Example:

```
data %>%
  ggplot(aes(x = type, y = fct_rev(abb), size = n, color = n)) +
  geom_point() +
  labs(
    title = "TITLE",
    x = "",
    y = "",
    color = "",
    caption = "Data Source: DATA_SOURCE\nVisualization: Allison Koh"
    ) +
    guides(size = "none")
```

### Fonts

`{extrafont}` and `{showtext}` are useful for adding different fonts to viz. The former is for loading in existing fonts from system, the latter is for making sure your text shows up in all graphics(and for loading in fonts from google and other places).

__LIFE HACK__ (or more likely, common sense thing that I often forget): Make sure to include font families in `theme_set()` at the top of a script instead of in individual graphics.

Useful lines of code for `{extrafont}` are as follows:

```
# load in system fonts
extrafont::load_fonts()

# show font names
fonts()

# show a data frame of all fonts available
fonttable()
```


Useful lines of code for `{showtext}` are as follows:

```
showtext_auto() # put at the beginning of a script to automatically show text in new graphics devices
```

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

### Alt Text

__Helper Function__

```
# helper function for writing alt text
# https://twitter.com/thomas_mock/status/1375853258145734660
write_alt_text <- function(
  chart_type,
  type_of_data,
  reason,
  misc,
  source
){
  glue::glue(
    "{chart_type} of {type_of_data} where {reason}. \n\n{misc}\n\nData source from {source}"
  )
}
```

__Examples__

The `{TidyTuesdayAltText}` package contains examples of AltText from #TidyTuesday posts between 2019 and 2021.

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

### Test palette with `pie()` function

```
pie(rep(1, 13), col=pal)
```

# Python

## Twarc 

twarc is a command line tool and Python library for collecting and archiving Twitter JSON data via the Twitter API. It has separate commands (twarc and twarc2) for working with the older v1.1 API and the newer v2 API and Academic Access (respectively).

Docs: https://github.com/DocNow/twarc 
More docs: https://scholarslab.github.io/learn-twarc/06-twarc-command-basics

You have to separately install `twarc-csv` to convert jsonl output to csv in the CLI. (What I use: Anaconda/Powershell CLI) 

Workflow for extracting tweets using this Python library, and converting the resulting file to CSV. 

```
cd "C:\datadir\path" 
twarc2 search --archive "search term" tweets.jsonl 
twarc2 csv tweets.jsonl output.csv
```

How to stop tweet collection: **Ctrl + C** 

# LaTeX (incl. knitr for Rmd stuff)

## Beamer

For adjusting vertical alignment of text in the template

```
\documentclass[10pt,professionalfonts,t]{beamer}
```

Get rid of the `t` to revert to vertically centering text.

# Other

## Source Code Images

The site for creating images of source code is https://carbon.now.sh/.

## Palette Sites

- https://coolors.co/
- https://pokepalettes.com/

## knitr

Add figure to slide

```
```{r figure-name, echo = F, out.width = '100%', fig.cap = "Source: SOURCE HERE"}
knitr::include_graphics(here("figures", "Figure.jpeg"))
```
```
