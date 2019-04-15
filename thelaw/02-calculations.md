Lawler’s Law: Calculations
================

``` r
library(tidyverse)
```

A series of heavy-duty calculations from these data.

### Success rate of Lawler’s Law

The simplest calculations pertaining to Lawler’s Law.

For the whole league, or a particular team, take the winners and
play-by-play summary and answer a few questions for each season:

  - How many games were played?
  - How many games saw Lawler’s Law invoked (at least one team reached
    100)?
  - How many games saw the first team to reach 100 end up winning the
    game?

<!-- end list -->

``` r
firstto100 = function(df) { df %>% filter(POINTS >= 100) %>% slice(1) }

calculate_law = function(y, teamcode = '') {
  games = suppressMessages(read_csv(str_c('processed/pbp-summary-', y, '.csv')))
  winners = suppressMessages(read_csv(str_c('processed/winners-', y, '.csv')))
  
  if (teamcode != '') {
    winners = winners %>% filter(str_detect(MATCHUP, teamcode))
  }
  
  law = inner_join(games, winners, by = 'GAME_ID') %>% 
    arrange(GAME_ID, EVENTNUM) %>% 
    group_by(GAME_ID) %>% 
    do(firstto100(.))
  
  total_games = nrow(winners)
  law_invoked = nrow(law)
  law_correct = nrow(law %>% filter(SIDE == WINNER))
  
  tibble(year = y, total_games = total_games,
         law_invoked = law_invoked, law_correct = law_correct)
}
```

``` r
years = tibble(year = 1997:2019)

thelaw.nba = years %>% 
  rowwise() %>% 
  do(res = calculate_law(.$year)) %>% 
  rowwise() %>% 
  do(bind_rows(.)) %>% 
  mutate(
    pct_games_invoked = law_invoked / total_games,
    pct_law_correct = law_correct / law_invoked
  )

thelaw.lac = years %>% 
  rowwise() %>% 
  do(res = calculate_law(.$year, teamcode = 'LAC')) %>% 
  rowwise() %>% 
  do(bind_rows(.)) %>% 
  mutate(
    pct_games_invoked = law_invoked / total_games,
    pct_law_correct = law_correct / law_invoked
  )
```

``` r
thelaw.nba
```

    ## Source: local data frame [23 x 6]
    ## Groups: <by row>
    ## 
    ## # A tibble: 23 x 6
    ##     year total_games law_invoked law_correct pct_games_invok…
    ##    <int>       <int>       <int>       <int>            <dbl>
    ##  1  1997        1189         677         640            0.569
    ##  2  1998        1189         597         578            0.502
    ##  3  1999         725         266         256            0.367
    ##  4  2000        1189         723         698            0.608
    ##  5  2001        1189         585         561            0.492
    ##  6  2002        1189         602         578            0.506
    ##  7  2003        1189         580         544            0.488
    ##  8  2004        1189         493         470            0.415
    ##  9  2005        1230         706         660            0.574
    ## 10  2006        1230         686         647            0.558
    ## # ... with 13 more rows, and 1 more variable: pct_law_correct <dbl>

``` r
thelaw.lac
```

    ## Source: local data frame [23 x 6]
    ## Groups: <by row>
    ## 
    ## # A tibble: 23 x 6
    ##     year total_games law_invoked law_correct pct_games_invok…
    ##    <int>       <int>       <int>       <int>            <dbl>
    ##  1  1997          82          48          43            0.585
    ##  2  1998          82          51          48            0.622
    ##  3  1999          50          28          28            0.56 
    ##  4  2000          82          53          52            0.646
    ##  5  2001          82          33          30            0.402
    ##  6  2002          82          42          41            0.512
    ##  7  2003          82          43          39            0.524
    ##  8  2004          82          48          47            0.585
    ##  9  2005          82          36          33            0.439
    ## 10  2006          82          43          41            0.524
    ## # ... with 13 more rows, and 1 more variable: pct_law_correct <dbl>

``` r
write_csv(thelaw.nba, 'analysis/law-calcs-nba.csv')
write_csv(thelaw.lac, 'analysis/law-calcs-lac.csv')
```

### Time at which teams hit 100

How much earlier are teams hitting 100 points in a game?

``` r
calculate_timeto100 = function(y) {
  games = suppressMessages(read_csv(str_c('processed/pbp-summary-', y, '.csv')))
  timeto100 = games %>% 
    filter(PERIOD <= 4) %>% 
    group_by(GAME_ID) %>% 
    do(firstto100(.))
  tibble(year = y, avgtimeto100 = mean(timeto100$TIME))
}
```

``` r
avgtimeto100 = years %>% 
  rowwise() %>% 
  do(calculate_timeto100(.$year)) %>% 
  rowwise() %>% 
  do(bind_rows(.))
```

``` r
write_csv(avgtimeto100, 'analysis/time-to-100.csv')
```

### First to X wins

Find the success rate of “First team to X wins” for all point values of
X reached in a given season.

``` r
success_at_x = function(p, tbl) {
  points = tbl %>% filter(POINTS >= p) %>% group_by(GAME_ID) %>% slice(1)
  games = nrow(points)
  games_correct = points %>% mutate(correct = (SIDE == WINNER)) %>% pull(correct) %>% sum()
  pct_correct = games_correct / games
  tibble(point = p, games = games, correct = games_correct, pct_correct = pct_correct)
}

# success_at_x(149)

calculate_firsttox = function(y) {
  games = suppressMessages(read_csv(str_c('processed/pbp-summary-', y, '.csv')))
  winners = suppressMessages(read_csv(str_c('processed/winners-', y, '.csv')))
  together = inner_join(games, winners, by = 'GAME_ID') %>% arrange(GAME_ID, EVENTNUM)
  
  points_range = tibble(point = min(together$POINTS):max(together$POINTS))
  
  points_range %>% 
    rowwise() %>% 
    do(success_at_x(.$point, together)) %>% 
    rowwise() %>% 
    do(bind_rows(.)) %>% 
    mutate(year = y) %>% 
    select(year, point:pct_correct)
}

# calculate_firsttox(2018)

firsttox = years %>% 
  rowwise() %>% 
  do(calculate_firsttox(.$year)) %>% 
  rowwise() %>% 
  do(bind_rows(.))
```

``` r
write_csv(firsttox, 'analysis/first-to-x-points.csv')
```
