# Data collection

This folder contains all NBA game logs (since 1950) and play-by-play data (since 1997).

Thanks to Swar Patel and all those who have contributed to the [nba_api](https://github.com/swar/nba_api) python package. It makes grabbing stats.nba.com data incredibly easy. 

There is one hurdle to the data collection: The API for stats.nba.com is rate-limited, meaning you can only make a certain number of requests in a given time period. (I don't know what the number or the length of time is.) To get around this, I made sure to not to make the same call twice in my python code by checking if the output file existed already. A library like [requests_cache](https://github.com/reclosedev/requests-cache) for python can also help with this process.

Still, it took a couple of days to get all 27,000-plus data files. So I posted them here for others to use.

The code that gathered the data is in two python scripts.

## Game logs

[01-get-game-logs.ipynb](/data/01-get-game-logs.ipynb) writes one game log for each season.

For every game, there is a line for each team. Here's what the top of the 2018-19 log looks like:

| SEASON_ID | TEAM_ID    | TEAM_ABBREVIATION | TEAM_NAME             | GAME_ID  | GAME_DATE  | MATCHUP     | WL | MIN | FGM | FGA | FG_PCT | FG3M | FG3A | FG3_PCT | FTM | FTA | FT_PCT | OREB | DREB | REB | AST | STL | BLK | TOV | PF | PTS | PLUS_MINUS | VIDEO_AVAILABLE | 
|-----------|------------|-------------------|-----------------------|----------|------------|-------------|----|-----|-----|-----|--------|------|------|---------|-----|-----|--------|------|------|-----|-----|-----|-----|-----|----|-----|------------|-----------------| 
| 22018     | 1610612744 | GSW               | Golden State Warriors | 21800002 | 2018-10-16 | GSW vs. OKC | W  | 240 | 42  | 95  | 0.442  | 7    | 26   | 0.269   | 17  | 18  | 0.944  | 17   | 41   | 58  | 28  | 7   | 7   | 21  | 29 | 108 | 8          | 1               | 
| 22018     | 1610612760 | OKC               | Oklahoma City Thunder | 21800002 | 2018-10-16 | OKC @ GSW   | L  | 240 | 33  | 91  | 0.363  | 10   | 37   | 0.27    | 24  | 37  | 0.649  | 16   | 29   | 45  | 21  | 12  | 6   | 15  | 21 | 100 | -8         | 1               | 
| 22018     | 1610612755 | PHI               | Philadelphia 76ers    | 21800001 | 2018-10-16 | PHI @ BOS   | L  | 240 | 34  | 87  | 0.391  | 5    | 26   | 0.192   | 14  | 23  | 0.609  | 6    | 41   | 47  | 18  | 8   | 5   | 16  | 20 | 87  | -18        | 1               | 
| 22018     | 1610612738 | BOS               | Boston Celtics        | 21800001 | 2018-10-16 | BOS vs. PHI | W  | 240 | 42  | 97  | 0.433  | 11   | 37   | 0.297   | 10  | 14  | 0.714  | 12   | 43   | 55  | 21  | 7   | 5   | 15  | 20 | 105 | 18         | 1               | 

## Play-by-play

[02-get-play-by-play.ipynb](/data/02-get-play-by-play.ipynb) grabs the play-by-play data for every game in every season.

The stats.nba.com API is rate-limited, though it remains unclear to me how many requests it takes to hit the limit. This code doesn't hit the API if a file containing that game ID's play-by-play data already exists, to avoid repeated requests.

Here's what the top of play-by-play looks like for the opening game of the 2018-19 season between the Boston Celtics and Philadelphia 76ers:

| GAME_ID  | EVENTNUM | EVENTMSGTYPE | EVENTMSGACTIONTYPE | PERIOD | WCTIMESTRING | PCTIMESTRING | HOMEDESCRIPTION                              | NEUTRALDESCRIPTION | VISITORDESCRIPTION               | SCORE | SCOREMARGIN | 
|----------|----------|--------------|--------------------|--------|--------------|--------------|----------------------------------------------|--------------------|----------------------------------|-------|-------------| 
| 21800001 | 2        | 12           | 0                  | 1      | 8:03 PM      | 12:00        |                                              |                    |                                  |       |             | 
| 21800001 | 4        | 10           | 0                  | 1      | 8:03 PM      | 12:00        | Jump Ball Horford vs. Embiid: Tip to Simmons |                    |                                  |       |             | 
| 21800001 | 7        | 2            | 1                  | 1      | 8:03 PM      | 11:40        |                                              |                    | MISS Covington 27' 3PT Jump Shot |       |             | 
| 21800001 | 8        | 4            | 0                  | 1      | 8:04 PM      | 11:40        | CELTICS Rebound                              |                    |                                  |       |             | 
| 21800001 | 10       | 2            | 1                  | 1      | 8:04 PM      | 11:15        | MISS Tatum 25' 3PT Jump Shot                 |                    |                                  |       |             | 

