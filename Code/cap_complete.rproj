---
title: "Untitled"
author: "Uchenna Ekeh"
date: "2024-05-29"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```


```{r }
library(dplyr)
library(tidyverse)
library(ggcorrplot)
library(tree)
library(randomForest)
spotify <- read.csv("spotify-2023.csv", fileEncoding = "UTF-8")
spotify$streams <- as.numeric(spotify$streams)
spotify$numeric_key <- as.numeric(factor(spotify$key))
spotify$in_shazam_charts  <- as.numeric(spotify$in_shazam_charts)
spotify$in_deezer_playlists  <- as.numeric(spotify$in_deezer_playlists)
spotify<- na.omit(spotify)
summary(spotify)
```



```{r }
top_artists <- spotify %>%
  count(artist.s._name) %>%
  top_n(20, n)

top_artists <- top_artists %>%
 rename(artist = artist.s._name) %>%
 arrange(desc(n))


top_artists$artist <- factor(top_artists$artist, levels = rev(top_artists$artist))


top_artists %>%
 ggplot(aes(artist, n)) +
 geom_bar(stat = "identity", fill = "skyblue", width = .9) +
 geom_text(aes(label = n), vjust = 0, size = 3) +
  ylim(0, 7)+
 theme(axis.text.x = element_text(angle = 45, hjust = 1)) +
 labs(x = "Artist", y = "Count", title = "Top 20 Artists by Song Count")
```




```{r}
spotify2 <- subset(spotify, select = -c(track_name, artist.s._name, in_apple_playlists, in_apple_charts, in_deezer_playlists, in_deezer_charts, in_shazam_charts, key, in_spotify_charts, released_day, released_year))
numeric <-  spotify2[,unlist(lapply(spotify2, is.numeric))]
cor <- cor(numeric)
ggcorrplot(cor)
```



```{r}
key_streams <- subset(spotify, !is.na(key), select = c(key, streams))
key_streams <- na.omit(key_streams)


ggplot(key_streams, aes(x = factor(key), y = streams)) +
  geom_boxplot(fill = "skyblue") +
  ylim(0, 1500000000)
  labs(x = "Key", y = "Streams", title = "Distribution of Streams by Key")
```



```{r}
library(car)
spotify_new <- subset(spotify, select = -c(track_name, artist.s._name, streams, in_apple_playlists, in_apple_charts, in_deezer_playlists, in_deezer_charts, in_shazam_charts, key, in_spotify_charts, released_day, released_year)) # Remove unneeded variables
```


```{r}
set.seed(1)
# Normalize streams
library(MASS)
b <- boxcox(lm(streams ~ 1, data = spotify))
# Extract lambda
lambda <- b$x[which.max(b$y)]
lambda
spotify$norm_streams <- (spotify$streams ^ lambda - 1) / lambda
spotify_new <-  spotify[,unlist(lapply(spotify, is.numeric))]
train <- sample(1:nrow(spotify_new), nrow(spotify_new) / 2)
tree.spotify <-tree(norm_streams ~., data = spotify_new, subset = train) #smaller tree
#, control = tree.control(nobs = length(train), mindev = 0) for big tree
summary(tree.spotify)
plot(tree.spotify, cex.lab = 0.8)
text(tree.spotify, pretty = 0)
#cross validation
cv.spotify <- cv.tree(tree.spotify)
plot(cv.spotify$size, cv.spotify$dev, type = "b")
#our attempt at pruning
prune.spotify <- prune.tree(tree.spotify, best = 5)
plot(prune.spotify)
text(prune.spotify, pretty = 0)
yhat <- predict(tree.spotify, newdata = spotify_new[-train,])
spotify.test <- spotify_new[-train, "norm_streams"]
plot(yhat, spotify.test)
abline(0,1)
mean((yhat- spotify.test)^2)
```



```{r}
set.seed(1)
rf.spotify <- randomForest(norm_streams ~ ., data = spotify_new, subset = train, mtry = 5, importance = TRUE)
yhat.rf <-predict(rf.spotify, newdata = spotify_new[-train, ])
mean((yhat.rf- spotify.test)^2)
importance(rf.spotify)
```


```{r}
mod <- lm(norm_streams ~ bpm + danceability_. + liveness_. + acousticness_. + valence_. + energy_. + instrumentalness_. + speechiness_., data = spotify)
summary(mod)
plot(mod)
```


```{r}
# LASSO
library(glmnet)
library(dplyr)
set.seed(8)
samp <- sample(seq_len(nrow(spotify_new)),  size = 0.75*nrow(spotify_new))
train <- spotify_new[samp,]
test <- spotify_new[-samp,]
xtrain <- model.matrix(norm_streams~., data=train)[,-1]
ytrain <- train$norm_streams
xtest <-model.matrix(norm_streams~., data=test)[,-1]
ytest <- test$norm_streams
lasso <- glmnet(xtrain, ytrain, alpha=1)
cv.lasso <- cv.glmnet(xtrain, ytrain,alpha=1)
plot(cv.lasso)
bestlam <- cv.lasso$lambda.min
lasso.pred <- predict(lasso ,s=bestlam ,newx=xtest)
lasso.err <- mean((lasso.pred - ytest)^2)
lasso.err
predict(lasso, type = "coefficients", s = bestlam)
```


```{r}
# Ridge Regression
set.seed(8)
xtrain <- model.matrix(norm_streams~., data=train[,-1])
ytrain <- train$norm_streams
ridge <- glmnet(xtrain,ytrain,alpha=0)
cvridge <- cv.glmnet(xtrain,ytrain,alpha=0)
plot(cvridge)
# Best lambda
bestlam <- cvridge$lambda.min
ridgepred <- predict(ridge, s = bestlam, newx = xtest)
ridge.error <- mean((ridgepred - ytest)^2)
ridge.error
predict(ridge, type = "coefficients", s = bestlam) # Final model
```



```{r}
# Best Subset Selection
library(leaps)
set.seed(8)
regfit.full = regsubsets(norm_streams~., spotify_new, nvmax=13)
regfit.sum <- summary(regfit.full)
which.min(regfit.sum$cp)
which.min(regfit.sum$bic)
which.max(regfit.sum$adjr2)
coef(regfit.full, 5) # Check five-variable mod
```



```{r}
set.seed(1)
# Validation Set Approach
train = sample(c(TRUE,FALSE), nrow(spotify_new),rep=TRUE)
test =(! train )
# return all the models, up to a 13-variable model
regfit.best = regsubsets(norm_streams~., spotify_new[train,], nvmax=13)
test.mat = model.matrix(norm_streams~., spotify_new[test,])
val.errors = rep(NA,13)


for (i in 1:13){
        coefi = coef(regfit.best, id=i)
        pred = test.mat[,names(coefi)]%*%coefi
        val.errors[i] = mean((spotify_new$norm_streams[test]-pred)^2)
}
i <- which.min(val.errors)
coef(regfit.best, i)
```



```{r}
#Forward Stepwise Selection
library(leaps)
# Return all the models, up to a 13-variable model
set.seed(8)
forward.mod = regsubsets(norm_streams~., spotify_new, nvmax=13, method="forward")
#Output the best set of variables for each model size
forward.sum <- summary(forward.mod)
which.min(forward.sum$cp)
which.min(forward.sum$bic)
which.max(forward.sum$adjr2)
coef(forward.mod, 3) # Check 3-variable mod
```


```{r}
# Backward Stepwise Selection
# return all the models, up to a 13-variable model
set.seed(1)
backward.mod = regsubsets(norm_streams~., spotify_new, nvmax=13, method="backward")
backward.sum <- summary(backward.mod)
which.min(backward.sum$cp)
which.min(backward.sum$bic)
which.max(backward.sum$adjr2)
coef(backward.mod, 6) # Check six-variable mod
```



```{r}
# LASSO Model
lasso_mod <- lm(norm_streams ~ artist_count + released_month + in_spotify_playlists + speechiness_. + bpm + danceability_. + liveness_., data = spotify)
summary(lasso_mod)
```


```{r}
# Three variable model
three_var_mod <- lm(norm_streams ~ artist_count + released_month + in_spotify_playlists, data = spotify)
summary(three_var_mod)
```


```{r}
# Five variable model
five_var_mod <- lm(norm_streams ~ artist_count + released_month + in_spotify_playlists + speechiness_. + danceability_., data = spotify)
summary(five_var_mod)
```


```{r}
# Six variable model
six_var_mod <- lm(norm_streams ~ artist_count + released_month + in_spotify_playlists + speechiness_. + danceability_. + bpm, data = spotify)
summary(six_var_mod)
```


```{r}


hist(spotify_new$norm_streams,
     main = "Distribution of Normalized Streams",
     sub = "A Histogram of the Transformed Response Variable",
     xlab = "Normalized Stream Count",
     ylab = "Frequency",
     col = "#1DB954")

res = rstandard(six_var_mod)
qqnorm(res,main = "Normal Q-Q Plot of Standardized Residuals",
       xlab = "Theoretical Quantiles",
       ylab = "Standardized Residuals",
       pch = 19,
       col = "#1DB954")
qqline(res, col="red")

plot(fitted(six_var_mod), resid(six_var_mod), 
     main = "Residuals vs. Fitted Values Plot",
     sub = "Assessing the Assumptions of the Linear Model",
     xlab = "Fitted Values",
     ylab = "Residuals",
     pch = 19,
     col = "#1DB954")
abline(h = 0, col = "red", lty = 2, lwd = 2)

```








