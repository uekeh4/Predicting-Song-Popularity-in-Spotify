# Analyzing the Factors that Influence a Song's Popularity
### The Data
Our project uses a comprehensive dataset from Kaggle, featuring 943 of the most famous songs from Spotify's 2023 charts. The dataset includes 24 variables, which can be categorized into three main groups:

- __Song Details:__ Track name, artist name(s), artist count, and release date (year, month, day).

- __Streaming Success Metrics:__ Number of Spotify streams, playlist inclusions on Spotify, and chart presence/rank on Apple Music, Spotify, Deezer, and Shazam.

- __Audio Features:__ Quantitative measures of a song's attributes such as beats per minute (BPM), danceability, valence, energy, acousticness, instrumentalness, liveliness, and speechiness, as measured by Spotify's audio intelligence. The song's mode (major/minor) and key are also included.

### Visualizations
We created several visualizations to explore trends within the dataset:

- __Top 20 Artists by Song Count:__ A bar chart revealing the artists with the most songs on the charts.

- __Correlation Heatmap:__ A heatmap displaying the correlation between the numeric variables. Most correlations were found to be relatively weak.

- __Distribution of Streams by Key:__ A boxplot illustrating the distribution of stream counts across different musical keys. The analysis of variance (ANOVA) test found no significant difference in streams based on a song's key.

### Objectives
The primary goal of this project was to determine what constitutes a hit song. We aimed to:

1. __Analyze__ the characteristics and audio features of successful songs from 2023.

2. __Uncover__ recurring patterns and factors that influence music popularity.

3. __Predict__ a song's success (measured by streams) using various song attributes.

4. __Formulate__ recommendations for artists and record labels on how to create and release a hit song.


### Exploratory Analysis Statistics
- __Most Frequent Artist:__ Taylor Swift was identified as the most frequent artist in the dataset, followed by Bad Bunny and SZA.
<img src="https://i8.amplience.net/i/naras/Taylor-Swift-Eras-Tour-Brazil-GettyImages-1801110030.jpg" alt="Taylor Swift" width="300"/> <img src="https://s.yimg.com/ny/api/res/1.2/HAc6Jv2BXy8yQxWMR2E44Q--/YXBwaWQ9aGlnaGxhbmRlcjt3PTI0MDA7aD0xNjAwO2NmPXdlYnA-/https://s.yimg.com/os/creatr-uploaded-images/2025-03/d3998b90-fed1-11ef-9fbf-013757dd31af" alt="SZA" width="300"/> <img src="https://www.vibe.com/wp-content/uploads/2022/12/GettyImages-1420876485-e1672333142540.jpg?w=910&h=511&crop=1&resize=1440%2C810" alt="Bad Bunny" width="350"/>
- __Streams by Key:__ An analysis of variance (ANOVA) test was conducted to check for differences in stream counts across different keys. The test yielded a **p-value of 0.668**, which is greater than the typical significance level of 0.05, indicating no statistically significant difference in stream counts based on the song's key.

### Model Performance and Outcomes

- __Decision Tree and Random Forest MSE:__ The initial decision tree model had an **MSE of 478.45**, which only slightly improved to 475.4 after pruning. The random forest model, despite using various `mtry` values, had a best performance of an **MSE of 423.39**. Both were considered too high for the final analysis.

- __LASSO vs. Ridge Regression:__ The report notes that **LASSO had a lower test mean squared error (MSE) than Ridge regression**, leading to the selection of LASSO for variable selection.

- __Best Model Selection:__ After comparing several models using Best, Forward, and Backward Subset Selection, the six-variable model was chosen as the most effective.

- __Adjusted R-squared:__ The six-variable model had an adjusted **R-squared value of 0.4952**. This means that the variables in the model collectively explain approximately 49.52% of the variation in the normalized stream counts.

- __Coefficient Direction:__ The analysis indicates the following relationships with streams:

  - **Speechiness:** Has a negative effect, meaning an increase in a song's speechiness is associated with a decrease in streams.

  - **Danceability:** Has a positive effect, meaning higher danceability is associated with an increase in streams.

  - **Artist Count:** Has a negative impact, which suggests that increasing the number of artists does not necessarily lead to more streams.

  - **Playlist Presence:** Has a positive impact, indicating that songs in more playlists tend to have higher stream counts.

  - **Release Timing:** Songs released later in the year tend to have higher streams.

### Outcome
We attempted to predict a song's streams using several supervised learning methods.

- __Decision Trees & Random Forests:__ Both models had a very high Mean Squared Error (MSE), indicating they were not a good fit for our data. The random forest model, despite - being more robust, still performed poorly, likely due to the non-linear nature of the data.

- __Linear Regression:__ We transformed the `streams` variable using the Box-Cox method to normalize the data. We then used LASSO, Ridge Regression, and Subset Selection methods (Best, Forward, and Backward) to find the best-performing linear model.

- __Final Model:__ The six-variable linear regression model had the lowest residual standard error and highest adjusted R-squared value (0.4952), making it the most effective model.

The model's key findings were:

- __Positive Impact:__ Higher danceability and presence in Spotify playlists positively impact a song's streams. Songs released later in the year also tended to have more streams.

- __Negative Impact:__ Higher speechiness (more lyrics) and an increased artist count were negatively associated with stream counts. This contradicts some industry beliefs and prior research on artist collaborations.

- __Model Limitations:__ The model accounts for only 49.52% of the variation in streams, indicating that a significant portion of a song's success is influenced by factors not included in our analysis, such as marketing, social media virality, and artist popularity.

### Methods and Technology
Languages: R

Data Format: CSV

Data Processing: `dplyr` and `tidyverse` for data manipulation, cleaning, and transformation. The `boxcox()` function from the `MASS` library was used to normalize the response variable.

Machine Learning: `tree` for decision trees, `randomForest` for random forests, and `glmnet` for LASSO and Ridge regression. `leaps` was used for subset selection.

Data Visualization: `ggplot2` for creating plots and `ggcorrplot` for the correlation heatmap.
#### In collaboration with Lauren Ellis, Orhan Gozutok, and Ava Meissner
### Relevant References
* **Dataset:** The original Spotify 2023 dataset was obtained from Kaggle.
   - [https://www.kaggle.com/datasets/nelgiriyewithana/top-spotify-songs-2023](https://www.kaggle.com/datasets/nelgiriyewithana/top-spotify-songs-2023)
* **External Studies:** Our analysis was informed by previous research on music popularity and machine learning methods.
   - Ordanini, A., Nunes, J. C., & Valsesia, F. (2018). The featuring phenomenon in music...
   - Lee, J., & Lee, J. S. (2018). Music Popularity: Metrics, Characteristics, and Audio-based Prediction...
