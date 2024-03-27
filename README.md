# Playlist Generation

[Data Source](https://www.kaggle.com/datasets/joebeachcapital/30000-spotify-songs)

[Main Repository](https://github.com/jamesspalding/playlist-generator/)

[Demo Video](https://github.com/jamesspalding/playlist-generator/blob/main/spotifyRDemo.mp4)

### An R function that uses multiple regression to generate Spotify playlists. 

## <ins> Usage </ins>

```
playlist.create(givenID, n = 1, expl = T, en = T, fullData = F, export = F, name = "playlist")
```

The **playlist.create** function takes the following parameters:


* givenID: Spotify songID of a song you wish to generate a playlist based off of, no default
* n: desired length of playlist, default value = 1
* expl: exploratory mode, default = TRUE
* en: eliminate songs containing letters outside of the English alphabet, default value = TRUE
* fullData: return all song data of playlist songs, default value = FALSE
* export: export playlist as .csv file, default value = FALSE
* name: name of exported .csv file, default = "playlist"

### Exploratory Mode

When expl = TRUE, the songs from the playlist will be found recursively. The function will take the previously generated song and generate a song similar to it, rather than the given song. This will bring you farther away from your original song as the length expands, with the idea of "exploring" the genre to find new songs.

When expl = FALSE, all songs in the playlist will be selected with only the given song in mind. This will generate a playlist of songs most similar to the given song.

### Export

When export = TRUE, a .csv file will be generated into the present working directory.

This file can be input into the following website to export directly to any music streaming service:

https://www.tunemymusic.com/transfer/csv-to-spotify

## <ins> Process </ins>
   
### 1. Data Cleaning

Cleaning data was simple, only requiring a few lines of code and dplyer. I removed duplicate tracks and unnecessary variables from the dataset to prepare for analysis.

```
library(tidyverse)
songs = read.csv("spotify_songs.csv") 
songs = distinct(songs, track_id, .keep_all = T) %>%   #removing duplicate songs
  select(c(-track_album_id, -track_album_name,         #removing album info
           -playlist_name, -playlist_id, -mode))       #removing unnessesary info
```

### 2. Stepwise Factor Selction

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

### 3. Playlist Function

The goal of the function is to use the above regression function to narrow down a huge list of songs into a short list of "best" songs using the most important variables determined by the regression.

My initial plan was to use a series of T-Tests to determine song similarity, but I was not able to do it by variable, so I used *literal* difference as the deciding factor. Where $S_{gx_i}$ is a given song's $X_i$ value, $S_{cx_i}$ is a chosen song's $X_i$ value, and $Sim$ is the similarity:

$$Sim = \mid S_{gx_i} - S_{cx_i} \mid$$

Therefore, the smaller $Sim$ is, the more similar the variable in $S_c$ is to the variable in $S_g$.

As genre($X_1$) was found to be the most important variable in determining song popularity, it was the first filter I used.

```
genreDF = songs %>% 
   filter(playlist_genre == givenSong$playlist_genre) %>%
   filter(!track_id %in% playList$track_id) 
```

For instrumentalness, I ran into a problem with songs with an instrumentalness value of 0, which lead me to use an if statement to select only said songs in the case that the given song has a 0 instrumentalness value. In the case that instrumentalness $\not = 0$, I used the quantile() function to select only the songs within the smallest 57.5% of values.

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
```
## <ins> Testing </ins>
To investigate and test the model made using the step wise function, I first created a validation model using the validation data. I then compared the coefficients from each model to observe any under fitting or over fitting errors. From the data set most coefficients are fairly similar, but a few differences do stick out. The intercept, x1 r&b, x3, x2, x9, and x10 each have significant differences (+ or - 1.5 values) between training and validation models. 
```{r}
#validation model
best.model.validation <- lm(y.validation~x1.validation+x8.validation+x3.validation+x5.validation+x2.validation+x7.validation+x9.validation+x11.validation+x10.validation)

#comparing coefficients
coef<-data.frame(coefficients(best.model.training),coefficients(best.model.validation))

diff<-data.frame(coefficients(best.model.training)-coefficients(best.model.validation))
labels<-c("Differences")
colnames(diff)<-labels
names<-c("Intercept","x1.latin","x1.pop","x1.r&b","x1.rap","x1.rock","x8","x3","x5","x2","x7","x9","x11","x10")
diff<-cbind(names,diff)
ggplot(diff, aes(x=diff$names, y=diff$Differences)) +  geom_bar(stat = "identity", fill = "deepskyblue4", width = 0.7) +labs(title = "Comparing Coefficients", x = "Predictors",y = " Differences")
```

## <ins> Limitations </ins>

1. The generator only has access to the ~30,000 songs in the kaggle dataset compared to the 100+ million songs on Spotify, so accuracy is not ensured.
2. Songs are not always classified into their correct genre, as the dataset is based off of automatically generated playlists.
3. The language filter is not perfect, as it can only filter out languages not using the Latin alphabet, leaving many non-English European languages in the list.
4. Popularity is not necessarily the best predictor of likingness.
5. The function seems to have a high chance to break at longer playlist lengths.
