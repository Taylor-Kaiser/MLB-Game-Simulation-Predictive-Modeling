# PIPELINE OVERVIEW

## INPUT DATA
- Bullpen Traditional Data -> Provides traditional bullpen statistics for modeling
- Bullpen Advanced Data    -> Provides advanced bullpen statistics for modeling
- Daily Lineups            -> Incorporates daily MLB lineups and game-specific player inputs
- Park Factors             -> Includes park-specific factors used in game simulation

## PIPELINE
- `SavingRawData` -> Downloads Statcast data for input dates, up to 3 days at a time
- `CombineRawData` -> Merges all raw Statcast data files into one dataset
- `FullData` -> Cleans and organizes raw Statcast data into hitter, pitcher, plate appearance, and bullpen datasets
- `BacktestingFullData` -> Produces leakage-safe historical training data
- `GameDataset` -> Converts plate appearance data into game-level outcomes (`batter_games` and `pitcher_games`) for rolling feature generation and predictive modeling
- `RollingFeatures` -> Generates time-aware rolling statistics for hitters and pitchers
- `Plate_Appearance_Model` -> Predicts lineup-based plate appearance opportunity
- `todays_pa_projections` -> Generates daily expected plate appearances by hitter
- `GameSimulationEngine` -> Simulates complete MLB games using player skill profiles, park factors, bullpen data, and other game-specific inputs

# GOAL
Estimate probabilities for MLB game, hitter, and pitcher outcomes
using historical performance data, player matchups, and game-specific inputs.

The project aims to develop a simulation and predictive modeling framework
capable of generating probability estimates for a variety of MLB outcomes,
including:

## Game outcomes:
- Team scoring and game totals
- First-five-inning outcomes
- Combined team hits
- Game winner

## Hitter outcomes:
- Player hits
- Total bases
- Hits, runs, and RBI combinations
- Strikeouts

## Pitcher outcomes:
- Strikeouts
- Hits allowed
- Innings pitched
- Walks issued

# Data and Methodology

## Data
The project uses pitch-level MLB Statcast data as the primary source of historical
player performance information. Raw Statcast data is cleaned, combined, and
transformed into hitter, pitcher, plate appearance, and game-level datasets
for downstream analysis.

Additional inputs include bullpen statistics, daily MLB lineups, park factors,
and other game-specific information used to represent the conditions of each
simulated game.

## Historical Performance
Historical game-level data is used to generate time-aware rolling statistics
for hitters and pitchers. Rolling windows of 3, 5, 10, and 20 games capture
recent performance while maintaining a historical perspective.

The project also uses lagged historical data when constructing training
datasets to ensure that model inputs represent information that would have
been available prior to the corresponding game.

## Player Skill Profiles
Hitter and pitcher performance is represented through player-specific skill
profiles derived from historical performance and pitch-level data. These
profiles incorporate offensive, pitching, and pitch-type performance
characteristics used as inputs throughout the predictive modeling and
simulation process.

## Plate Appearance Modeling
A dedicated plate appearance model estimates expected playing time for
hitters based on lineup position and other game-specific inputs. These
estimates are applied to daily lineups to generate projected plate
appearances for each hitter before simulation.

## Game Simulation
The game simulation engine uses player skill profiles, projected plate
appearances, matchup adjustments, starting pitcher workload, bullpen
statistics, park factors, and other game-specific inputs to simulate complete
games.

Monte Carlo simulation is used to repeatedly simulate game events and produce
probability distributions for modeled hitter and pitcher outcomes. Starting
pitchers transition to bullpen usage based on simulated workload, allowing
relief-pitching performance to influence later plate appearances.

# Component Descriptions

## SavingRawData

Downloads Statcast data for a user-defined date range, with requests limited
to three days at a time to ensure reliable data retrieval. The resulting raw
data files serve as the primary pitch-level input for the downstream pipeline.

## CombineRawData

Combines individual Statcast data files into a unified dataset for downstream
processing and analysis.

## FullData

Cleans and organizes the combined Statcast dataset into core hitter, pitcher,
plate appearance, and bullpen datasets. These datasets provide the foundation
for feature engineering, historical analysis, and subsequent modeling.

## BacktestingFullData

Creates a historical database using lagged data to ensure that training
datasets only use information available prior to each observation. This
provides a leakage-safe foundation for reliable model training and
backtesting.

## GameDataset

Constructs game-level outcome datasets from plate appearance data, creating
one row per game for each batter and pitcher.

The resulting `batter_games` and `pitcher_games` datasets provide the historical
game-level foundation for rolling features and subsequent predictive modeling.

## RollingFeatures

Generates time-aware rolling features for hitters and pitchers using the
game-level datasets produced by `GameDataset`. Rolling windows include the
previous 3, 5, 10, and 20 games and capture recent performance across key
hitting and pitching statistics.

For hitters, features include hits, strikeouts, walks, home runs, total bases,
and rate statistics. For pitchers, features include strikeouts, walks, hits
allowed, and home runs allowed, along with derived metrics such as damage
allowed, K/BB rates, and workload indicators.

These features create a leakage-safe historical performance layer that
represents information available prior to each game.

## Plate_Appearance_Model

Predicts the expected number of plate appearances for each hitter based on
lineup position and other relevant game-specific inputs. The model provides
the plate appearance estimates used directly by `todays_pa_projections`.

## todays_pa_projections

Generates daily plate appearance projections by applying the
`Plate_Appearance_Model` to the day's expected lineups and game-specific
inputs. The resulting projections provide expected playing-time inputs for
the game simulation engine.

## GameSimulationEngine

The game simulation engine uses player-specific skill profiles, projected
plate appearances, and game-specific inputs to simulate complete MLB games.
The engine incorporates hitter-pitcher matchup adjustments, starting pitcher
workload, bullpen transitions, and team bullpen statistics to model player
outcomes across multiple innings.

The simulator models hitter outcomes including hits, total bases, strikeouts,
walks, and home runs, as well as pitcher outcomes including innings pitched,
strikeouts, hits allowed, walks allowed, and pitch count. Starting pitchers
transition to the bullpen based on simulated workload rather than fixed
innings, while late-game plate appearances incorporate bullpen performance
metrics such as xwOBA, strikeout rate, walk rate, and home run rate.

The engine produces probability distributions for modeled player and game
outcomes and is being further developed through additional matchup
interactions, game-state logic, and historical calibration.


# Future Improvements

The project is actively being developed, with several areas planned for future expansion:

- Game-Level Outcomes: Expand the simulation engine beyond player-level outcomes to model how individual events translate into runs, scoring, and game results, including game winners and first-five-inning outcomes.
- Base State and Baserunning: Incorporate base states, baserunning, and stolen-base outcomes to provide a more complete representation of game situations.
- Historical Calibration: Continue calibrating simulated outcomes against historical MLB data and evaluating model performance across different outcome types.
- Rolling Bullpen Statistics: Incorporate time-aware rolling statistics for bullpen performance to better represent recent relief-pitching form.
- Expanded Historical Data: Incorporate multiple seasons of historical data to develop more complete hitter and pitcher profiles rather than relying primarily on current-season statistics.
- Pinch-Hitter Integration: Incorporate pinch hitters into the game simulation to account for lineup changes and additional player substitutions during games.

# Tools & Technologies

## Programming Language
- R

## R Packages
- baseballr
- dplyr
- jsonlite
- Lahman
- purrr
- readr
- slider
- tidyr
- xgboost

## Methods & Data
- Monte Carlo Simulation
- MLB Statcast
