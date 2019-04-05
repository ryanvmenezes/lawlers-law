Lawler's Law calculations
================

``` r
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────────────────────────────────────────────────────────── tidyverse 1.2.1 ──

    ## ✔ ggplot2 3.1.0       ✔ purrr   0.3.0  
    ## ✔ tibble  2.0.1       ✔ dplyr   0.8.0.1
    ## ✔ tidyr   0.8.2       ✔ stringr 1.4.0  
    ## ✔ readr   1.3.1       ✔ forcats 0.3.0

    ## ── Conflicts ────────────────────────────────────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

### Who won each game?

This can be inferred from the game log easily.

``` r
start = 1951
end = 2019

pb = txtProgressBar(min = start, max = end) 

for (i in start:end) {
  glog = read_csv(
    str_c('../data/logs/log-', i, '.csv'),
    col_types = cols(GAME_ID = col_character())
  )
  
  final = glog %>% 
    # negative plus-minus means the home team won ...
    mutate(WINNER = case_when(PLUS_MINUS < 0 ~ 'HOME', TRUE ~ 'AWAY')) %>% 
    # ... but only in games that are listed as where the matchup is "AWAY @ HOME"
    filter(str_detect(MATCHUP, '@')) %>% 
    select(GAME_ID, WINNER, MATCHUP)
  
  write_csv(
    final,
    str_c('processed/winners-', i, '.csv')
  )
  
  setTxtProgressBar(pb, i)
}
```

    ## ===========================================================================

### Distill the play-by-play

The play-by-play is long and includes virtually every event in the game. Scoring is the only thing that matters here.
