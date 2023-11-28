# Playlist Generation

https://www.kaggle.com/datasets/joebeachcapital/30000-spotify-songs

### Goal: Build a regression model that predicts song popularity based off of a variety of variables. Then use this model to create a function that takes a song as input and outputs a playlist of similar music.

### <ins> Process </ins>
**Regression Model:**
* Determine which factors are significant or can be dropped for song popularity using stepwise selection
* Plot findings with regression lines colored by genre
* Search for other possible groupings or correlation between 

**Playlist Function:**
* Input track_id and desired playlist length n
* Search through dataset for track (If not found, returns error)
* Filter out songs outside of specefied genre(s)
* Use T-Test to find similar songs
* Use a random number generator to select one of the found songs, weighted by popularity
* Return track_id and track_title
* Using output, repeat process n-1 times until playlist of desired lengeth has been created

### <ins> Steps </ins> 

Regression Model:

1. Clean data
2. Find significant factors in determining popularity
3. Create regression model to predict popularity based off significant variables found in step 2

Playlist Function:

1. Use T-test to find list of significantly similar songs
2. Use the popularity regression to predict the likeness of the songs in the list
3. Repeat on specified length of playlist
   
**1. Data Cleaning**

Cleaning data was simple, only requiring a few lines of code and dplyer. I removed duplicate tracks and unnecessary variables from the dataset to prepare for analysis.

```
library(tidyverse)
songs = read.csv("spotify_songs.csv") 
songs = distinct(songs, track_id, .keep_all = T) %>%   #removing duplicate songs
  select(c(-track_album_id, -track_album_name,         #removing album info
           -playlist_name, -playlist_id, -mode))       #removing unnessesary info
```

**2. Stepwise Factor Selction** 
