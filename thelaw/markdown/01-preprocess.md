Lawler's Law: Data pre-processing
================

``` r
library(tidyverse)
```

Some steps need to be taken before we can ascertain the accuracy of Lawler's Law.

### Who won each game?

This can be inferred from the game log easily.

``` r
start = 1951
end = 2019

for (y in start:end) {
  glog = read_csv(
    str_c('../data/logs/log-', y, '.csv'),
    col_types = cols(GAME_ID = col_character())
  )
  
  final = glog %>% 
    # negative plus-minus means the home team won ...
    mutate(WINNER = case_when(PLUS_MINUS < 0 ~ 'HOME', TRUE ~ 'AWAY')) %>% 
    # ... but only in games where the matchup is listed as "AWAY @ HOME"
    filter(str_detect(MATCHUP, '@')) %>% 
    select(GAME_ID, WINNER, MATCHUP)
  
  write_csv(final, str_c('processed/winners-', y, '.csv'))
}
```

### Distill the play-by-play

The play-by-play is long and includes virtually every event in the game. Scoring is the only thing that matters here.

This is a massive amount of munging that plucks out only scoring plays out and reshapes the data into long format.

``` r
process = function(fpath) {
  df = read_csv(
    fpath,
    col_types = cols(
      GAME_ID = col_character(),
      SCOREMARGIN = col_character(),
      WCTIMESTRING = col_character()
    )
  )
  
  g = df %>% 
    drop_na(SCORE) %>% 
    filter(
      (!is.na(HOMEDESCRIPTION) | !is.na(VISITORDESCRIPTION))
    ) %>% 
    # filter(PERIOD <= 4) %>% 
    arrange(EVENTNUM) %>% 
    separate(PCTIMESTRING, c("minutes", "seconds", "milliseconds"), sep = ':', remove = FALSE) %>% 
    mutate(
      TIME = 2880 - (720)*(as.integer(PERIOD) - 1) - (11 - as.integer(minutes))*(60) - (60 - as.integer(seconds))
    ) %>% 
    separate(SCORE, c('AWAY','HOME'), sep = ' - ', remove = FALSE) %>% 
    replace_na(list(HOMEDESCRIPTION = '', VISITORDESCRIPTION = '')) %>% 
    mutate(
      SCORINGTEAM = case_when(
        HOMEDESCRIPTION != '' ~ 'HOME',
        VISITORDESCRIPTION != '' ~ 'AWAY',
        TRUE ~ 'NULL'
      )
    ) %>% 
    mutate(
      HOMEMARGIN = as.integer(replace(SCOREMARGIN, SCOREMARGIN == 'TIE', 0))
    ) %>% 
    select(GAME_ID,EVENTNUM,PERIOD,TIME,SCORINGTEAM,HOMEMARGIN,SCORE,AWAY,HOME) %>% 
    gather(AWAY, HOME, key = 'SIDE', value = 'POINTS') %>% 
    arrange(EVENTNUM) %>% 
    filter(SCORINGTEAM == SIDE)
  
  g
}

# process('../data/playbyplay/2019/0021800002.csv')
```

Do the munging for each game and spit out one file per season.

``` r
start = 2019 # set to 1997 to run everything all over again
end = 2019

for (y in start:end) {
  fpaths = tibble(path = Sys.glob(str_c('../data/playbyplay/', y, '/*.csv')))
  
  szn = fpaths %>% 
    rowwise() %>% 
    do(res = process(.$path)) %>% 
    rowwise() %>% 
    do(bind_rows(.)) %>% 
    arrange(GAME_ID, EVENTNUM) %>%
    mutate(TIME = as.character(TIME)) # sometimes write_csv is writing TIME out in scientific notation??
  
  write_csv(szn, str_c('processed/pbp-summary-', y, '.csv'))
}
```
