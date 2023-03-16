# EPL-Betting-Model
## In use by professional handicappers at [Nelly's](https://www.nellysports.com/)

![epll](https://user-images.githubusercontent.com/73828790/225392231-83bdfec7-affc-46fa-97a6-0b41bb4101f0.jpeg)

### Overview
### Can we used advanced statistics and machine learning to predict winners of EPL matches?

The repository contains two R markdown files: 
#### 1. Creating, Training and Validating Model (EPL Model)
The XG Boost model is trained and validated on over 1,500 and 600, respectively, English Premiere League (EPL) matches. As a logit model, it generates three probabilities; the home team winning, visiting team winning, and a draw.

#### 2. Deploying the Model for Upcoming Games (EPL MW27.Run)
This file allows the user to deploy the model against the day’s games by after entering the teams playing one another and the posted odds for each team winning. The output is a csv file which shows the “edge” of each pick as defined by the implied probability of the given odds to win minus the probability of winning provided by the model.  

### Data Collection
The data used for this comes from [Football Reference](https://fbref.com/en/) via the WorldFootball package in R. While soccer has been played for decades, I only went back to the 2017-2018 season because that was the first which contained expected goals (xG), a statistic proven to be quite influential in predicting game outcomes.

### Data Cleaning 
Data cleaning was required to created consistency across variables. Specific efforts included:
1. Creating consistent team names, 
2. Assigning a single year to seasons that span across two year, and
3. Assigning each team a unique game ID

### Feature Engineering
With over 200 variables ingested through WorldfootballR package, I selected the ones I believe to have the most predictive value. In addition, I added several variables:
1. Assigned points to each outcome,
2. Created a Pythagorean record; a well-understood metric for determining what a team's point total "should be" based on goal differential,
3. Read in the posted odds for each team winning from [Football-Data](https://www.football-data.co.uk/englandm.php).

### Data Leakage
In training a model based on games already played, it was important to avoid any data leakage. Season to date and recent totals were calculated using *cumsum* and *roll away* formulas, respectively. In so doing, these formulas would include data from that day's games, data not available when making a prediction. As such, I subtracted all of the data from that day's games from each total.  This resulted in choosing a value of *k* for recent games one higher than the number of games you want to evaluate (e.g. if you want to look at the last 7 games, the value of *k* should be 8).

### Data Transformation
Heretofore, each observation in the data was for a team at each point in the season to enable calculations of the required inputs for the model. For the model to function properly, the teams playing against one another needed to be combined into a single observation.  This required the creation and assignment of a unique game ID to both teams playing in a match. The data was then split into home and away teams. A left join was performed based on the unique game ID to create a single observation for each game containing the data for both the home and away teams.  

### Model Creation 
After cleaning and feature engineering, I perform cross validation to determine the optimal number of trees to train. Because there are 3 possible outcomes in a soccer match, the model was trained using "multi:softprob" as the objective function and "mlogloss" as the evaluation metric, and 3 as the number of classes. 
The XGBoost model also requires the dependent variable to be an integer and the first level to be 0 requiring a small but important data transformation.

### Model Performance
The model was validated using matches from the 2020-21 and 2021-22 seasons. I want to ensure the model performs well on how the game is played today. The overall accuracy is 55%. With 95% confidence, we can expect the overall accuracy to fall between 51 and 59 percent. 

<img width="335" alt="Capture" src="https://user-images.githubusercontent.com/73828790/225404441-64c638f5-a3b2-4ddc-ba92-3a1de9f9cf69.PNG">

### Variable Importance
I created a variable importance matrix to show which variables are most influential in predicting winners. The visualization below shows the relative importance of each variable by size, the larger the area, the more important the variable. Not surprisingly, the implied probability of the odds (TmWinProbability) and post shot xG-goals allowed are the most important predictors.

![Picture2](https://user-images.githubusercontent.com/73828790/225406112-62c23fbc-7003-4325-8b3b-297d7bc7fe29.png)

### Deployment
To deploy the model, the user needs to execute 5 steps: 
1. Ingest the most recent game data
2. Input match data
3. Input status as home or away
3. Input each team's odds (American) of winning and draw
4. Run remaining code

#### Ingest data
To execute this step, the user must simply run the code already written.  All required cleaning and feature engineering to ensure the data’s format exactly matches what the model was trained on is included.    

#### Input Teams
In the chunck titled, "Create Data Upcoming Matches", the user must input each team in *alphabetical order* and their opponent. If they have a bye, the team still needs to be entered.  Additionally, the order of teams should be the same for each team with the home team listed first. 

<img width="725" alt="Capture" src="https://user-images.githubusercontent.com/73828790/225408791-b672d57d-be01-4a7e-a8d4-8b382bbfde80.PNG">

#### Indicate Home, Away, or Bye
In the H_A vector, the user should input an "H", "A", "B" for each team, again in alphabetical order.

#### Input Odds
To determine where the value lies, the user needs to input the American odds for each team to win in the respective odds vector.  The order is critically important. The first number in the Tm_win_odds vector should correspond to the team alphabetically first; the same should be done for the draw_odds vector. 

#### Run remaining code
The output of the code is a csv file containing the model's prediction with the necessary contextual information to determine the "edge" associated with each bet.  
<img width="848" alt="Capture" src="https://user-images.githubusercontent.com/73828790/225409884-f68145db-1cb1-4d0f-a1c2-a93b760d1418.PNG">

