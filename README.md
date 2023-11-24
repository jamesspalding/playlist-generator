# Playlist Generation

https://www.kaggle.com/datasets/joebeachcapital/30000-spotify-songs

### Goal: Create a model that takes a song as input and outputs a playlist of similar music.

### Process
* Input track_id and desired playlist length n
* Model searches through dataset for track (If not found, returns error)
* Filter out songs outside of specefied genre(s)
* Finds songs of similar tempo/other stats
* Use a random number generator to select one of the found songs
* Return track_id and track_title
* Using output, repeat process n-1 times until playlist of desired lengeth has been created

### Steps
1. Clean data
2. Group songs based on factors such as genre and sub-genre
3. Partition groups into smaller sub-groups based on factors such as tempo and danceability
4. Use a random number generator to create unique playlists

*Detail steps taken here*
