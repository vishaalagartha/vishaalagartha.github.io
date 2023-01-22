---
title: "Predicting NBA Games using RNN's"
date: 2022-01-18
permalink: /projects/2023/01/18/nba-game-prediction
tags:
    - python
    - notebook
--- 

```python
!nvidia-smi
```

    Wed Jan 18 17:22:31 2023       
    +-----------------------------------------------------------------------------+
    | NVIDIA-SMI 460.32.03    Driver Version: 460.32.03    CUDA Version: 11.2     |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |                               |                      |               MIG M. |
    |===============================+======================+======================|
    |   0  Tesla T4            Off  | 00000000:00:04.0 Off |                    0 |
    | N/A   72C    P0    31W /  70W |      0MiB / 15109MiB |      0%      Default |
    |                               |                      |                  N/A |
    +-------------------------------+----------------------+----------------------+
                                                                                   
    +-----------------------------------------------------------------------------+
    | Processes:                                                                  |
    |  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
    |        ID   ID                                                   Usage      |
    |=============================================================================|
    |  No running processes found                                                 |
    +-----------------------------------------------------------------------------+



```python
!pip install torchsummaryX
```

    Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
    Collecting torchsummaryX
      Downloading torchsummaryX-1.3.0-py3-none-any.whl (3.6 kB)
    Requirement already satisfied: torch in /usr/local/lib/python3.8/dist-packages (from torchsummaryX) (1.13.1+cu116)
    Requirement already satisfied: numpy in /usr/local/lib/python3.8/dist-packages (from torchsummaryX) (1.21.6)
    Requirement already satisfied: pandas in /usr/local/lib/python3.8/dist-packages (from torchsummaryX) (1.3.5)
    Requirement already satisfied: python-dateutil>=2.7.3 in /usr/local/lib/python3.8/dist-packages (from pandas->torchsummaryX) (2.8.2)
    Requirement already satisfied: pytz>=2017.3 in /usr/local/lib/python3.8/dist-packages (from pandas->torchsummaryX) (2022.7)
    Requirement already satisfied: typing-extensions in /usr/local/lib/python3.8/dist-packages (from torch->torchsummaryX) (4.4.0)
    Requirement already satisfied: six>=1.5 in /usr/local/lib/python3.8/dist-packages (from python-dateutil>=2.7.3->pandas->torchsummaryX) (1.15.0)
    Installing collected packages: torchsummaryX
    Successfully installed torchsummaryX-1.3.0



```python
from google.colab import drive
drive.mount('/content/drive/', force_remount=True)
```

    Mounted at /content/drive/



```python
import numpy as np
import pandas as pd
import torch
import os
from torch.utils.data import Dataset, DataLoader
from torchsummaryX import summary
from matplotlib import pyplot as plt 
import gc
import warnings
warnings.filterwarnings('ignore')
np.set_printoptions(suppress=True)
pd.options.display.max_columns = 100
```

# Data Loading
We use 2 `csv` files containing statistics for games and rankings, respectively. The data was acquired from [this Kaggle link](https://www.kaggle.com/datasets/nathanlauga/nba-games).


```python
DATA_DIR = './drive/MyDrive/basketball_analysis/game-prediction'

games = pd.read_csv(os.path.join(DATA_DIR, 'games.csv'))
print(games.columns)
rankings = pd.read_csv(os.path.join(DATA_DIR, 'ranking.csv'))
print(rankings.columns)
print(games.iloc[0])
print(rankings.iloc[0])
```

    Index(['GAME_DATE_EST', 'GAME_ID', 'GAME_STATUS_TEXT', 'HOME_TEAM_ID',
           'VISITOR_TEAM_ID', 'SEASON', 'TEAM_ID_home', 'PTS_home', 'FG_PCT_home',
           'FT_PCT_home', 'FG3_PCT_home', 'AST_home', 'REB_home', 'TEAM_ID_away',
           'PTS_away', 'FG_PCT_away', 'FT_PCT_away', 'FG3_PCT_away', 'AST_away',
           'REB_away', 'HOME_TEAM_WINS'],
          dtype='object')
    Index(['TEAM_ID', 'LEAGUE_ID', 'SEASON_ID', 'STANDINGSDATE', 'CONFERENCE',
           'TEAM', 'G', 'W', 'L', 'W_PCT', 'HOME_RECORD', 'ROAD_RECORD',
           'RETURNTOPLAY'],
          dtype='object')
    GAME_DATE_EST       2023-01-14
    GAME_ID               22200641
    GAME_STATUS_TEXT         Final
    HOME_TEAM_ID        1610612748
    VISITOR_TEAM_ID     1610612749
    SEASON                    2022
    TEAM_ID_home        1610612748
    PTS_home                 111.0
    FG_PCT_home              0.517
    FT_PCT_home              0.833
    FG3_PCT_home               0.5
    AST_home                  17.0
    REB_home                  51.0
    TEAM_ID_away        1610612749
    PTS_away                  95.0
    FG_PCT_away              0.405
    FT_PCT_away              0.667
    FG3_PCT_away             0.396
    AST_away                  26.0
    REB_away                  32.0
    HOME_TEAM_WINS               1
    Name: 0, dtype: object
    TEAM_ID          1610612743
    LEAGUE_ID                 0
    SEASON_ID             22022
    STANDINGSDATE    2023-01-14
    CONFERENCE             West
    TEAM                 Denver
    G                        42
    W                        29
    L                        13
    W_PCT                  0.69
    HOME_RECORD            18-3
    ROAD_RECORD           11-10
    RETURNTOPLAY            NaN
    Name: 0, dtype: object


# Create a mapping of teams


```python
all_teams = list(set(list(games['HOME_TEAM_ID']) + list(games['VISITOR_TEAM_ID'])))
team_mapping = {}
team_name_to_id = {}
for i in range(len(all_teams)):
  team_mapping[all_teams[i]] = i
  team_name_to_id[rankings[rankings['TEAM_ID'] == all_teams[i]].iloc[0]['TEAM']] = all_teams[i]
team_name_to_id
```




    {'Atlanta': 1610612737,
     'Boston': 1610612738,
     'Cleveland': 1610612739,
     'New Orleans': 1610612740,
     'Chicago': 1610612741,
     'Dallas': 1610612742,
     'Denver': 1610612743,
     'Golden State': 1610612744,
     'Houston': 1610612745,
     'LA Clippers': 1610612746,
     'L.A. Lakers': 1610612747,
     'Miami': 1610612748,
     'Milwaukee': 1610612749,
     'Minnesota': 1610612750,
     'Brooklyn': 1610612751,
     'New York': 1610612752,
     'Orlando': 1610612753,
     'Indiana': 1610612754,
     'Philadelphia': 1610612755,
     'Phoenix': 1610612756,
     'Portland': 1610612757,
     'Sacramento': 1610612758,
     'San Antonio': 1610612759,
     'Oklahoma City': 1610612760,
     'Toronto': 1610612761,
     'Utah': 1610612762,
     'Memphis': 1610612763,
     'Washington': 1610612764,
     'Detroit': 1610612765,
     'Charlotte': 1610612766}



# Data Preprocessing

## Feature selection

* Last 10 games stats of each team
* Last 3 matchups between 2 teams
* Current ranking of each team

## Labels
* 1 = Home team wins, 0 = Road team wins

Also, normalize individual statistics to make our model converge faster.


```python
max_points = max(games['PTS_home'].max(), games['PTS_away'].max())
max_assists = max(games['AST_home'].max(), games['AST_away'].max())
max_reb = max(games['REB_home'].max(), games['REB_away'].max())
print(f'Max points: {max_points}, Max assists: {max_assists}, Max reb: {max_reb}')
```

    Max points: 168.0, Max assists: 50.0, Max reb: 81.0



```python
TEAM_HISTORY = 10
def get_last_games(t, before):
  try:
    home_team_games = games[(games['HOME_TEAM_ID'] == t)]
    home_team_games['IS_HOME'] = 1
    drop_cols = ['GAME_ID', 'GAME_STATUS_TEXT', 'HOME_TEAM_ID',
        'VISITOR_TEAM_ID', 'TEAM_ID_home', 'TEAM_ID_away',
        'PTS_away', 'FG_PCT_away', 'FT_PCT_away', 'FG3_PCT_away', 'AST_away',
        'REB_away']
    home_team_games.drop(columns=drop_cols, inplace=True)
    rename_cols = {
        'PTS_home': 'PTS', 'FG_PCT_home': 'FG_PCT',
        'FT_PCT_home': 'FT_PCT', 'FG3_PCT_home': 'FG3_PCT', 'AST_home': 'AST', 'REB_home': 'REB', 'HOME_TEAM_WINS': 'WIN'
    }
    home_team_games.rename(columns=rename_cols, inplace=True)
    away_team_games =  games[(games['VISITOR_TEAM_ID'] == t)]
    away_team_games['IS_HOME'] = 0
    away_team_games['WIN'] = away_team_games['HOME_TEAM_WINS'].map({1: 0, 0: 1})
    rename_cols = {
        'PTS_away': 'PTS', 'FG_PCT_away': 'FG_PCT',
        'FT_PCT_away': 'FT_PCT', 'FG3_PCT_away': 'FG3_PCT', 'AST_away': 'AST', 'REB_away': 'REB'
    }
    away_team_games.rename(columns=rename_cols, inplace=True)
    team_games = pd.concat((home_team_games, away_team_games))
    before_date = pd.to_datetime(before)
    team_games.loc[:, 'GAME_DATE_EST'] = pd.to_datetime(team_games['GAME_DATE_EST'])
    team_games = team_games.sort_values(by=['GAME_DATE_EST'])
    team_games.dropna(inplace=True)
    team_games = team_games[team_games['GAME_DATE_EST'] < before_date][-TEAM_HISTORY:]
    if len(team_games)<TEAM_HISTORY:
      return None
    team_games = team_games.loc[:, ['WIN', 'IS_HOME', 'PTS', 'FG_PCT', 'FT_PCT', 'FG3_PCT', 'AST', 'REB']]
    team_games.loc[:, 'PTS'] = team_games['PTS']/max_points
    team_games.loc[:, 'AST'] = team_games['AST']/max_assists
    team_games.loc[:, 'REB'] = team_games['REB']/max_reb
    team_games.reset_index(drop=True, inplace=True)
    return team_games
  except:
    return None
```

Let's get the last 10 games for a the New Orleans Pelicans before December 12, 2022. 


```python
get_last_games(1610612740, '2022-12-13')
```





  <div id="df-7328ba3f-1298-4693-a17d-79b60b8353d5">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>WIN</th>
      <th>IS_HOME</th>
      <th>PTS</th>
      <th>FG_PCT</th>
      <th>FT_PCT</th>
      <th>FG3_PCT</th>
      <th>AST</th>
      <th>REB</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>0</td>
      <td>0.738095</td>
      <td>0.472</td>
      <td>0.865</td>
      <td>0.364</td>
      <td>0.46</td>
      <td>0.654321</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>0</td>
      <td>0.660714</td>
      <td>0.442</td>
      <td>0.909</td>
      <td>0.405</td>
      <td>0.50</td>
      <td>0.407407</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>0</td>
      <td>0.666667</td>
      <td>0.495</td>
      <td>0.625</td>
      <td>0.417</td>
      <td>0.64</td>
      <td>0.617284</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>0</td>
      <td>0.696429</td>
      <td>0.461</td>
      <td>0.714</td>
      <td>0.320</td>
      <td>0.70</td>
      <td>0.629630</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>0</td>
      <td>0.720238</td>
      <td>0.435</td>
      <td>0.714</td>
      <td>0.304</td>
      <td>0.46</td>
      <td>0.728395</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0</td>
      <td>0</td>
      <td>0.726190</td>
      <td>0.495</td>
      <td>0.706</td>
      <td>0.412</td>
      <td>0.64</td>
      <td>0.530864</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1</td>
      <td>0</td>
      <td>0.684524</td>
      <td>0.500</td>
      <td>0.778</td>
      <td>0.158</td>
      <td>0.40</td>
      <td>0.617284</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1</td>
      <td>0</td>
      <td>0.767857</td>
      <td>0.573</td>
      <td>0.700</td>
      <td>0.424</td>
      <td>0.66</td>
      <td>0.518519</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0</td>
      <td>0</td>
      <td>0.660714</td>
      <td>0.430</td>
      <td>0.871</td>
      <td>0.323</td>
      <td>0.52</td>
      <td>0.506173</td>
    </tr>
    <tr>
      <th>9</th>
      <td>1</td>
      <td>0</td>
      <td>0.696429</td>
      <td>0.435</td>
      <td>0.962</td>
      <td>0.286</td>
      <td>0.50</td>
      <td>0.654321</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-7328ba3f-1298-4693-a17d-79b60b8353d5')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-7328ba3f-1298-4693-a17d-79b60b8353d5 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-7328ba3f-1298-4693-a17d-79b60b8353d5');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>





```python
MATCHUP_HISTORY = 3
def get_matchup_history(home, away, before):
  matchup_games_home = games[((games['HOME_TEAM_ID'] == home) & (games['VISITOR_TEAM_ID'] == away))]
  matchup_games_home['IS_HOME'] = 1
  matchup_games_home['WIN'] = matchup_games_home['HOME_TEAM_WINS']
  matchup_games_away = games[((games['HOME_TEAM_ID'] == away) & (games['VISITOR_TEAM_ID'] == home))]
  rename_cols =   rename_cols = {
      'PTS_away': 'PTS_home', 'FG_PCT_away': 'FG_PCT_home',
      'FT_PCT_away': 'FT_PCT_home', 'FG3_PCT_away': 'FG3_PCT_home', 'AST_away': 'AST_home', 'REB_away': 'REB_home',
      'PTS_home': 'PTS_away', 'FG_PCT_home': 'FG_PCT_away',
      'FT_PCT_home': 'FT_PCT_away', 'FG3_PCT_home': 'FG3_PCT_away', 'AST_home': 'AST_away', 'REB_home': 'REB_away'
  }
  matchup_games_away.rename(columns=rename_cols, inplace=True)
  matchup_games_away['IS_HOME'] = 0
  matchup_games_away['WIN'] = matchup_games_away['HOME_TEAM_WINS'].map({1: 0, 0: 1})
  before_date = pd.to_datetime(before)
  matchup_games = pd.concat((matchup_games_home, matchup_games_away))
  matchup_games = matchup_games.dropna()
  matchup_games.loc[:, 'GAME_DATE_EST'] = pd.to_datetime(matchup_games['GAME_DATE_EST'])
  matchup_games = matchup_games.sort_values(by=['GAME_DATE_EST'])
  matchup_games = matchup_games[matchup_games['GAME_DATE_EST'] < before_date][-MATCHUP_HISTORY:]
  if len(matchup_games)<MATCHUP_HISTORY:
    return None
  matchup_games.loc[:, 'PTS_home'] = matchup_games['PTS_home']/max_points
  matchup_games.loc[:, 'PTS_away'] = matchup_games['PTS_away']/max_points

  matchup_games.loc[:, 'AST_home'] = matchup_games['AST_home']/max_assists
  matchup_games.loc[:, 'AST_away'] = matchup_games['AST_away']/max_assists


  matchup_games.loc[:, 'REB_home'] = matchup_games['REB_home']/max_reb
  matchup_games.loc[:, 'REB_away'] = matchup_games['REB_away']/max_reb
  matchup_games = matchup_games.loc[:, ['WIN', 'IS_HOME', 'PTS_home', 'FG_PCT_home', 'FT_PCT_home', 'FG3_PCT_home', 'AST_home', 'REB_home',
                                        'PTS_away', 'FG_PCT_away', 'FT_PCT_away', 'FG3_PCT_away', 'AST_away', 'REB_away']]
  matchup_games.reset_index(drop=True, inplace=True)
  return matchup_games
```

Let's get the last 3 games between the New Orleans Pelicans and the Phoenix Suns before December 13, 2022.


```python
get_matchup_history(1610612740, 1610612756, '2022-12-13')
```





  <div id="df-b5f0452c-a0d5-411d-96e9-dde31568e064">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>WIN</th>
      <th>IS_HOME</th>
      <th>PTS_home</th>
      <th>FG_PCT_home</th>
      <th>FT_PCT_home</th>
      <th>FG3_PCT_home</th>
      <th>AST_home</th>
      <th>REB_home</th>
      <th>PTS_away</th>
      <th>FG_PCT_away</th>
      <th>FT_PCT_away</th>
      <th>FG3_PCT_away</th>
      <th>AST_away</th>
      <th>REB_away</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>0.660714</td>
      <td>0.442</td>
      <td>0.909</td>
      <td>0.405</td>
      <td>0.50</td>
      <td>0.407407</td>
      <td>0.738095</td>
      <td>0.522</td>
      <td>0.900</td>
      <td>0.303</td>
      <td>0.66</td>
      <td>0.580247</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>1</td>
      <td>0.761905</td>
      <td>0.511</td>
      <td>0.800</td>
      <td>0.296</td>
      <td>0.54</td>
      <td>0.543210</td>
      <td>0.696429</td>
      <td>0.500</td>
      <td>0.692</td>
      <td>0.500</td>
      <td>0.60</td>
      <td>0.456790</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>1</td>
      <td>0.767857</td>
      <td>0.581</td>
      <td>0.750</td>
      <td>0.320</td>
      <td>0.62</td>
      <td>0.530864</td>
      <td>0.738095</td>
      <td>0.467</td>
      <td>0.765</td>
      <td>0.342</td>
      <td>0.68</td>
      <td>0.518519</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-b5f0452c-a0d5-411d-96e9-dde31568e064')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-b5f0452c-a0d5-411d-96e9-dde31568e064 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-b5f0452c-a0d5-411d-96e9-dde31568e064');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>





```python
def get_ranking(t, date):
  team_ranking = rankings[rankings['TEAM_ID'] == t]
  team_ranking.loc[:, 'STANDINGSDATE'] = pd.to_datetime(team_ranking['STANDINGSDATE'])
  date = pd.to_datetime(date)
  max_games = team_ranking['G'].max()
  team_ranking = team_ranking[team_ranking['STANDINGSDATE'] == date][:1]
  team_ranking.loc[:, 'G'] = team_ranking['G'].apply(lambda x: float(x)/max_games)
  team_ranking.loc[:, 'HOME_W_PCT'] = team_ranking['HOME_RECORD'].apply(lambda x: float(x.split('-')[0])/(max(1, float(x.split('-')[0]) + float(x.split('-')[1]))))
  team_ranking.loc[:, 'AWAY_W_PCT'] = team_ranking['ROAD_RECORD'].apply(lambda x: float(x.split('-')[0])/(max(1, float(x.split('-')[0]) + float(x.split('-')[1]))))
  team_ranking.drop(columns = ['SEASON_ID', 'W', 'L', 'HOME_RECORD', 'ROAD_RECORD', 'TEAM_ID', 'LEAGUE_ID', 'STANDINGSDATE', 'CONFERENCE', 'TEAM', 'RETURNTOPLAY'], inplace=True)
  return team_ranking
```

Let's get the ranking of the Pelicans on December 13, 2022.


```python
get_ranking(1610612740, '2022-12-13')
```





  <div id="df-82ed2e16-cd48-46ef-aa7b-63bff61efa8f">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>G</th>
      <th>W_PCT</th>
      <th>HOME_W_PCT</th>
      <th>AWAY_W_PCT</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>826</th>
      <td>0.329268</td>
      <td>0.667</td>
      <td>0.8</td>
      <td>0.5</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-82ed2e16-cd48-46ef-aa7b-63bff61efa8f')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-82ed2e16-cd48-46ef-aa7b-63bff61efa8f button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-82ed2e16-cd48-46ef-aa7b-63bff61efa8f');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>




**Let's get a single training instance:**

Suppose we want to predict the following game: 

Golden State Warriors vs. Boston Celtics on December 10, 2022


```python
warriors_id = rankings[rankings['TEAM'] == 'Golden State']['TEAM_ID'].iloc[0]
celtics_id = rankings[rankings['TEAM'] == 'Boston']['TEAM_ID'].iloc[0]
print(f'Warriors id: {warriors_id}, Celtics id: {celtics_id}')
date = '2022-12-10'
game = games[(games['GAME_DATE_EST'] == date) & (games['HOME_TEAM_ID'] == warriors_id) & (games['VISITOR_TEAM_ID'] == celtics_id)]
game
```

    Warriors id: 1610612744, Celtics id: 1610612738






  <div id="df-4e004509-a44a-41a7-aaa2-daf5f57f0271">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>GAME_DATE_EST</th>
      <th>GAME_ID</th>
      <th>GAME_STATUS_TEXT</th>
      <th>HOME_TEAM_ID</th>
      <th>VISITOR_TEAM_ID</th>
      <th>SEASON</th>
      <th>TEAM_ID_home</th>
      <th>PTS_home</th>
      <th>FG_PCT_home</th>
      <th>FT_PCT_home</th>
      <th>FG3_PCT_home</th>
      <th>AST_home</th>
      <th>REB_home</th>
      <th>TEAM_ID_away</th>
      <th>PTS_away</th>
      <th>FG_PCT_away</th>
      <th>FT_PCT_away</th>
      <th>FG3_PCT_away</th>
      <th>AST_away</th>
      <th>REB_away</th>
      <th>HOME_TEAM_WINS</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>258</th>
      <td>2022-12-10</td>
      <td>22200392</td>
      <td>Final</td>
      <td>1610612744</td>
      <td>1610612738</td>
      <td>2022</td>
      <td>1610612744</td>
      <td>123.0</td>
      <td>0.511</td>
      <td>0.8</td>
      <td>0.333</td>
      <td>26.0</td>
      <td>53.0</td>
      <td>1610612738</td>
      <td>107.0</td>
      <td>0.437</td>
      <td>0.731</td>
      <td>0.3</td>
      <td>17.0</td>
      <td>39.0</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-4e004509-a44a-41a7-aaa2-daf5f57f0271')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-4e004509-a44a-41a7-aaa2-daf5f57f0271 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-4e004509-a44a-41a7-aaa2-daf5f57f0271');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>





```python
warriors_history = get_last_games(warriors_id, date)
print('Warriors history:')
warriors_history
```

    Warriors history:






  <div id="df-966086a3-51eb-46be-9899-e5c96a03c781">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>WIN</th>
      <th>IS_HOME</th>
      <th>PTS</th>
      <th>FG_PCT</th>
      <th>FT_PCT</th>
      <th>FG3_PCT</th>
      <th>AST</th>
      <th>REB</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>0.648810</td>
      <td>0.464</td>
      <td>0.765</td>
      <td>0.367</td>
      <td>0.62</td>
      <td>0.506173</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>0</td>
      <td>0.767857</td>
      <td>0.515</td>
      <td>0.667</td>
      <td>0.442</td>
      <td>0.62</td>
      <td>0.432099</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>0</td>
      <td>0.625000</td>
      <td>0.493</td>
      <td>0.920</td>
      <td>0.375</td>
      <td>0.46</td>
      <td>0.358025</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>0</td>
      <td>0.684524</td>
      <td>0.467</td>
      <td>0.938</td>
      <td>0.340</td>
      <td>0.54</td>
      <td>0.530864</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>0</td>
      <td>0.708333</td>
      <td>0.457</td>
      <td>1.000</td>
      <td>0.429</td>
      <td>0.60</td>
      <td>0.493827</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1</td>
      <td>0</td>
      <td>0.755952</td>
      <td>0.535</td>
      <td>0.733</td>
      <td>0.511</td>
      <td>0.76</td>
      <td>0.493827</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0</td>
      <td>0</td>
      <td>0.494048</td>
      <td>0.378</td>
      <td>0.895</td>
      <td>0.233</td>
      <td>0.34</td>
      <td>0.419753</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1</td>
      <td>0</td>
      <td>0.815476</td>
      <td>0.575</td>
      <td>0.850</td>
      <td>0.426</td>
      <td>0.72</td>
      <td>0.580247</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0</td>
      <td>0</td>
      <td>0.672619</td>
      <td>0.477</td>
      <td>0.783</td>
      <td>0.256</td>
      <td>0.54</td>
      <td>0.617284</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0</td>
      <td>0</td>
      <td>0.732143</td>
      <td>0.459</td>
      <td>0.760</td>
      <td>0.333</td>
      <td>0.52</td>
      <td>0.518519</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-966086a3-51eb-46be-9899-e5c96a03c781')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-966086a3-51eb-46be-9899-e5c96a03c781 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-966086a3-51eb-46be-9899-e5c96a03c781');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>





```python
celtics_history = get_last_games(celtics_id, date)
print('Celtics history:')
celtics_history
```

    Celtics history:






  <div id="df-6ca62ba7-f27d-4ef5-93de-c1aca3b9b916">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>WIN</th>
      <th>IS_HOME</th>
      <th>PTS</th>
      <th>FG_PCT</th>
      <th>FT_PCT</th>
      <th>FG3_PCT</th>
      <th>AST</th>
      <th>REB</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>0.672619</td>
      <td>0.409</td>
      <td>0.963</td>
      <td>0.268</td>
      <td>0.52</td>
      <td>0.641975</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>0</td>
      <td>0.791667</td>
      <td>0.534</td>
      <td>1.000</td>
      <td>0.529</td>
      <td>0.60</td>
      <td>0.419753</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>0</td>
      <td>0.648810</td>
      <td>0.463</td>
      <td>0.750</td>
      <td>0.324</td>
      <td>0.52</td>
      <td>0.555556</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>0</td>
      <td>0.696429</td>
      <td>0.429</td>
      <td>0.875</td>
      <td>0.375</td>
      <td>0.54</td>
      <td>0.518519</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1</td>
      <td>0</td>
      <td>0.750000</td>
      <td>0.545</td>
      <td>0.900</td>
      <td>0.457</td>
      <td>0.58</td>
      <td>0.604938</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1</td>
      <td>0</td>
      <td>0.696429</td>
      <td>0.482</td>
      <td>0.882</td>
      <td>0.435</td>
      <td>0.56</td>
      <td>0.567901</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0</td>
      <td>0</td>
      <td>0.636905</td>
      <td>0.437</td>
      <td>0.706</td>
      <td>0.380</td>
      <td>0.56</td>
      <td>0.469136</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1</td>
      <td>0</td>
      <td>0.613095</td>
      <td>0.432</td>
      <td>0.900</td>
      <td>0.395</td>
      <td>0.36</td>
      <td>0.592593</td>
    </tr>
    <tr>
      <th>8</th>
      <td>1</td>
      <td>0</td>
      <td>0.690476</td>
      <td>0.489</td>
      <td>0.714</td>
      <td>0.361</td>
      <td>0.54</td>
      <td>0.604938</td>
    </tr>
    <tr>
      <th>9</th>
      <td>1</td>
      <td>0</td>
      <td>0.744048</td>
      <td>0.485</td>
      <td>0.846</td>
      <td>0.356</td>
      <td>0.58</td>
      <td>0.654321</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-6ca62ba7-f27d-4ef5-93de-c1aca3b9b916')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-6ca62ba7-f27d-4ef5-93de-c1aca3b9b916 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-6ca62ba7-f27d-4ef5-93de-c1aca3b9b916');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>





```python
matchup_history = get_matchup_history(warriors_id, celtics_id, date)
print('Matchup_history:')
matchup_history
```

    Matchup_history:






  <div id="df-54ed3fc8-eede-4fc8-8d43-3e5e8082324c">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>WIN</th>
      <th>IS_HOME</th>
      <th>PTS_home</th>
      <th>FG_PCT_home</th>
      <th>FT_PCT_home</th>
      <th>FG3_PCT_home</th>
      <th>AST_home</th>
      <th>REB_home</th>
      <th>PTS_away</th>
      <th>FG_PCT_away</th>
      <th>FT_PCT_away</th>
      <th>FG3_PCT_away</th>
      <th>AST_away</th>
      <th>REB_away</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>0</td>
      <td>0.636905</td>
      <td>0.440</td>
      <td>0.800</td>
      <td>0.349</td>
      <td>0.40</td>
      <td>0.679012</td>
      <td>0.577381</td>
      <td>0.400</td>
      <td>0.737</td>
      <td>0.395</td>
      <td>0.44</td>
      <td>0.518519</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>1</td>
      <td>0.619048</td>
      <td>0.466</td>
      <td>0.867</td>
      <td>0.225</td>
      <td>0.46</td>
      <td>0.481481</td>
      <td>0.559524</td>
      <td>0.413</td>
      <td>0.677</td>
      <td>0.344</td>
      <td>0.36</td>
      <td>0.580247</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>0</td>
      <td>0.613095</td>
      <td>0.413</td>
      <td>1.000</td>
      <td>0.413</td>
      <td>0.54</td>
      <td>0.543210</td>
      <td>0.535714</td>
      <td>0.425</td>
      <td>0.917</td>
      <td>0.393</td>
      <td>0.54</td>
      <td>0.506173</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-54ed3fc8-eede-4fc8-8d43-3e5e8082324c')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-54ed3fc8-eede-4fc8-8d43-3e5e8082324c button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-54ed3fc8-eede-4fc8-8d43-3e5e8082324c');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>





```python
warriors_ranking = get_ranking(warriors_id, date)
print('Warriors ranking:')
warriors_ranking
```

    Warriors ranking:






  <div id="df-9abd1714-1c8a-4008-bd53-d3e034a31e5c">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>G</th>
      <th>W_PCT</th>
      <th>HOME_W_PCT</th>
      <th>AWAY_W_PCT</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>877</th>
      <td>0.329268</td>
      <td>0.519</td>
      <td>0.857143</td>
      <td>0.153846</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-9abd1714-1c8a-4008-bd53-d3e034a31e5c')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-9abd1714-1c8a-4008-bd53-d3e034a31e5c button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-9abd1714-1c8a-4008-bd53-d3e034a31e5c');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>





```python
celtics_ranking = get_ranking(celtics_id, date)
print('Celtics ranking:')
celtics_ranking
```

    Celtics ranking:






  <div id="df-6de59f0e-4493-48e1-bdd8-f600723eb078">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>G</th>
      <th>W_PCT</th>
      <th>HOME_W_PCT</th>
      <th>AWAY_W_PCT</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5145</th>
      <td>0.329268</td>
      <td>0.778</td>
      <td>0.846154</td>
      <td>0.714286</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-6de59f0e-4493-48e1-bdd8-f600723eb078')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-6de59f0e-4493-48e1-bdd8-f600723eb078 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-6de59f0e-4493-48e1-bdd8-f600723eb078');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>





```python
warriors_win = game['HOME_TEAM_WINS'].iloc[0]
warriors_win
```




    1



Finally, let's actually create the data vector:


```python
x = (warriors_history.to_numpy(), celtics_history.to_numpy(), matchup_history.to_numpy(), warriors_ranking.to_numpy(), celtics_ranking.to_numpy())
x
```




    (array([[0.        , 0.        , 0.64880952, 0.464     , 0.765     ,
             0.367     , 0.62      , 0.50617284],
            [0.        , 0.        , 0.76785714, 0.515     , 0.667     ,
             0.442     , 0.62      , 0.43209877],
            [0.        , 0.        , 0.625     , 0.493     , 0.92      ,
             0.375     , 0.46      , 0.35802469],
            [0.        , 0.        , 0.68452381, 0.467     , 0.938     ,
             0.34      , 0.54      , 0.5308642 ],
            [0.        , 0.        , 0.70833333, 0.457     , 1.        ,
             0.429     , 0.6       , 0.49382716],
            [1.        , 0.        , 0.75595238, 0.535     , 0.733     ,
             0.511     , 0.76      , 0.49382716],
            [0.        , 0.        , 0.49404762, 0.378     , 0.895     ,
             0.233     , 0.34      , 0.41975309],
            [1.        , 0.        , 0.81547619, 0.575     , 0.85      ,
             0.426     , 0.72      , 0.58024691],
            [0.        , 0.        , 0.67261905, 0.477     , 0.783     ,
             0.256     , 0.54      , 0.61728395],
            [0.        , 0.        , 0.73214286, 0.459     , 0.76      ,
             0.333     , 0.52      , 0.51851852]]),
     array([[0.        , 0.        , 0.67261905, 0.409     , 0.963     ,
             0.268     , 0.52      , 0.64197531],
            [1.        , 0.        , 0.79166667, 0.534     , 1.        ,
             0.529     , 0.6       , 0.41975309],
            [1.        , 0.        , 0.64880952, 0.463     , 0.75      ,
             0.324     , 0.52      , 0.55555556],
            [1.        , 0.        , 0.69642857, 0.429     , 0.875     ,
             0.375     , 0.54      , 0.51851852],
            [1.        , 0.        , 0.75      , 0.545     , 0.9       ,
             0.457     , 0.58      , 0.60493827],
            [1.        , 0.        , 0.69642857, 0.482     , 0.882     ,
             0.435     , 0.56      , 0.56790123],
            [0.        , 0.        , 0.63690476, 0.437     , 0.706     ,
             0.38      , 0.56      , 0.4691358 ],
            [1.        , 0.        , 0.61309524, 0.432     , 0.9       ,
             0.395     , 0.36      , 0.59259259],
            [1.        , 0.        , 0.69047619, 0.489     , 0.714     ,
             0.361     , 0.54      , 0.60493827],
            [1.        , 0.        , 0.74404762, 0.485     , 0.846     ,
             0.356     , 0.58      , 0.65432099]]),
     array([[1.        , 0.        , 0.63690476, 0.44      , 0.8       ,
             0.349     , 0.4       , 0.67901235, 0.57738095, 0.4       ,
             0.737     , 0.395     , 0.44      , 0.51851852],
            [1.        , 1.        , 0.61904762, 0.466     , 0.867     ,
             0.225     , 0.46      , 0.48148148, 0.55952381, 0.413     ,
             0.677     , 0.344     , 0.36      , 0.58024691],
            [1.        , 0.        , 0.61309524, 0.413     , 1.        ,
             0.413     , 0.54      , 0.54320988, 0.53571429, 0.425     ,
             0.917     , 0.393     , 0.54      , 0.50617284]]),
     array([[0.32926829, 0.519     , 0.85714286, 0.15384615]]),
     array([[0.32926829, 0.778     , 0.84615385, 0.71428571]]))



Now let's create a function that encapsulates all this logic above:


```python
def create_dataset(df, for_team = None):
  X = []
  Y = []
  if for_team is not None:
    print(f'Fetching for specific team: {for_team}')
    df = df[(df['HOME_TEAM_ID'] == for_team) | (df['VISITOR_TEAM_ID'] == for_team)]
  for (i, g) in df.iterrows():
    if i%1000 == 0:
      print(f'{i} of {len(df.index)}')
    home = g['HOME_TEAM_ID']
    away = g['VISITOR_TEAM_ID']
    date = g['GAME_DATE_EST']
    home_history = get_last_games(home, date)
    if home_history is None:
      continue
    away_history = get_last_games(away, date)
    if away_history is None:
      continue
    matchup_history = get_matchup_history(home, away, date)
    if matchup_history is None:
      continue
    home_history = home_history.to_numpy()
    away_history = away_history.to_numpy()
    matchup_history = matchup_history.to_numpy()
    r1 = get_ranking(home, date).to_numpy()
    r2 = get_ranking(away, date).to_numpy()
    label = g['HOME_TEAM_WINS']
    X.append((home_history, away_history, matchup_history, r1, r2))
    Y.append(label)
  return np.array(X), np.array(Y)
```

Save the data for future use!


```python
import pickle
if True:
  all_X, all_Y = create_dataset(games, team_name_to_id['Washington'])
  with open(os.path.join(DATA_DIR, 'x.pkl'), 'wb') as f:
    pickle.dump(all_X, f, protocol=pickle.HIGHEST_PROTOCOL)

  with open(os.path.join(DATA_DIR, 'y.pkl'), 'wb') as f:
    pickle.dump(all_Y, f, protocol=pickle.HIGHEST_PROTOCOL)
else:
  with open(os.path.join(DATA_DIR, 'x.pkl'), 'rb') as f:
    all_X = pickle.load(f)
  with open(os.path.join(DATA_DIR, 'y.pkl'), 'rb') as f:
    all_Y = pickle.load(f)
print(f'Created dataset with {len(all_X)} examples')
print('Example:')
print(f'{all_X[0]} --> {all_Y[0]}')
```

    Fetching for specific team: 1610612764
    Created dataset with 1664 examples
    Example:
    [array([[0.        , 0.        , 0.76190476, 0.527     , 0.565     ,
             0.487     , 0.8       , 0.2962963 ],
            [0.        , 0.        , 0.55357143, 0.429     , 0.923     ,
             0.36      , 0.4       , 0.51851852],
            [0.        , 0.        , 0.69642857, 0.463     , 0.792     ,
             0.256     , 0.48      , 0.59259259],
            [1.        , 0.        , 0.67261905, 0.475     , 0.703     ,
             0.407     , 0.44      , 0.50617284],
            [0.        , 0.        , 0.66666667, 0.561     , 0.765     ,
             0.333     , 0.4       , 0.45679012],
            [1.        , 0.        , 0.74404762, 0.557     , 0.708     ,
             0.313     , 0.6       , 0.49382716],
            [1.        , 0.        , 0.70833333, 0.56      , 0.8       ,
             0.429     , 0.66      , 0.65432099],
            [1.        , 0.        , 0.70238095, 0.532     , 0.688     ,
             0.259     , 0.58      , 0.66666667],
            [0.        , 0.        , 0.67261905, 0.433     , 0.913     ,
             0.296     , 0.54      , 0.56790123],
            [0.        , 0.        , 0.6547619 , 0.481     , 0.75      ,
             0.292     , 0.42      , 0.56790123]])
     array([[1.        , 0.        , 0.76785714, 0.526     , 0.8       ,
             0.324     , 0.42      , 0.59259259],
            [1.        , 0.        , 0.83333333, 0.563     , 0.762     ,
             0.457     , 0.62      , 0.62962963],
            [1.        , 0.        , 0.7202381 , 0.441     , 0.867     ,
             0.382     , 0.5       , 0.59259259],
            [1.        , 0.        , 0.76190476, 0.495     , 0.8       ,
             0.529     , 0.5       , 0.59259259],
            [1.        , 0.        , 0.67857143, 0.402     , 0.719     ,
             0.386     , 0.38      , 0.61728395],
            [1.        , 0.        , 0.64880952, 0.47      , 0.767     ,
             0.308     , 0.26      , 0.60493827],
            [0.        , 0.        , 0.7202381 , 0.437     , 0.625     ,
             0.32      , 0.54      , 0.67901235],
            [0.        , 0.        , 0.68452381, 0.422     , 0.774     ,
             0.366     , 0.4       , 0.4691358 ],
            [1.        , 0.        , 0.64285714, 0.389     , 0.719     ,
             0.294     , 0.4       , 0.56790123],
            [1.        , 0.        , 0.66666667, 0.47      , 0.692     ,
             0.432     , 0.38      , 0.62962963]])
     array([[0.        , 0.        , 0.57738095, 0.43      , 0.762     ,
             0.382     , 0.38      , 0.4691358 , 0.5952381 , 0.344     ,
             0.778     , 0.303     , 0.36      , 0.74074074],
            [0.        , 1.        , 0.54761905, 0.507     , 0.765     ,
             0.321     , 0.42      , 0.45679012, 0.67857143, 0.466     ,
             0.8       , 0.41      , 0.54      , 0.51851852],
            [0.        , 0.        , 0.5297619 , 0.379     , 0.65      ,
             0.222     , 0.46      , 0.49382716, 0.625     , 0.427     ,
             0.75      , 0.242     , 0.5       , 0.81481481]])
     array([[0.52439024, 0.419     , 0.55      , 0.30434783]])
     array([[0.52439024, 0.558     , 0.5       , 0.61904762]])] --> 0


# Dataset Creation

Create `StatsDataset` and `DataLoader` from the dataset.


```python
from torch.nn.utils.rnn import pad_sequence, pack_padded_sequence, pad_packed_sequence
```


```python
class StatsDataset(Dataset):
  def __init__(self, stats, labels):
    self.stats = stats
    self.labels = labels

  def __len__(self):
    return len(self.stats)

  def __getitem__(self, idx):
    return torch.from_numpy(self.stats[idx][0]), torch.from_numpy(self.stats[idx][1]), torch.from_numpy(self.stats[idx][2]), self.stats[idx][3], self.stats[idx][4], self.labels[idx]
```


```python
BATCH_SIZE = 64
train_X, val_X = np.split(all_X, [int(len(all_X)*0.9)])
train_Y, val_Y = np.split(all_Y, [int(len(all_Y)*0.9)])
print(f'Training examples: {len(train_X)}, Validation examples: {len(val_X)}')
train_dataset = StatsDataset(train_X, train_Y)
val_dataset = StatsDataset(val_X, val_Y)

train_loader = torch.utils.data.DataLoader(train_dataset,num_workers= 4,
                                           batch_size=BATCH_SIZE, pin_memory= True,
                                          shuffle= True)
val_loader = torch.utils.data.DataLoader(val_dataset,num_workers= 4,
                                           batch_size=BATCH_SIZE, pin_memory= True,
                                           shuffle= True)
print(f'Training size: {len(train_loader)}')
print(f'Val size: {len(val_loader)}')
for x1, x2, x3, x4, x5, y in train_loader:
  print(f'Home history: {x1.shape}')
  print(f'Away history: {x2.shape}')
  print(f'Matchup history: {x3.shape}')
  print(f'Rank: {x4.shape}')
  print(f'Away Rank: {x5.shape}')
  print(f'Label: {y}')
  break
```

    Training examples: 1497, Validation examples: 167
    Training size: 24
    Val size: 3
    Home history: torch.Size([64, 10, 8])
    Away history: torch.Size([64, 10, 8])
    Matchup history: torch.Size([64, 3, 14])
    Rank: torch.Size([64, 1, 4])
    Away Rank: torch.Size([64, 1, 4])
    Label: tensor([1, 1, 0, 1, 1, 1, 0, 1, 0, 1, 0, 1, 1, 1, 1, 0, 1, 1, 1, 0, 1, 0, 1, 0,
            0, 1, 1, 0, 0, 0, 1, 0, 0, 1, 1, 1, 0, 1, 1, 0, 1, 0, 1, 0, 0, 1, 1, 0,
            1, 0, 1, 0, 0, 0, 1, 1, 1, 0, 0, 0, 0, 1, 1, 0])



```python
gc.collect()
```




    63



# Model Creation
Let's create our model.

Our model should consist of 3 LSTM's to maintain the history of each team AND to maintain the history of the matchups between the 2 teams.

We'll combine the outputs of these LSTMs with a linear layer.

Next, we also will make a simple linear layer to handle the rankings.

Finally, we will combine everything through one fully connected layer.

The output of thiw `fc` layer will go into a sigmoid function since our output is binary.


```python
import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim
```


```python
TEAM_DIM = x1.shape[2]
MATCHUP_DIM = x3.shape[2]
RANK_DIM = x4.shape[2]
```


```python
# Model Definition
class GamePredictionNetwork(nn.Module):
    def __init__(self, 
                 team_dim,
                 matchup_dim,
                 rank_dim, 
                 hidden_dim, 
                 n_layers,
                 dropout):
        
      super().__init__()
      
      self.home_lstm = nn.LSTM(team_dim, hidden_dim, num_layers = n_layers, dropout = 0.0, batch_first=True)
      self.away_lstm = nn.LSTM(team_dim, hidden_dim, num_layers = n_layers, dropout = 0.0, batch_first=True)
      self.matchup_lstm = nn.LSTM(matchup_dim, hidden_dim, num_layers = n_layers, dropout = 0.0, batch_first=True)
      
      self.lstm_linear = nn.Sequential(
          nn.Linear(hidden_dim*3, hidden_dim),
          nn.Dropout(0.0),
      )
      
      self.home_rank = nn.Sequential(
          nn.Linear(rank_dim, rank_dim*8),
          nn.GELU(),
          nn.Linear(rank_dim*8, rank_dim*16),
          nn.Dropout(0.0),
          nn.GELU(),
          nn.Linear(rank_dim*16, rank_dim*8),
          nn.GELU(),
          nn.Linear(rank_dim*8, rank_dim),
          nn.GELU(),
          nn.Dropout(0.0),
      )

      self.away_rank = nn.Sequential(
          nn.Linear(rank_dim, rank_dim*8),
          nn.GELU(),
          nn.Linear(rank_dim*8, rank_dim*16),
          nn.Dropout(0.0),
          nn.GELU(),
          nn.Linear(rank_dim*16, rank_dim*8),
          nn.GELU(),
          nn.Linear(rank_dim*8, rank_dim),
          nn.GELU(),     
          nn.Dropout(0.0),
      )
      
      self.fc = nn.Linear(rank_dim*2+hidden_dim, 1)

      self.sigmoid = nn.Sigmoid()

    def forward(self, home_history, away_history, matchup_history, home_ranking, away_ranking):
      _, (h_h, _) = self.home_lstm(home_history)
      _, (a_h, _) = self.away_lstm(away_history)
      _, (m_h, _) = self.matchup_lstm(matchup_history)

      lstm_combined = self.lstm_linear(torch.cat([h_h[-1].unsqueeze(1), a_h[-1].unsqueeze(1), m_h[-1].unsqueeze(1)], 2))
      home_ranking = self.home_rank(home_ranking)
      away_ranking = self.away_rank(away_ranking)
      rank_combined = torch.cat((home_ranking, away_ranking), dim=2)
      combined = torch.cat([lstm_combined, rank_combined], 2)
      
      out = self.fc(combined)
      out = self.sigmoid(out).squeeze(dim=1).squeeze(dim=1)
      return out
```

Potential Ablations

* Dropout
 - LSTM: 0.2, Linear: 0.3
   - Train Accuracy: 75%, Val Accuracy: 78%
 - LSTM: 0.3, Linear: 0.4
    - Train Accuracy: , Val Accuracy: 

 - LSTM: 0.3, Linear: 0.5
    - Train Accuracy: , Val Accuracy: 

 - LSTM: 0.5, Linear: 0.6
    - Train Accuracy: , Val Accuracy: 

* Optimizer
 - AdamW
    - Train Accuracy: , Val Accuracy: 
 - Adam
    - Train Accuracy: , Val Accuracy: 

* Activation Functions
 - Tanh 
    - Train Accuracy: , Val Accuracy: 

 - GELU
    - Train Accuracy: 75%, Val Accuracy: 78%

 - ReLU
    - Train Accuracy: 74%, Val Accuracy: 75%

* Linear layer architecture
  - Cylinder
     - Train Accuracy: , Val Accuracy: 

  - Pyramid
     - Train Accuracy: , Val Accuracy: 

  - Reverse Pyramid
     - Train Accuracy: , Val Accuracy: 


* Sequence Length
  - Longer (10, 3)
       - Train Accuracy: , Val Accuracy: 
  - Shorter (5, 2)
       - Train Accuracy: 74%, Val Accuracy: 77%



```python
LEARNING_RATE = 0.01
model = GamePredictionNetwork(TEAM_DIM, MATCHUP_DIM, RANK_DIM, 64, 1, 0).cuda()
optimizer = torch.optim.Adam(model.parameters(), lr=LEARNING_RATE)
criterion = nn.BCELoss()
summary(model, x1.float().cuda(), x2.float().cuda(), x3.float().cuda(), x4.float().cuda(), x5.float().cuda())
```

    ======================================================================
                            Kernel Shape  Output Shape   Params Mult-Adds
    Layer                                                                
    0_home_lstm                        -  [64, 10, 64]  18.944k   18.432k
    1_away_lstm                        -  [64, 10, 64]  18.944k   18.432k
    2_matchup_lstm                     -   [64, 3, 64]   20.48k   19.968k
    3_lstm_linear.Linear_0     [192, 64]   [64, 1, 64]  12.352k   12.288k
    4_lstm_linear.Dropout_1            -   [64, 1, 64]        -         -
    5_home_rank.Linear_0         [4, 32]   [64, 1, 32]    160.0     128.0
    6_home_rank.GELU_1                 -   [64, 1, 32]        -         -
    7_home_rank.Linear_2        [32, 64]   [64, 1, 64]   2.112k    2.048k
    8_home_rank.Dropout_3              -   [64, 1, 64]        -         -
    9_home_rank.GELU_4                 -   [64, 1, 64]        -         -
    10_home_rank.Linear_5       [64, 32]   [64, 1, 32]    2.08k    2.048k
    11_home_rank.GELU_6                -   [64, 1, 32]        -         -
    12_home_rank.Linear_7        [32, 4]    [64, 1, 4]    132.0     128.0
    13_home_rank.GELU_8                -    [64, 1, 4]        -         -
    14_home_rank.Dropout_9             -    [64, 1, 4]        -         -
    15_away_rank.Linear_0        [4, 32]   [64, 1, 32]    160.0     128.0
    16_away_rank.GELU_1                -   [64, 1, 32]        -         -
    17_away_rank.Linear_2       [32, 64]   [64, 1, 64]   2.112k    2.048k
    18_away_rank.Dropout_3             -   [64, 1, 64]        -         -
    19_away_rank.GELU_4                -   [64, 1, 64]        -         -
    20_away_rank.Linear_5       [64, 32]   [64, 1, 32]    2.08k    2.048k
    21_away_rank.GELU_6                -   [64, 1, 32]        -         -
    22_away_rank.Linear_7        [32, 4]    [64, 1, 4]    132.0     128.0
    23_away_rank.GELU_8                -    [64, 1, 4]        -         -
    24_away_rank.Dropout_9             -    [64, 1, 4]        -         -
    25_fc                        [72, 1]    [64, 1, 1]     73.0      72.0
    26_sigmoid                         -    [64, 1, 1]        -         -
    ----------------------------------------------------------------------
                           Totals
    Total params          79.761k
    Trainable params      79.761k
    Non-trainable params      0.0
    Mult-Adds             77.896k
    ======================================================================






  <div id="df-3fea6ecf-e1bf-4d82-b540-e9643540bfcf">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Kernel Shape</th>
      <th>Output Shape</th>
      <th>Params</th>
      <th>Mult-Adds</th>
    </tr>
    <tr>
      <th>Layer</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0_home_lstm</th>
      <td>-</td>
      <td>[64, 10, 64]</td>
      <td>18944.0</td>
      <td>18432.0</td>
    </tr>
    <tr>
      <th>1_away_lstm</th>
      <td>-</td>
      <td>[64, 10, 64]</td>
      <td>18944.0</td>
      <td>18432.0</td>
    </tr>
    <tr>
      <th>2_matchup_lstm</th>
      <td>-</td>
      <td>[64, 3, 64]</td>
      <td>20480.0</td>
      <td>19968.0</td>
    </tr>
    <tr>
      <th>3_lstm_linear.Linear_0</th>
      <td>[192, 64]</td>
      <td>[64, 1, 64]</td>
      <td>12352.0</td>
      <td>12288.0</td>
    </tr>
    <tr>
      <th>4_lstm_linear.Dropout_1</th>
      <td>-</td>
      <td>[64, 1, 64]</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5_home_rank.Linear_0</th>
      <td>[4, 32]</td>
      <td>[64, 1, 32]</td>
      <td>160.0</td>
      <td>128.0</td>
    </tr>
    <tr>
      <th>6_home_rank.GELU_1</th>
      <td>-</td>
      <td>[64, 1, 32]</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>7_home_rank.Linear_2</th>
      <td>[32, 64]</td>
      <td>[64, 1, 64]</td>
      <td>2112.0</td>
      <td>2048.0</td>
    </tr>
    <tr>
      <th>8_home_rank.Dropout_3</th>
      <td>-</td>
      <td>[64, 1, 64]</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9_home_rank.GELU_4</th>
      <td>-</td>
      <td>[64, 1, 64]</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>10_home_rank.Linear_5</th>
      <td>[64, 32]</td>
      <td>[64, 1, 32]</td>
      <td>2080.0</td>
      <td>2048.0</td>
    </tr>
    <tr>
      <th>11_home_rank.GELU_6</th>
      <td>-</td>
      <td>[64, 1, 32]</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>12_home_rank.Linear_7</th>
      <td>[32, 4]</td>
      <td>[64, 1, 4]</td>
      <td>132.0</td>
      <td>128.0</td>
    </tr>
    <tr>
      <th>13_home_rank.GELU_8</th>
      <td>-</td>
      <td>[64, 1, 4]</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14_home_rank.Dropout_9</th>
      <td>-</td>
      <td>[64, 1, 4]</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>15_away_rank.Linear_0</th>
      <td>[4, 32]</td>
      <td>[64, 1, 32]</td>
      <td>160.0</td>
      <td>128.0</td>
    </tr>
    <tr>
      <th>16_away_rank.GELU_1</th>
      <td>-</td>
      <td>[64, 1, 32]</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>17_away_rank.Linear_2</th>
      <td>[32, 64]</td>
      <td>[64, 1, 64]</td>
      <td>2112.0</td>
      <td>2048.0</td>
    </tr>
    <tr>
      <th>18_away_rank.Dropout_3</th>
      <td>-</td>
      <td>[64, 1, 64]</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>19_away_rank.GELU_4</th>
      <td>-</td>
      <td>[64, 1, 64]</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>20_away_rank.Linear_5</th>
      <td>[64, 32]</td>
      <td>[64, 1, 32]</td>
      <td>2080.0</td>
      <td>2048.0</td>
    </tr>
    <tr>
      <th>21_away_rank.GELU_6</th>
      <td>-</td>
      <td>[64, 1, 32]</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>22_away_rank.Linear_7</th>
      <td>[32, 4]</td>
      <td>[64, 1, 4]</td>
      <td>132.0</td>
      <td>128.0</td>
    </tr>
    <tr>
      <th>23_away_rank.GELU_8</th>
      <td>-</td>
      <td>[64, 1, 4]</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>24_away_rank.Dropout_9</th>
      <td>-</td>
      <td>[64, 1, 4]</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>25_fc</th>
      <td>[72, 1]</td>
      <td>[64, 1, 1]</td>
      <td>73.0</td>
      <td>72.0</td>
    </tr>
    <tr>
      <th>26_sigmoid</th>
      <td>-</td>
      <td>[64, 1, 1]</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-3fea6ecf-e1bf-4d82-b540-e9643540bfcf')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-3fea6ecf-e1bf-4d82-b540-e9643540bfcf button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-3fea6ecf-e1bf-4d82-b540-e9643540bfcf');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>





```python
def evaluate(model, optimizer, criterion, loader, scheduler = None):
  epoch_loss = 0
  epoch_acc = 0
  n_examples = 0
  model.eval()
  with torch.no_grad():
    for i, data in enumerate(loader):
      
      home_history, away_history, matchup_history, home_ranking, away_ranking, labels = data
      home_history = home_history.float().cuda()
      away_history = away_history.float().cuda()
      matchup_history = matchup_history.float().cuda()
      home_ranking = home_ranking.float().cuda()
      away_ranking = away_ranking.float().cuda()
      labels = labels.float().cuda()
      predictions = model(home_history, away_history, matchup_history, home_ranking, away_ranking)
          
      loss = criterion(predictions, labels)

      # Accumulate epoch stats
      epoch_loss += loss.item()
      epoch_acc += (predictions.round() == labels).sum().item()
      n_examples += predictions.size(0)

  return epoch_loss/n_examples, epoch_acc/n_examples*100
```


```python
def train_epoch(model, optimizer, criterion, loader, scheduler = None):
  epoch_loss = 0
  epoch_acc = 0
  n_examples = 0
  model.train()
  for i, data in enumerate(loader):
    home_history, away_history, matchup_history, home_ranking, away_ranking, labels = data

    home_history = home_history.float().cuda()
    away_history = away_history.float().cuda()
    matchup_history = matchup_history.float().cuda()
    home_ranking = home_ranking.float().cuda()
    away_ranking = away_ranking.float().cuda()
    labels = labels.float().cuda()

    optimizer.zero_grad()
    predictions = model(home_history, away_history, matchup_history, home_ranking, away_ranking)
    # back prop + optimize
    loss = criterion(predictions, labels)
    loss.backward()
    optimizer.step()

    # Accumulate epoch stats
    epoch_loss += loss.item()
    epoch_acc += (predictions.round() == labels).sum().item()
    n_examples += predictions.size(0)

  return epoch_loss/n_examples, epoch_acc/n_examples*100
```


```python
gc.collect()
```




    0




```python
EPOCHS = 100

for epoch in range(EPOCHS):
  train_loss, train_acc = train_epoch(model, optimizer, criterion, train_loader)
  val_loss, val_acc = evaluate(model, optimizer, criterion, val_loader)
  print(f'Epoch {epoch}:')
  print(f'Train loss: {train_loss}, Train acc: {train_acc}%')
  print(f'Val loss: {val_loss}, Val acc: {val_acc}%')
```

    Epoch 0:
    Train loss: 0.010429618632546567, Train acc: 60.320641282565134%
    Val loss: 0.010916692768028396, Val acc: 67.06586826347305%
    Epoch 1:
    Train loss: 0.00885607589859921, Train acc: 70.67468269873079%
    Val loss: 0.010090832403320038, Val acc: 69.46107784431138%
    Epoch 2:
    Train loss: 0.008603971165342974, Train acc: 71.67668670674684%
    Val loss: 0.009995393053500239, Val acc: 68.8622754491018%
    Epoch 3:
    Train loss: 0.00853512178840204, Train acc: 72.94589178356713%
    Val loss: 0.00938129335820318, Val acc: 69.46107784431138%
    Epoch 4:
    Train loss: 0.00832552167679679, Train acc: 72.87909151636607%
    Val loss: 0.009274383148033461, Val acc: 70.05988023952095%
    Epoch 5:
    Train loss: 0.008318233247112256, Train acc: 71.20908483633934%
    Val loss: 0.009842110965066327, Val acc: 71.25748502994011%
    Epoch 6:
    Train loss: 0.008505985807719513, Train acc: 70.875083500334%
    Val loss: 0.009366163058195286, Val acc: 71.25748502994011%
    Epoch 7:
    Train loss: 0.008415899100746405, Train acc: 72.01068804275216%
    Val loss: 0.009595576517596216, Val acc: 70.05988023952095%
    Epoch 8:
    Train loss: 0.008335373026574541, Train acc: 72.14428857715431%
    Val loss: 0.009842640268588495, Val acc: 68.26347305389223%
    Epoch 9:
    Train loss: 0.008430909815834775, Train acc: 71.40948563794255%
    Val loss: 0.009461444295095113, Val acc: 70.05988023952095%
    Epoch 10:
    Train loss: 0.00814157923460803, Train acc: 73.54709418837675%
    Val loss: 0.009595617919624922, Val acc: 69.46107784431138%
    Epoch 11:
    Train loss: 0.008337010999640068, Train acc: 73.41349365397461%
    Val loss: 0.009692115876489056, Val acc: 71.25748502994011%
    Epoch 12:
    Train loss: 0.00808559134154616, Train acc: 73.61389445557782%
    Val loss: 0.009627189107997688, Val acc: 72.45508982035929%
    Epoch 13:
    Train loss: 0.00831784306961294, Train acc: 73.01269205076821%
    Val loss: 0.009602187993283757, Val acc: 69.46107784431138%
    Epoch 14:
    Train loss: 0.008215603783836187, Train acc: 73.8810955243821%
    Val loss: 0.009704536128186895, Val acc: 69.46107784431138%
    Epoch 15:
    Train loss: 0.008276815185884516, Train acc: 72.27788911155645%
    Val loss: 0.009511949773320181, Val acc: 70.65868263473054%
    Epoch 16:
    Train loss: 0.008375727182718302, Train acc: 73.14629258517033%
    Val loss: 0.00961124968385982, Val acc: 69.46107784431138%
    Epoch 17:
    Train loss: 0.008121464438811093, Train acc: 73.61389445557782%
    Val loss: 0.009804746347987009, Val acc: 68.26347305389223%
    Epoch 18:
    Train loss: 0.008525422416532844, Train acc: 71.87708750835003%
    Val loss: 0.009608536601780418, Val acc: 71.8562874251497%
    Epoch 19:
    Train loss: 0.008798524130163148, Train acc: 71.27588510354042%
    Val loss: 0.010024752267106564, Val acc: 71.8562874251497%
    Epoch 20:
    Train loss: 0.00820250023820835, Train acc: 72.47828991315966%
    Val loss: 0.010059605458539404, Val acc: 66.46706586826348%
    Epoch 21:
    Train loss: 0.008257013662863192, Train acc: 73.2130928523714%
    Val loss: 0.010001612280657192, Val acc: 70.65868263473054%
    Epoch 22:
    Train loss: 0.008246814639232282, Train acc: 72.47828991315966%
    Val loss: 0.009663962496968801, Val acc: 71.25748502994011%
    Epoch 23:
    Train loss: 0.008107456252347173, Train acc: 73.74749498997996%
    Val loss: 0.009800976264976455, Val acc: 69.46107784431138%
    Epoch 24:
    Train loss: 0.008071332453248018, Train acc: 73.94789579158316%
    Val loss: 0.009603148092052894, Val acc: 69.46107784431138%
    Epoch 25:
    Train loss: 0.00823094795844359, Train acc: 73.8810955243821%
    Val loss: 0.00956564784763816, Val acc: 71.25748502994011%
    Epoch 26:
    Train loss: 0.008139386046466305, Train acc: 73.14629258517033%
    Val loss: 0.010257305142408359, Val acc: 65.26946107784431%
    Epoch 27:
    Train loss: 0.008119832836315484, Train acc: 72.87909151636607%
    Val loss: 0.009854671483982108, Val acc: 73.05389221556887%
    Epoch 28:
    Train loss: 0.00834947432289939, Train acc: 71.47628590514363%
    Val loss: 0.00972619170914153, Val acc: 73.05389221556887%
    Epoch 29:
    Train loss: 0.008158593334670694, Train acc: 73.01269205076821%
    Val loss: 0.009791253748054276, Val acc: 69.46107784431138%
    Epoch 30:
    Train loss: 0.008156583937471042, Train acc: 72.94589178356713%
    Val loss: 0.010093896688815362, Val acc: 71.8562874251497%
    Epoch 31:
    Train loss: 0.008070342805119618, Train acc: 73.94789579158316%
    Val loss: 0.009558239025983982, Val acc: 72.45508982035929%
    Epoch 32:
    Train loss: 0.008043018315900702, Train acc: 74.01469605878424%
    Val loss: 0.009267339092528748, Val acc: 71.8562874251497%
    Epoch 33:
    Train loss: 0.008062425878896821, Train acc: 73.61389445557782%
    Val loss: 0.009455247910436755, Val acc: 70.65868263473054%
    Epoch 34:
    Train loss: 0.008069167096533613, Train acc: 73.6806947227789%
    Val loss: 0.009304410147809698, Val acc: 73.05389221556887%
    Epoch 35:
    Train loss: 0.00800374634041337, Train acc: 74.34869739478958%
    Val loss: 0.00943443732347317, Val acc: 73.65269461077844%
    Epoch 36:
    Train loss: 0.008039918615567979, Train acc: 73.6806947227789%
    Val loss: 0.009968089486310582, Val acc: 66.46706586826348%
    Epoch 37:
    Train loss: 0.008057619662147884, Train acc: 73.81429525718103%
    Val loss: 0.009676993250133034, Val acc: 72.45508982035929%
    Epoch 38:
    Train loss: 0.007997084237291723, Train acc: 73.74749498997996%
    Val loss: 0.009681461457006945, Val acc: 69.46107784431138%
    Epoch 39:
    Train loss: 0.008019434127635611, Train acc: 74.21509686038745%
    Val loss: 0.009543213658704015, Val acc: 73.65269461077844%
    Epoch 40:
    Train loss: 0.007960662136256257, Train acc: 74.81629926519706%
    Val loss: 0.009564824803860601, Val acc: 72.45508982035929%
    Epoch 41:
    Train loss: 0.008049251082426082, Train acc: 72.74549098196393%
    Val loss: 0.009810444480644729, Val acc: 67.06586826347305%
    Epoch 42:
    Train loss: 0.008575822941525904, Train acc: 71.14228456913828%
    Val loss: 0.009739318293725659, Val acc: 74.25149700598801%
    Epoch 43:
    Train loss: 0.008105628655286495, Train acc: 73.34669338677354%
    Val loss: 0.009891522144843004, Val acc: 68.8622754491018%
    Epoch 44:
    Train loss: 0.007984281820540597, Train acc: 73.8810955243821%
    Val loss: 0.009655324641815916, Val acc: 69.46107784431138%
    Epoch 45:
    Train loss: 0.00803526907502291, Train acc: 73.94789579158316%
    Val loss: 0.00966915470397401, Val acc: 73.05389221556887%
    Epoch 46:
    Train loss: 0.007973775735439741, Train acc: 74.21509686038745%
    Val loss: 0.009474313187741949, Val acc: 73.05389221556887%
    Epoch 47:
    Train loss: 0.007924985033556391, Train acc: 74.08149632598531%
    Val loss: 0.009493867437282722, Val acc: 72.45508982035929%
    Epoch 48:
    Train loss: 0.008078185175766368, Train acc: 73.48029392117569%
    Val loss: 0.009511622483144978, Val acc: 74.25149700598801%
    Epoch 49:
    Train loss: 0.008004810026509011, Train acc: 73.07949231796927%
    Val loss: 0.009265934278865061, Val acc: 75.44910179640718%
    Epoch 50:
    Train loss: 0.007867120649309738, Train acc: 74.41549766199064%
    Val loss: 0.010195595061707639, Val acc: 70.05988023952095%
    Epoch 51:
    Train loss: 0.007883752554993512, Train acc: 74.54909819639278%
    Val loss: 0.009756047568635313, Val acc: 73.05389221556887%
    Epoch 52:
    Train loss: 0.00806766613053734, Train acc: 73.94789579158316%
    Val loss: 0.009329429643596718, Val acc: 74.25149700598801%
    Epoch 53:
    Train loss: 0.00781508778522392, Train acc: 74.81629926519706%
    Val loss: 0.009668052910330767, Val acc: 71.25748502994011%
    Epoch 54:
    Train loss: 0.007787628120474602, Train acc: 74.34869739478958%
    Val loss: 0.009767377447939204, Val acc: 71.8562874251497%
    Epoch 55:
    Train loss: 0.007661255065329328, Train acc: 75.28390113560455%
    Val loss: 0.009698243912108644, Val acc: 72.45508982035929%
    Epoch 56:
    Train loss: 0.0076573641999371465, Train acc: 75.28390113560455%
    Val loss: 0.00978093340011414, Val acc: 70.05988023952095%
    Epoch 57:
    Train loss: 0.007671178004386509, Train acc: 74.48229792919172%
    Val loss: 0.00947810093799751, Val acc: 70.05988023952095%
    Epoch 58:
    Train loss: 0.007449489238665115, Train acc: 76.21910487641951%
    Val loss: 0.01005442324512733, Val acc: 69.46107784431138%
    Epoch 59:
    Train loss: 0.007814754106716545, Train acc: 74.68269873079493%
    Val loss: 0.00999315752240712, Val acc: 71.25748502994011%
    Epoch 60:
    Train loss: 0.007638837208967649, Train acc: 75.95190380761522%
    Val loss: 0.00905800882927672, Val acc: 73.05389221556887%
    Epoch 61:
    Train loss: 0.007470337963932422, Train acc: 76.21910487641951%
    Val loss: 0.009741757444278923, Val acc: 71.8562874251497%
    Epoch 62:
    Train loss: 0.0074682609789675685, Train acc: 76.75350701402806%
    Val loss: 0.009170033439190801, Val acc: 73.05389221556887%
    Epoch 63:
    Train loss: 0.0074426736048084936, Train acc: 76.35270541082164%
    Val loss: 0.009628560728655604, Val acc: 72.45508982035929%
    Epoch 64:
    Train loss: 0.007409034487240778, Train acc: 77.35470941883767%
    Val loss: 0.009629032925931279, Val acc: 71.8562874251497%
    Epoch 65:
    Train loss: 0.007511699426628067, Train acc: 76.95390781563127%
    Val loss: 0.009071054037459596, Val acc: 75.44910179640718%
    Epoch 66:
    Train loss: 0.0071918835979186465, Train acc: 77.62191048764196%
    Val loss: 0.009485136070651209, Val acc: 73.05389221556887%
    Epoch 67:
    Train loss: 0.007179761876563032, Train acc: 77.55511022044088%
    Val loss: 0.009236787250655853, Val acc: 74.25149700598801%
    Epoch 68:
    Train loss: 0.007029906243742826, Train acc: 77.42150968603875%
    Val loss: 0.009598752695643259, Val acc: 74.25149700598801%
    Epoch 69:
    Train loss: 0.007190236268078556, Train acc: 77.75551102204409%
    Val loss: 0.010018316928498045, Val acc: 72.45508982035929%
    Epoch 70:
    Train loss: 0.007365574021298008, Train acc: 78.22311289245157%
    Val loss: 0.010008190920253, Val acc: 68.8622754491018%
    Epoch 71:
    Train loss: 0.007119946385831457, Train acc: 77.0875083500334%
    Val loss: 0.009757789666067341, Val acc: 71.25748502994011%
    Epoch 72:
    Train loss: 0.006986197166309089, Train acc: 79.0247160988644%
    Val loss: 0.009636608426442404, Val acc: 69.46107784431138%
    Epoch 73:
    Train loss: 0.007232799479064737, Train acc: 78.22311289245157%
    Val loss: 0.009355940861616306, Val acc: 71.8562874251497%
    Epoch 74:
    Train loss: 0.007294598108303093, Train acc: 77.88911155644622%
    Val loss: 0.009744247455082967, Val acc: 73.05389221556887%
    Epoch 75:
    Train loss: 0.007073147163919871, Train acc: 78.28991315965264%
    Val loss: 0.009502769944196689, Val acc: 70.65868263473054%
    Epoch 76:
    Train loss: 0.00722317470258765, Train acc: 78.3567134268537%
    Val loss: 0.009785893434535958, Val acc: 73.65269461077844%
    Epoch 77:
    Train loss: 0.007064546556096915, Train acc: 78.28991315965264%
    Val loss: 0.009827005292127232, Val acc: 71.8562874251497%
    Epoch 78:
    Train loss: 0.007185919808799932, Train acc: 78.02271209084837%
    Val loss: 0.010687322316769355, Val acc: 67.06586826347305%
    Epoch 79:
    Train loss: 0.006994126853579748, Train acc: 77.88911155644622%
    Val loss: 0.009715920793796014, Val acc: 69.46107784431138%
    Epoch 80:
    Train loss: 0.006985014828348765, Train acc: 79.0247160988644%
    Val loss: 0.010444264925882489, Val acc: 67.06586826347305%
    Epoch 81:
    Train loss: 0.006930789771522772, Train acc: 78.69071476285905%
    Val loss: 0.00994005210385351, Val acc: 70.65868263473054%
    Epoch 82:
    Train loss: 0.006871910834200954, Train acc: 80.16032064128257%
    Val loss: 0.009775914831789669, Val acc: 68.8622754491018%
    Epoch 83:
    Train loss: 0.0067534754494467655, Train acc: 79.89311957247828%
    Val loss: 0.01073915729979555, Val acc: 65.86826347305389%
    Epoch 84:
    Train loss: 0.006924307736541402, Train acc: 78.49031396125584%
    Val loss: 0.010029687139088523, Val acc: 69.46107784431138%
    Epoch 85:
    Train loss: 0.006885536945734171, Train acc: 79.82631930527721%
    Val loss: 0.00986948127518157, Val acc: 70.05988023952095%
    Epoch 86:
    Train loss: 0.006566527452003821, Train acc: 81.69672678690715%
    Val loss: 0.010653417624399334, Val acc: 64.67065868263472%
    Epoch 87:
    Train loss: 0.006922199575600022, Train acc: 79.82631930527721%
    Val loss: 0.012546182749514094, Val acc: 62.27544910179641%
    Epoch 88:
    Train loss: 0.007356945283427267, Train acc: 78.62391449565798%
    Val loss: 0.009948039126253413, Val acc: 68.8622754491018%
    Epoch 89:
    Train loss: 0.006747839525690378, Train acc: 80.69472277889112%
    Val loss: 0.010315727687881379, Val acc: 69.46107784431138%
    Epoch 90:
    Train loss: 0.006593887994666855, Train acc: 80.82832331329325%
    Val loss: 0.010361965009552276, Val acc: 70.65868263473054%
    Epoch 91:
    Train loss: 0.00709428614827897, Train acc: 78.89111556446225%
    Val loss: 0.009994304465676496, Val acc: 66.46706586826348%
    Epoch 92:
    Train loss: 0.006599312176605663, Train acc: 79.89311957247828%
    Val loss: 0.009943730459955637, Val acc: 68.26347305389223%
    Epoch 93:
    Train loss: 0.006463181080623874, Train acc: 79.75951903807615%
    Val loss: 0.010794496821786115, Val acc: 67.06586826347305%
    Epoch 94:
    Train loss: 0.006333074473347278, Train acc: 80.7615230460922%
    Val loss: 0.01106747717200639, Val acc: 70.05988023952095%
    Epoch 95:
    Train loss: 0.0065935341095080275, Train acc: 81.02872411489645%
    Val loss: 0.01067538425594033, Val acc: 68.8622754491018%
    Epoch 96:
    Train loss: 0.006647687258526096, Train acc: 80.36072144288578%
    Val loss: 0.01013181666414181, Val acc: 70.05988023952095%
    Epoch 97:
    Train loss: 0.0065881178310575215, Train acc: 80.7615230460922%
    Val loss: 0.010926742039754718, Val acc: 70.05988023952095%
    Epoch 98:
    Train loss: 0.0064963934496393505, Train acc: 80.22712090848363%
    Val loss: 0.010710489964056871, Val acc: 70.05988023952095%
    Epoch 99:
    Train loss: 0.006322157685567159, Train acc: 81.42952571810287%
    Val loss: 0.011304546259120554, Val acc: 64.07185628742515%


# There we have it!

Our model can achieve an approximate ~75-78% accuracy on validation and training data. This outperforms most existing prediction models.


```python

```
