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
   
## 1. Data Cleaning

Cleaning data was simple, only requiring a few lines of code and dplyer. I removed duplicate tracks and unnecessary variables from the dataset to prepare for analysis.

```
library(tidyverse)
songs = read.csv("spotify_songs.csv") 
songs = distinct(songs, track_id, .keep_all = T) %>%   #removing duplicate songs
  select(c(-track_album_id, -track_album_name,         #removing album info
           -playlist_name, -playlist_id, -mode))       #removing unnessesary info
```

## 2. Stepwise Factor Selction

Data was first partitioned into training and validation sets in order to test for accuracy.

```
# setting a random training dataset from the song data to run the Stepwise process
set.seed(123)
training.data <- sort(sample(1:28356, 10000, replace = FALSE)) 

# creating the variables holding the training data
y.training <- y[training.data]; x1.training <- x1[training.data]; x2.training <- x2[training.data]; x3.training <- x3[training.data]; x4.training <- x4[training.data]
x5.training <- x5[training.data]; x6.training <- x6[training.data]; x7.training <- x7[training.data]; x8.training <- x8[training.data]
x9.training <- x9[training.data]; x10.training <- x10[training.data]; x11.training <- x11[training.data]

# finding the validation dataset, all 
validation.data <- setdiff(1:28356, training.data)

# creating the remaining variables holding the validation data
y.validation <- y[validation.data]; x1.validation <- x1[validation.data]; x2.validation <- x2[validation.data]; x3.validation <- x3[validation.data]; x4.validation <- x4[validation.data]
x5.validation <- x5[validation.data]; x6.validation <- x6[validation.data]; x7.validation <- x7[validation.data]; x8.validation <- x8[validation.data]
x9.validation <- x9[validation.data]; x10.validation <- x10[validation.data]; x11.validation <- x11[validation.data]
```

Next step was to use stepwise factor selection in order to determine importance of variables. The first step was conducted as follows:

```
### 1st Stepwise step
# gather p-values
pval.x1.latin=coefficients(summary(lm(y.training~x1.training)))[2,4]
pval.x1.pop=coefficients(summary(lm(y.training~x1.training)))[3,4]
pval.x1.randb=coefficients(summary(lm(y.training~x1.training)))[4,4]
pval.x1.rap=coefficients(summary(lm(y.training~x1.training)))[5,4]
pval.x1.rock=coefficients(summary(lm(y.training~x1.training)))[6,4]
pval.x2=coefficients(summary(lm(y.training~x2.training)))[2,4]
pval.x3=coefficients(summary(lm(y.training~x3.training)))[2,4]
pval.x4=coefficients(summary(lm(y.training~x4.training)))[2,4]
pval.x5=coefficients(summary(lm(y.training~x5.training)))[2,4]
pval.x6=coefficients(summary(lm(y.training~x6.training)))[2,4]
pval.x7=coefficients(summary(lm(y.training~x7.training)))[2,4]
pval.x8=coefficients(summary(lm(y.training~x8.training)))[2,4]
pval.x9=coefficients(summary(lm(y.training~x9.training)))[2,4]
pval.x10=coefficients(summary(lm(y.training~x10.training)))[2,4]
pval.x11=coefficients(summary(lm(y.training~x11.training)))[2,4]

# test which p-value is the smallest
pvals=c(pval.x1.latin,pval.x1.pop,pval.x1.randb,pval.x1.rap,pval.x1.rock,pval.x2,pval.x3,pval.x4,pval.x5,pval.x6,pval.x7,pval.x8,pval.x9,pval.x10,pval.x11)
which(pvals==min(pvals))  # 2nd entry in pvals returns the smallest p-value, which is the pop genre of variable x1 (playlist_genre)

pvals[2]  # 3.030825e-80 < .05
```

The first step concludes that the genre of the track being "pop" is the most important variable in determining popularity.

This process is repeated a total of 10 times to result in the following model to determine popularity:

$$\widehat{Popularity} = (46.84327861+Genre_I) - 7.33963458(Instrumentalness) - 20.96812717(Energy) + 1.44788261(Loudness) + 13.68405666(Danceability) + 6.28231307(Acousticness) - 4.12188004(Liveness) + 0.02403758(Tempo) - 2.32546662(Valence)$$

With categorical variable $genre_I$ where $I$ represents the follwing genres:

* Pop = 12.395
* Rock = 10.383
* Rap = 7.701
* Latin = 6.761
* R&B = 2.308
* EDM = 0

## 3. Playlist Function

The goal of the function is to use the above regression line to narrow down a huge list of songs into a short list of "best" songs using the most important variables determined by the regression line.

My initial plan was to use a series of T-Tests to determine song similarity, but I was not able to do it by variable, so I used *literal* difference as the deciding factor. Where $S_{gx_i}$ is a given song's $X_i$ value, $S_{cx_i}$ is a chosen song's $X_i$ value, and $Sim$ is the similarity:

$$Sim = \mid S_{gx_i} - S_{cx_i} \mid$$

Therefore, the smaller $Sim$ is, the more similar the variable in $S_c$ is to the variable in $S_g$.

Before starting, I initialized the repetition variable and playlist dataframe.

```
playList = songs %>%
   filter(track_id == givenID) #selects all data on selected song
r = 1 #used for repetition
while(r < n){ 
   #initialize song selection
   givenSong = playList[r,]
```

As genre($X_1$) was found to be the most important variable in determining song popularity, it was the first filter I used.

```
genreDF = songs %>% 
   filter(playlist_genre == givenSong$playlist_genre) %>%
   filter(!track_id %in% playList$track_id) 
```

For instrumentalness, I ran into a problem with songs with an instrumentalness value of 0, which lead me to use an if statement to select only said songs in the case that the given song has a 0 instrumentalness value. In the case that instrumentalness $\not = 0$, I used the quantile() function to select only the songs within the smallest 50% of values.

```
if(givenSong[1,14] == 0){ #Needed if/else statements to avoid an error
      tempDF = genreDF %>%
        filter(genreDF[,14] == 0)
    }   
else{
      similarity = data.frame(sim = numeric(nrow(genreDF)))
      for(i in 1:nrow(similarity)){
        similarity[i,] = abs(givenSong[1,14] - genreDF[i,14])
      }   
      
      tempDF = cbind(genreDF, similarity) %>% #selecting songs with bottom 50% closest
        filter(sim < quantile(similarity[,1])[3]) %>%
        select(-sim) #removing temp sim
   }
```

I was then able to generalize the rest of the variables using a for loop and a list of the variables, referenced by index.

```
variableIndex = c(9, 11, 8, 13, 15, 17, 16) #energy, loudness, danceability, acousticness, liveness, tempo, valence
for(x in variableIndex){ #loop through remaining variables
   similarity = data.frame(sim = numeric(nrow(tempDF))) 
   for(i in 1:nrow(similarity)){
      similarity[i,] = abs(givenSong[1,x] - tempDF[i,x])
   }
      
   tempDF = cbind(tempDF, similarity) %>% #selecting songs with bottom 50% closest
      filter(sim < quantile(similarity[,1])[3]) %>%
      select(-sim) #removing temp sim
}
```

Finally, I used a random number generator to select from the short list of "best" songs, and used the repetition variable to add it to the playlist dataframe.

```
    playList[r+1,] = tempDF[sample(1:nrow(tempDF), 1),] #using a random number in order to obtain unique playlists.
    r = r+1 #iteration
  }
  
  return(playList)
}
```
