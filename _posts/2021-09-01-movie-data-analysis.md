---
title: Movie data analysis
date: 2021-09-01T19:02:00+00:00
layout: single
permalink: /2021/09/movie-data-analysis/
classes: wide
toc: true
---
## Intro
I decided to dig into my film-watching habits as a data exploration exercise. 

I _used_ to watch a _lot_ of films. And while watching movies is no longer a significant pastime for me, it was fun at the time. Aside from the apparent interest in the films themselves, the extensive pop-culture knowledge I gained acted as some social currency. Back then, I would make the (outrageous) claim that I'd probably watched every film that would ever come up in casual conversation.

After each film I watched, I would immediately then give it a rating out of 10 on [IMDb](https://www.imdb.com/). I didn't think I was a critic - this was just the best way to get recommendations for even more films in the pre-Netflix era. A pleasant side effect of this ritual is that I now have a record of every movie I ever watched in that time, along with when I watched it, what I thought of it and other metadata. With the help of IMDb's data export feature, I now have my hands on my own little cinematic time capsule _!_ This blog post aims to take a look at that data in pursuit of insights. 

### Some of the questions I want to answer
- What were my viewing habits?
- Did I have interesting or unusual tastes, or was I poser? 
- How do my ratings compare to other IMDB users?

### Housekeeping notes
- I've written this post as a [Jupyter](https://jupyter.org/) notebook, with inline code snippets, and exported as markdown (you can see the original notebook [here](https://github.com/ma489/imdb/blob/main/imdb_data.ipynb)). 
 - If you're not familiar with Jupyter, it's a [notebook environment](https://en.wikipedia.org/wiki/Notebook_interface) that enables [literate programming](https://en.wikipedia.org/wiki/Literate_programming).
- I will keep my dataset private (see [here](https://arxiv.org/abs/cs/0610105) for why _!_), but you can grab your own dataset [here](https://www.imdb.com/list/ratings). 
- I'll be using the terms _'movies'_ and _'films'_ interchangably throughout. I'm sure there's some technical distinction, but generally I think [the former is an Americanism](https://en.wiktionary.org/wiki/movie#Noun).
- I'll be making use of the two canonical Python libraries: [`pandas`](https://pandas.pydata.org/) (for data analysis) and [`seaborn`](https://seaborn.pydata.org/) (for visualisation).

## Setup
First, some (non-interesting) initial setup


```python
# Imports
import pandas as pd
import seaborn as sns

# Configuration
pd.options.display.float_format = '{:.1f}'.format
pd.options.mode.chained_assignment = None  # default='warn'
sns.set(rc={'figure.figsize':(16,12)})
```


```python
# Let's read in the data
INPUT_FILE_ENCODING = "ISO-8859-1"
input_data_path = "imdb_ratings.csv"
imdb_data = pd.read_csv(input_data_path, encoding=INPUT_FILE_ENCODING)
# ... and preview it
imdb_data.head(1)
```

<style>
  table {
    display: block;
    max-width: -moz-fit-content;
    max-width: fit-content;
    margin: 50 auto;
    overflow-x: auto;
    white-space: nowrap;
  }
</style>


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
      <th>Const</th>
      <th>Your Rating</th>
      <th>Date Rated</th>
      <th>Title</th>
      <th>URL</th>
      <th>Title Type</th>
      <th>IMDb Rating</th>
      <th>Runtime (mins)</th>
      <th>Year</th>
      <th>Genres</th>
      <th>Num Votes</th>
      <th>Release Date</th>
      <th>Directors</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>tt0100053</td>
      <td>1</td>
      <td>2013-01-18</td>
      <td>Loose Cannons</td>
      <td>https://www.imdb.com/title/tt0100053/</td>
      <td>movie</td>
      <td>4.9</td>
      <td>94.0</td>
      <td>1990</td>
      <td>Action, Comedy, Crime, Thriller</td>
      <td>4394.0</td>
      <td>1990-02-09</td>
      <td>Bob Clark</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Let's perform some minor cleanup of the data
# ... filtering out the non-movie entries (TV shows, video games, etc.)
movies = imdb_data[imdb_data['Title Type'] == 'movie']
# ... and formatting the date field
movies['Date Rated'] = pd.to_datetime(movies['Date Rated'])
```

Great, now we're ready to explore!

## Summary
Next, let's look at some [summary statistics](https://en.wikipedia.org/wiki/Summary_statistics) to get an overview of the dataset. Think of this as a [tl;dr](https://en.wiktionary.org/wiki/tl;dr#Phrase).

We have different datatypes in our dataset, so we'll summarize these separately:


```python
# Numeric fields
movies.describe()
```




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
      <th>Your Rating</th>
      <th>IMDb Rating</th>
      <th>Runtime (mins)</th>
      <th>Year</th>
      <th>Num Votes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>869.0</td>
      <td>868.0</td>
      <td>868.0</td>
      <td>869.0</td>
      <td>868.0</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>5.1</td>
      <td>6.8</td>
      <td>111.6</td>
      <td>2000.3</td>
      <td>273587.3</td>
    </tr>
    <tr>
      <th>std</th>
      <td>2.2</td>
      <td>1.1</td>
      <td>22.2</td>
      <td>11.7</td>
      <td>320816.3</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.0</td>
      <td>2.1</td>
      <td>60.0</td>
      <td>1941.0</td>
      <td>88.0</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>4.0</td>
      <td>6.2</td>
      <td>96.0</td>
      <td>1995.0</td>
      <td>65191.5</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>6.0</td>
      <td>6.9</td>
      <td>107.0</td>
      <td>2002.0</td>
      <td>166071.5</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>7.0</td>
      <td>7.6</td>
      <td>123.2</td>
      <td>2009.0</td>
      <td>361292.5</td>
    </tr>
    <tr>
      <th>max</th>
      <td>10.0</td>
      <td>9.3</td>
      <td>210.0</td>
      <td>2018.0</td>
      <td>2301033.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Object fields
movies.describe(include=[object])
```




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
      <th>Const</th>
      <th>Title</th>
      <th>URL</th>
      <th>Title Type</th>
      <th>Genres</th>
      <th>Release Date</th>
      <th>Directors</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>869</td>
      <td>869</td>
      <td>869</td>
      <td>869</td>
      <td>868</td>
      <td>868</td>
      <td>869</td>
    </tr>
    <tr>
      <th>unique</th>
      <td>869</td>
      <td>867</td>
      <td>869</td>
      <td>1</td>
      <td>300</td>
      <td>835</td>
      <td>533</td>
    </tr>
    <tr>
      <th>top</th>
      <td>tt0277371</td>
      <td>The Omen</td>
      <td>https://www.imdb.com/title/tt0061695/</td>
      <td>movie</td>
      <td>Comedy</td>
      <td>2007-06-12</td>
      <td>Steven Spielberg</td>
    </tr>
    <tr>
      <th>freq</th>
      <td>1</td>
      <td>2</td>
      <td>1</td>
      <td>869</td>
      <td>56</td>
      <td>3</td>
      <td>14</td>
    </tr>
  </tbody>
</table>
</div>



OK, so I've watched (at least) 869 films. A quick glance of the IMDb website shows that figure includes 96 of [their Top 250 films](https://www.imdb.com/chart/top/) (and also 6 of [their bottom 100 films](https://www.imdb.com/chart/bottom)). It goes without saying that 869 is a lower-bound for my total lifetime count, since I've continued to watch films but not rate them, and of course I watched films before I discovered IMDb.

## Questions
Now, let's ask some questions of the data

### What do my ratings look like?
Zooming in on just the _ratings_, I'll use a [histogram](https://en.wikipedia.org/wiki/Histogram) to visualise the [frequency distribution](https://en.wikipedia.org/wiki/Frequency_distribution):


```python
sns.catplot(data=movies, kind="count",  x='Your Rating')
```


![png](/assets/img/output_12_1.png)


A lot more '1' ratings than I expected! I must have been hard to please.

We can also plot the [KDE](https://en.wikipedia.org/wiki/Kernel_density_estimation) for an estimate of the [probability density](https://en.wikipedia.org/wiki/Probability_density_function) (the y-axis here is 'density'):


```python
sns.set(rc={'figure.figsize':(8,6)})
sns.distplot(a=movies['Your Rating'], bins=range(1,11))
```


![png](/assets/img/output_14_1.png)


It look's like about half of my ratings were either a 6 or a 7 out of 10. How very diplomatic(!)

### How do my ratings compare to other IMDB users?
Let's calculate the [Spearman's rank correlation coefficient](https://en.wikipedia.org/wiki/Spearman%27s_rank_correlation_coefficient), as a measure of [Inter-rater reliability](https://en.wikipedia.org/wiki/Inter-rater_reliability). It's values range from -1 to 1 (fully opposed to identical). We're using Spearman here because our data (Ratings out of 10) is [ordinal](https://en.wikipedia.org/wiki/Ordinal_data).


```python
movies['Your Rating'].corr(movies['IMDb Rating'], method='spearman')
```




    0.4413428717210753



Intuitively, a weak positive correlation. 

Let's visualize this relationship. I'll use a catplot here to get around the problem of representing categorical data with a scatter plot


```python
sns.catplot(x='Your Rating', y='IMDb Rating', data=movies)
```



![png](/assets/img/output_19_1.png)


### What day did I tend to watch films? 


```python
# non-interesting date formatting
movies['Weekday'] = movies['Date Rated'].dt.day_name()
movies['WeekdayNumeric'] = movies['Date Rated'].dt.weekday
movies = movies.sort_values('WeekdayNumeric', inplace=False)
# Let's plot 
sns.catplot(x="Weekday", kind="count", data=movies, height=5, aspect=12/9)
```



![png](/assets/img/output_21_1.png)


The weekends of course make sense. But Monday is unexpected - TGIM?

### What year did I watch the most films?


```python
movies['YearWatched'] = movies['Date Rated'].dt.year
sns.catplot(x="YearWatched", kind="count", data=movies, height=6, aspect=12/9)
```


![png](/assets/img/output_24_1.png)


2008, clearly! That was about 1 film per weekday! It looks like the intensity waned over time: 2010, in contrast, was my first year in college - studying obviously took precedence!

### What is the *least* well-known film I've watched? (by number of ratings by other users)


```python
movies.sort_values('Num Votes').head(1)
```




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
      <th>Const</th>
      <th>Your Rating</th>
      <th>Date Rated</th>
      <th>Title</th>
      <th>URL</th>
      <th>Title Type</th>
      <th>IMDb Rating</th>
      <th>Runtime (mins)</th>
      <th>Year</th>
      <th>Genres</th>
      <th>Num Votes</th>
      <th>Release Date</th>
      <th>Directors</th>
      <th>Weekday</th>
      <th>WeekdayNumeric</th>
      <th>YearWatched</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>878</th>
      <td>tt0097073</td>
      <td>1</td>
      <td>2007-08-11</td>
      <td>Code Name Vengeance</td>
      <td>https://www.imdb.com/title/tt0097073/</td>
      <td>movie</td>
      <td>4.1</td>
      <td>96.0</td>
      <td>1987</td>
      <td>Action, Adventure, Drama, Thriller</td>
      <td>88.0</td>
      <td>1987-12-31</td>
      <td>David Winters</td>
      <td>Saturday</td>
      <td>5</td>
      <td>2007</td>
    </tr>
  </tbody>
</table>
</div>



Frankly, this movie looks terrible. Thankfully, I don't remember watching it (!)

### What is the *most* well-known film I've watched? (by number of ratings by other users)


```python
movies.sort_values('Num Votes', ascending=False).head(1)
```




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
      <th>Const</th>
      <th>Your Rating</th>
      <th>Date Rated</th>
      <th>Title</th>
      <th>URL</th>
      <th>Title Type</th>
      <th>IMDb Rating</th>
      <th>Runtime (mins)</th>
      <th>Year</th>
      <th>Genres</th>
      <th>Num Votes</th>
      <th>Release Date</th>
      <th>Directors</th>
      <th>Weekday</th>
      <th>WeekdayNumeric</th>
      <th>YearWatched</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>100</th>
      <td>tt0111161</td>
      <td>10</td>
      <td>2007-08-11</td>
      <td>The Shawshank Redemption</td>
      <td>https://www.imdb.com/title/tt0111161/</td>
      <td>movie</td>
      <td>9.3</td>
      <td>142.0</td>
      <td>1994</td>
      <td>Drama</td>
      <td>2301033.0</td>
      <td>1994-09-10</td>
      <td>Frank Darabont</td>
      <td>Saturday</td>
      <td>5</td>
      <td>2007</td>
    </tr>
  </tbody>
</table>
</div>



This one needs no introduction!

### What was the *least* liked film I've watched? (by ratings by other users)



```python
movies.sort_values('IMDb Rating').head(1)
```




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
      <th>Const</th>
      <th>Your Rating</th>
      <th>Date Rated</th>
      <th>Title</th>
      <th>URL</th>
      <th>Title Type</th>
      <th>IMDb Rating</th>
      <th>Runtime (mins)</th>
      <th>Year</th>
      <th>Genres</th>
      <th>Num Votes</th>
      <th>Release Date</th>
      <th>Directors</th>
      <th>Weekday</th>
      <th>WeekdayNumeric</th>
      <th>YearWatched</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>704</th>
      <td>tt0473024</td>
      <td>1</td>
      <td>2007-08-11</td>
      <td>Crossover</td>
      <td>https://www.imdb.com/title/tt0473024/</td>
      <td>movie</td>
      <td>2.1</td>
      <td>95.0</td>
      <td>2006</td>
      <td>Action, Sport</td>
      <td>8910.0</td>
      <td>2006-07-22</td>
      <td>Preston A. Whitmore II</td>
      <td>Saturday</td>
      <td>5</td>
      <td>2007</td>
    </tr>
  </tbody>
</table>
</div>



Ah, this doesn't look *that* bad

### What was the *most* liked film I've watched? (by ratings by other users)


```python
movies.sort_values('IMDb Rating', ascending=False).head(1)
```




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
      <th>Const</th>
      <th>Your Rating</th>
      <th>Date Rated</th>
      <th>Title</th>
      <th>URL</th>
      <th>Title Type</th>
      <th>IMDb Rating</th>
      <th>Runtime (mins)</th>
      <th>Year</th>
      <th>Genres</th>
      <th>Num Votes</th>
      <th>Release Date</th>
      <th>Directors</th>
      <th>Weekday</th>
      <th>WeekdayNumeric</th>
      <th>YearWatched</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>100</th>
      <td>tt0111161</td>
      <td>10</td>
      <td>2007-08-11</td>
      <td>The Shawshank Redemption</td>
      <td>https://www.imdb.com/title/tt0111161/</td>
      <td>movie</td>
      <td>9.3</td>
      <td>142.0</td>
      <td>1994</td>
      <td>Drama</td>
      <td>2301033.0</td>
      <td>1994-09-10</td>
      <td>Frank Darabont</td>
      <td>Saturday</td>
      <td>5</td>
      <td>2007</td>
    </tr>
  </tbody>
</table>
</div>



A certified classic.

### Biggest disparity in my vote vs IMDB? 
- What was the popularly-least-liked film I've enjoyed? 
- And the popularly-most-liked film I didn't?


```python
movies['Vote Disparity - IMDb Liked more'] = movies['IMDb Rating'] - movies['Your Rating']
movies.sort_values(['Vote Disparity - IMDb Liked more', 'IMDb Rating', 'Num Votes'], ascending=False).head(1)
```




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
      <th>Const</th>
      <th>Your Rating</th>
      <th>Date Rated</th>
      <th>Title</th>
      <th>URL</th>
      <th>Title Type</th>
      <th>IMDb Rating</th>
      <th>Runtime (mins)</th>
      <th>Year</th>
      <th>Genres</th>
      <th>Num Votes</th>
      <th>Release Date</th>
      <th>Directors</th>
      <th>Weekday</th>
      <th>WeekdayNumeric</th>
      <th>YearWatched</th>
      <th>Vote Disparity - IMDb Liked more</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>95</th>
      <td>tt0110912</td>
      <td>1</td>
      <td>2008-08-19</td>
      <td>Pulp Fiction</td>
      <td>https://www.imdb.com/title/tt0110912/</td>
      <td>movie</td>
      <td>8.9</td>
      <td>154.0</td>
      <td>1994</td>
      <td>Crime, Drama</td>
      <td>1796406.0</td>
      <td>1994-05-21</td>
      <td>Quentin Tarantino</td>
      <td>Tuesday</td>
      <td>1</td>
      <td>2008</td>
      <td>7.9</td>
    </tr>
  </tbody>
</table>
</div>



Yeah, Pulp Fiction is overrated.


```python
movies['Vote Disparity - I Liked more'] = movies['Your Rating'] - movies['IMDb Rating']
movies.sort_values(['Vote Disparity - I Liked more', 'IMDb Rating', 'Num Votes'], ascending=False).head(1)
```




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
      <th>Const</th>
      <th>Your Rating</th>
      <th>Date Rated</th>
      <th>Title</th>
      <th>URL</th>
      <th>Title Type</th>
      <th>IMDb Rating</th>
      <th>Runtime (mins)</th>
      <th>Year</th>
      <th>Genres</th>
      <th>Num Votes</th>
      <th>Release Date</th>
      <th>Directors</th>
      <th>Weekday</th>
      <th>WeekdayNumeric</th>
      <th>YearWatched</th>
      <th>Vote Disparity - IMDb Liked more</th>
      <th>Vote Disparity - I Liked more</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>469</th>
      <td>tt0250310</td>
      <td>7</td>
      <td>2009-02-19</td>
      <td>Corky Romano</td>
      <td>https://www.imdb.com/title/tt0250310/</td>
      <td>movie</td>
      <td>4.7</td>
      <td>86.0</td>
      <td>2001</td>
      <td>Comedy, Crime</td>
      <td>12425.0</td>
      <td>2001-10-12</td>
      <td>Rob Pritts</td>
      <td>Thursday</td>
      <td>3</td>
      <td>2009</td>
      <td>-2.3</td>
      <td>2.3</td>
    </tr>
  </tbody>
</table>
</div>



Corky Romano is a masterpiece!

# General patterns
Finally, let's visualize some general patterns

### Film durations


```python
sns.distplot(movies['Runtime (mins)'])
```



![png](/assets/img/output_45_1.png)


### Film Release Years


```python
sns.catplot(x="Year", kind="count", data=movies, height=10, aspect=21/9)
```



![png](/assets/img/output_47_1.png)


### Film genres


```python
genres = movies['Genres'].astype('str').tolist()
genres = list(map(lambda x: x.split(','), genres))
genres = sum(list(genres), [])
genres = list(map(lambda x: x.strip(), genres))
genres = list(map(lambda x: [x], genres))
genres = pd.DataFrame(genres, columns=['Genres'])
sns.catplot(x="Genres", kind="count", data=genres, height=9, aspect=21/9, order = genres['Genres'].value_counts().index)
```



![png](/assets/img/output_49_1.png)


### Directors


```python
directors = movies['Directors'].astype('str').tolist()
directors = list(map(lambda x: x.split(','), directors))
directors = sum(list(directors), [])
directors = list(map(lambda x: x.strip(), directors))
directors = list(map(lambda x: [x], directors))
directors = pd.DataFrame(directors, columns=['Directors'])
directors = directors.groupby(['Directors']).size()
directors = directors.reset_index(name='size')
directors = directors.sort_values(['size'], ascending=False)
directors.head(10)
```




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
      <th>Directors</th>
      <th>size</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>514</th>
      <td>Steven Spielberg</td>
      <td>15</td>
    </tr>
    <tr>
      <th>453</th>
      <td>Robert Zemeckis</td>
      <td>10</td>
    </tr>
    <tr>
      <th>111</th>
      <td>David Fincher</td>
      <td>8</td>
    </tr>
    <tr>
      <th>345</th>
      <td>Martin Scorsese</td>
      <td>8</td>
    </tr>
    <tr>
      <th>415</th>
      <td>Peter Jackson</td>
      <td>7</td>
    </tr>
    <tr>
      <th>423</th>
      <td>Quentin Tarantino</td>
      <td>7</td>
    </tr>
    <tr>
      <th>256</th>
      <td>John Hughes</td>
      <td>7</td>
    </tr>
    <tr>
      <th>85</th>
      <td>Christopher Nolan</td>
      <td>6</td>
    </tr>
    <tr>
      <th>464</th>
      <td>Ron Howard</td>
      <td>6</td>
    </tr>
    <tr>
      <th>155</th>
      <td>Ethan Coen</td>
      <td>6</td>
    </tr>
  </tbody>
</table>
</div>



### What's the perfect film for me?
Based on frequent attributes: It looks like that would be a Drama or Comedy, directed by Spielberg, made in 2011, that is about 100 minutes long! 'War Horse', anybody?

If we base it on rating, by restricting this to just films I rated >= 8: A Drama, directed by Spielberg, made in 1999, that's 120 minutes long. Perhaps, 'Saving Private Ryan'?

This however would be an interesting small ML project: to create a model to predict a numeric rating for a given film, based on my previous ratings.
