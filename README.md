# MovieLens-25M-project

#### It is a database about films rating by users between the late 1990s and early 2000.
#### The data cames from 3 databases:

  - users.dat
  - rating.csv
  - movies.csv
  
## users.dat

#### User information is in the file "users.dat" and is in the following format:

#### UserID::Gender::Age::Occupation::Zip-code

#### All demographic information is provided voluntarily by the users and is not checked for accuracy.  Only users who have provided some demographic information are included in this data set.

- Gender is denoted by a "M" for male and "F" for female
- Age is chosen from the following ranges:

	*  1:  "Under 18"
	* 18:  "18-24"
	* 25:  "25-34"
	* 35:  "35-44"
	* 45:  "45-49"
	* 50:  "50-55"
	* 56:  "56+"

- Occupation is chosen from the following choices:

	*  0:  "other" or not specified
	*  1:  "academic/educator"
	*  2:  "artist"
	*  3:  "clerical/admin"
	*  4:  "college/grad student"
	*  5:  "customer service"
	*  6:  "doctor/health care"
	*  7:  "executive/managerial"
	*  8:  "farmer"
	*  9:  "homemaker"
	* 10:  "K-12 student"
	* 11:  "lawyer"
	* 12:  "programmer"
	* 13:  "retired"
	* 14:  "sales/marketing"
	* 15:  "scientist"
	* 16:  "self-employed"
	* 17:  "technician/engineer"
	* 18:  "tradesman/craftsman"
	* 19:  "unemployed"
	* 20:  "writer"
  #### 

## rating.csv

#### All ratings are contained in the file "ratings.dat" and are in the following format:

#### UserID;MovieID;Rating;Timestamp

- UserIDs range between 1 and 6040 
- MovieIDs range between 1 and 3952
- Ratings are made on a 5-star scale (whole-star ratings only)
- Timestamp is represented in seconds since the epoch as returned by time(2)
- Each user has at least 20 ratings

## movies.csv

#### Movie information is in the file "movies.csv" and is in the following format:

MovieID;Title;Genres

- UserIDs range between 1 and 6040 
- MovieIDs range between 1 and 3952
- Ratings are made on a 5-star scale (whole-star ratings only)
- Timestamp is represented in seconds since the epoch as returned by time(2)
- Each user has at least 20 ratings

- Titles are identical to titles provided by the IMDB (includingyear of release)
- Genres are pipe-separated and are selected from the following genres:

	* Action
	* Adventure
	* Animation
	* Children's
	* Comedy
	* Crime
	* Documentary
	* Drama
	* Fantasy
	* Film-Noir
	* Horror
	* Musical
	* Mystery
	* Romance
	* Sci-Fi
	* Thriller
	* War
	* Western  
  
 ### Extracting data
 
 ```python
 unames=["user_id", "gender", "age", "occupation", "zip"]
users= pd.read_table("users.dat", sep="::", header=None, names=unames, engine="python")

rnames=["user_id", "movie_id", "rating", "timestamp"]
ratings= pd.read_csv("ratings.csv")

mnames=["movie_id", "title", "genres"]
movies=pd.read_csv("movies.csv")
```

### Cleaning the data
 
#### Due to the data being spread across three tables, the first thing to do is to merge them.
 
  ```python
 ratings.rename(columns = {'userId':'user_id', 'movieId':'movie_id'}, inplace = True)
 
 movies.rename(columns = {'movieId':'movie_id'}, inplace = True)
 
 data = pd.merge(pd.merge(ratings, users), movies)
 ```
 
### Transforming the data 
 
#### Once the tables are merged it was decided to study the mean films rated by gender, to then plotting the 20 best rated films by gender.
 
 ```python
 mean_ratings= data.pivot_table("rating", index="title", columns="gender", aggfunc="mean")
 ```
![image](https://github.com/EduardoJMR/MovieLens-25M-project/blob/master/images/Capture.JPG)

```python 
indexer=mean_ratings.sum(axis="columns").argsort()
 
 mean_ratings_20 = mean_ratings.take(indexer[-20:])
 ```
![image](https://github.com/EduardoJMR/MovieLens-25M-project/blob/master/images/Capture2.JPG)
 
```python 
mean_ratings_20 = mean_ratings_20.stack()
mean_ratings_20.name = "rating_by_gender"
mean_ratings_20 = mean_ratings_20.reset_index() 
```
 
### Visualizing the data
 
```python 
import seaborn as sns
sns.barplot(x="rating_by_gender",y="title",hue="gender", data= mean_ratings_20)
```
### The 20 best rated films by gender 
![image](https://github.com/EduardoJMR/MovieLens-25M-project/blob/master/images/Capture3.JPG)

## Top films among female viewers?

### Transforming the data
 
#### Now we have decided to study those titles receiving more than 250 ratings by gender. 

```python 
ratings_by_title=data.groupby("title").size()
 
most_rated_titles= ratings_by_title.index[ratings_by_title >= 250]

mean_ratings= mean_ratings.loc[most_rated_titles]
mean_ratings
```
![image](https://github.com/EduardoJMR/MovieLens-25M-project/blob/master/images/Capture4.JPG)

#### Once we have the film titles sorted by the 250 most rated we can study which has been rated by gender, female or male.

```python 
top_female_ratings= mean_ratings.sort_values("F", ascending=False)
top_female_ratings.head()
```
![image](https://github.com/EduardoJMR/MovieLens-25M-project/blob/master/images/Capture11.JPG)

## Movies that are most divisive between male and female viewers?

### Transforming the data

#### Once we have the titles' ratings sorted by gender we can measure the difference between them and get to know which titles have the greatest rating difference, sorting them by it.

```python 
mean_ratings["diff"]= mean_ratings["M"]-mean_ratings["F"]

sorted_by_diff= mean_ratings.sort_values("diff")
sorted_by_diff[::-1].head()
sorted_by_diff.tail()
```

![image](https://github.com/EduardoJMR/MovieLens-25M-project/blob/master/images/Capture6.JPG)

## Movies that elicited the most disagreement among viewers, independent of gender identification?

### Transforming the data

#### Suppose instead you wanted the movies that elicited the most disagreement among viewers, independent of gender identification. Disagreement can be measured by the variance or standard deviation of the ratings. To get this, we first compute the rating standard deviation by title and then filter down to the active titles 
 
```python 
rating_std_by_title= data.groupby("title")["rating"].std()
rating_std_by_title=rating_std_by_title.loc[most_rated_titles] 
 
rating_std_by_title.sort_values(ascending=False)[:10] 
```
![image](https://github.com/EduardoJMR/MovieLens-25M-project/blob/master/images/Capture7.JPG)


## Film titles classification by genre and group of age.

### Transforming the data

#### Another thing we can do with this database is classify the film's title by genre and group of age, to do that first we need to split the "genres" column in the amount of genres every film is classified with. Then once we have split the array of genres using .explode() function we can generate another dataframe with a genre per row.

```python
movies["genres"].head()
```
![image](https://github.com/EduardoJMR/MovieLens-25M-project/blob/master/images/Capture8.JPG)

```python
movies["genres"].head().str.split("|")
movies["genre"]=movies.pop("genres").str.split("|")
```

#### Now, calling movies.explode("genre") generates a new DataFrame with one row for each "inner" element in each list of movie genres. For example, if a movie is classified as both a comedy and a romance, then there will be two rows in the result, one with just "Comedy" and the other with just "Romance": 
 
```python
rating_with_genre= pd.merge(pd.merge(movies_exploded,ratings),users) 
rating_with_genre.iloc[0]
```
![image](https://github.com/EduardoJMR/MovieLens-25M-project/blob/master/images/Capture9.JPG)

```python
genre_ratings= (rating_with_genre.groupby(["genre", "age"])["rating"].mean().unstack("age"))
genre_ratings[:10]
```
![image](https://github.com/EduardoJMR/MovieLens-25M-project/blob/master/images/Capture10.JPG) 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
