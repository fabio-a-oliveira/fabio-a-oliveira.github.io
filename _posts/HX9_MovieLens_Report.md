

------------------------------------------------------------------------

Introduction
============

This report describes the construction of a statistical model that
predicts movie ratings, based on the *MovieLens 10M* dataset provided by
the [GroupLens](https://grouplens.org/) research lab at the University
of Minnesota. This version of the dataset has been released in 2009 and
contains over 10 million ratings given by around 70 thousand individual
users to over 10 thousand different movies, with each entry containing
identification numbers for the user and movie, as well as the movie
title and release year, its genres and a timestamp identifying when the
rating was given. The dataset has been used regularly by the Data
Science community in teaching and in the development of machine learning
models.

Before attempting to construct any statistical model, some basic data
wrangling is performed, followed by an exploratory data analysis, in
order to identify trends in the data that may be later used to construct
predictions.

An additive approach has been used to construct the final model, in the
sense that a number of increasingly complex models has been created so
that, at each step, an additional term accounts for patterns identified
in the residuals from the previous step. With this approach, the
variable under evaluation at each step can be analyzed individually, and
the correlation between different predictors is not a point of concern.

For the purposes of this project, the full 10M dataset has been
partitioned into three sets:

1.  A *train* set, from where patterns are identified and quantified in
    order to build the statistical model;
2.  A *test* set, where different parameters are tested and tuned in
    order to find optimal performance and
3.  A *validation* set, which will not be used for training or tuning of
    parameters, and where the models will be evaluated.

The ultimate goal of the project is to create a model capable of
predicting ratings in the *validation* set as precisely as practical.
The *Root Mean Squared Error* (RMSE) of the predictions compared to the
actual ratings is reported as a performance metric. The RMSE obtained
after application of the complete model to the *validation* set was
0.843.

A Shiny Web App with a minimalist version of the final model has been
created for illustrative purposes and can be accessed at
<a href="https://fabio-a-oliveira.shinyapps.io/MovieRecommenderApp/" class="uri">https://fabio-a-oliveira.shinyapps.io/MovieRecommenderApp/</a>.

The GitHub page for this project is
<a href="https://github.com/fabio-a-oliveira/movie-recommendations" class="uri">https://github.com/fabio-a-oliveira/movie-recommendations</a>.

This analysis is part of the capstone project for the Data Science
Professional Certificate offered by HarvardX and hosted on edX. More
information can be found at
<a href="https://www.edx.org/professional-certificate/harvardx-data-science" class="uri">https://www.edx.org/professional-certificate/harvardx-data-science</a>.

------------------------------------------------------------------------

Methods/analyses
================

The project has been developed according to a sequence of steps:

1.  Data preparation: during this step, the MovieLens 10M dataset is
    downloaded programmaticaly from the GroupLens website and
    partitioned into the *train*, *test*, and *validation* sets. Care is
    taken to ensure that every user and movie are represented in the
    *train* set (so that the model is not required to make predictions
    for users and movies to which it has not been exposed during
    training). Separate data frames are created with statistics for each
    individual users, movies and movie genres, so that information
    gathered during the following steps can be store and accessed
    easily. Empty data frames are also created to store functions to
    apply each of the models, its predictions for each of the datasets
    and the results.
2.  Exploratory data analysis: during this step, the *train* set is
    explored to identify relevant features, how the different variables
    are related, and which characteristics can potentially be used
    during the modeling step in order to predict ratings. This step is
    also used to get familiarity with the data.
3.  Modeling: during this step, increasingly complex models are
    developed, trained and tested to account for the influence of the
    factors identified during the exploratory data analysis on the
    ratings given to movies by each user. These models are created so
    that the simplest connections are accounted for sooner (how does a
    single predictor relate to the ratings?), followed by more complex
    phenomenons (how do combinations of predictors relate to the
    ratings?). The final model includes an *Item-Based Collaborative
    Filter* (IBCF), in which the predictions for how each individual
    user rates each movie takes into consideration how the user has
    previously rated similar movies.

Each of these steps is explained in more details in the corresponding
section of this report.

Additionally, due to the size of the original dataset used in this
analysis, significant challenges related to computing performance were
tackled. Constant care was taken to avoid using unnecessary memory.
Nevertheless, certain sections of the code required a command to
increase the amount of memory available to R with the
`memory.limit(size = 16000)` command. During the creation of the model
with the IBCF in particular, the calculations required the use of the
`Matrix` package for dealing with sparse matrices and the implementation
of custom functions to identify the similarity between different movies
in a memory-efficient manner.

The R code for this project is divided into two different files. All of
the data preparation and the modeling are done in the
`HX9_MovieLens_Main.R` script. This script prepares the data and does
all of the calculations necessary for the development of the models,
creates functions to make predictions according to each model, and
calculates the performance for each model. It then saves the resulting
objects into files that are loaded by the `HX9_MovieLens_Report.RMD`
script (the R Markdown script that generates this report) to create all
the tables and graphics in this report.

------------------------------------------------------------------------

Data preparation
----------------

A series of adjustments to the original *MovieLens 10M* dataset is
necessary before we begin analysis and modeling. These adjustments are
described below:

#### 1. The original data is partitioned into the *train*, *test*, and *validation* datasets.

As a first step, the original data is partitioned into the *edx* and
*validation* data frames at a 0.9/0.1 proportion. The *edx* object is
then partitioned into the *train* and *test* sets at a 0.8/0.2
proportion.

The table below shows how the full set of entries from the *MovieLens
10M* dataset is distributed among the three different sets used in this
project:

<table>
<caption>Entries in the different sets</caption>
<thead>
<tr class="header">
<th style="text-align: left;">Object</th>
<th style="text-align: left;">Number of Ratings (% of total)</th>
<th style="text-align: left;">Number of Users (% of total)</th>
<th style="text-align: left;">Number of Movies (% of total)</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">train</td>
<td style="text-align: left;">7200088 (72%)</td>
<td style="text-align: left;">69878 (100%)</td>
<td style="text-align: left;">10677 (100%)</td>
</tr>
<tr class="even">
<td style="text-align: left;">test</td>
<td style="text-align: left;">1799967 (18%)</td>
<td style="text-align: left;">69754 (100%)</td>
<td style="text-align: left;">10153 (95%)</td>
</tr>
<tr class="odd">
<td style="text-align: left;">validation</td>
<td style="text-align: left;">999999 (10%)</td>
<td style="text-align: left;">68534 (98%)</td>
<td style="text-align: left;">9809 (92%)</td>
</tr>
</tbody>
</table>

It is relevant to notice that the *train* set contains entries for all
the individual users and movies, so the different models are not
required to predict ratings for users and movies with which they have
had not previous training.

A glimpse at a sample of these objects reveals what information is
available and what further adjustments are necessary:

    ## # A tibble: 15 x 6
    ##    userId movieId rating  timestamp title                         genres          
    ##     <dbl>   <dbl>  <dbl>      <dbl> <chr>                         <chr>           
    ##  1  29658    4054    3   1144040512 Save the Last Dance (2001)    Drama|Romance   
    ##  2  71180     524    3    833565936 Rudy (1993)                   Drama           
    ##  3  24263    4957    4.5 1186765094 Sudden Impact (1983)          Crime|Drama     
    ##  4  11226     282    3   1112374700 Nell (1994)                   Drama           
    ##  5  27269     416    3    974841556 Bad Girls (1994)              Western         
    ##  6  25730    1035    5   1142620501 Sound of Music, The (1965)    Musical|Romance 
    ##  7  23159    1060    4    944061984 Swingers (1996)               Comedy|Drama    
    ##  8  46748    1021    3   1107138618 Angels in the Outfield (1994) Children|Comedy 
    ##  9  18129     150    3.5 1113561555 Apollo 13 (1995)              Adventure|Drama 
    ## 10  20866    3035    4   1005867535 Mister Roberts (1955)         Comedy|Drama|War
    ## 11  49437    2841    5    994907991 Stir of Echoes (1999)         Thriller        
    ## 12  66280    2707    3    980280673 Arlington Road (1999)         Thriller        
    ## 13  55941    1387    4    855326789 Jaws (1975)                   Action|Horror   
    ## 14  13582    2395    5   1112471869 Rushmore (1998)               Comedy|Drama    
    ## 15  53826    1982    3   1044477057 Halloween (1978)              Horror

After a quick glimpse at the dataset, we see that the `timestamp` column
is formatted in a manner that is difficult to read. The `title` column
contains both the movie title and its year of release. Additionally, the
`genres` column contains all the genres associated with the movie,
separated by a “|” character. Some basic data wrangling is necessary to
have this information in usable formats. However, in order to preserve
these objects as close as possible to the original data, any
manipulation and formatting of the data is done in the objects created
in the next step.

The choice for doing training and testing with a single split of the
*edx* data (as opposed to performing k-fold cross-validation) has been
motivated by simplicity and saving time in code execution. With this
approach, statistics can be drawn directly from the *train* set and
performance for different parameters can be tested directly on the
*test* set, without the need to average results over multiple folds. The
fact that there are millions of observations in both sets guarantees
that the outcomes are robust to random variability and mitigate the risk
of overtraining.

#### 2. Objects are created to hold data pertinent to individual entries on the datasets

Several R objects are created to hold relevant information that will be
used for analysis, modeling and holding results during the project.
Basic summary statistics and some necessary data wrangling identified in
the analysis of the *train*, *test*, and *validation* sets are also
performed and saved to the new objects.

-   The `users` object contains a summary with the average rating and
    number of ratings for each individual user in the form of a data
    frame;

<!-- -->

    ##   userId average rating number of ratings
    ## 1  36576       3.475410                61
    ## 2  57596       3.880435                92
    ## 3  39938       4.367188                64
    ## 4  24653       3.746032               126
    ## 5  37715       3.308333               120
    ## 6    169       3.611111                18

-   The `movies` object contains a summary with the average rating and
    number of ratings for each individual movie in the form of a data
    frame;

<!-- -->

    ##   movieId             title year          genres average rating number of ratings
    ## 1    8947       Grudge, The 2004 Horror|Thriller       2.761675               621
    ## 2    2808 Universal Soldier 1992   Action|Sci-Fi       2.538986              1539
    ## 3   34193          Conflict 1945 Drama|Film-Noir       3.400000                 5
    ## 4    2967     Bad Seed, The 1956  Drama|Thriller       3.697479               357
    ## 5   48262       Bridge, The 2006     Documentary       3.653846                39
    ## 6    3288       Cotton Mary 1999           Drama       2.875000                12

-   The `genres` object contains both a data frame with each individual
    movie genre associated with each movie (in “long” format) and a data
    frame with columns for each movie genre indicating whether each
    movie is associated with each particular movie (in “wide” format),
    in the form of a list;

<!-- -->

    ##                          genres     genre
    ## 1         Action|Crime|Thriller    Action
    ## 2         Action|Crime|Thriller     Crime
    ## 3         Action|Crime|Thriller  Thriller
    ## 4  Action|Drama|Sci-Fi|Thriller    Action
    ## 5  Action|Drama|Sci-Fi|Thriller     Drama
    ## 6  Action|Drama|Sci-Fi|Thriller    Sci-Fi
    ## 7  Action|Drama|Sci-Fi|Thriller  Thriller
    ## 8       Action|Adventure|Sci-Fi    Action
    ## 9       Action|Adventure|Sci-Fi Adventure
    ## 10      Action|Adventure|Sci-Fi    Sci-Fi

    ##                                        genres Action Crime Thriller Drama Sci-Fi
    ## 1                       Action|Crime|Thriller   TRUE  TRUE     TRUE FALSE  FALSE
    ## 2                Action|Drama|Sci-Fi|Thriller   TRUE FALSE     TRUE  TRUE   TRUE
    ## 3                     Action|Adventure|Sci-Fi   TRUE FALSE    FALSE FALSE   TRUE
    ## 4               Action|Adventure|Drama|Sci-Fi   TRUE FALSE    FALSE  TRUE   TRUE
    ## 5                     Children|Comedy|Fantasy  FALSE FALSE    FALSE FALSE  FALSE
    ## 6                    Comedy|Drama|Romance|War  FALSE FALSE    FALSE  TRUE  FALSE
    ## 7                  Adventure|Children|Romance  FALSE FALSE    FALSE FALSE  FALSE
    ## 8  Adventure|Animation|Children|Drama|Musical  FALSE FALSE    FALSE  TRUE  FALSE
    ## 9                               Action|Comedy   TRUE FALSE    FALSE FALSE  FALSE
    ## 10                    Action|Romance|Thriller   TRUE FALSE     TRUE FALSE  FALSE
    ##    Adventure Children Comedy Fantasy Romance   War Animation Musical Western Mystery
    ## 1      FALSE    FALSE  FALSE   FALSE   FALSE FALSE     FALSE   FALSE   FALSE   FALSE
    ## 2      FALSE    FALSE  FALSE   FALSE   FALSE FALSE     FALSE   FALSE   FALSE   FALSE
    ## 3       TRUE    FALSE  FALSE   FALSE   FALSE FALSE     FALSE   FALSE   FALSE   FALSE
    ## 4       TRUE    FALSE  FALSE   FALSE   FALSE FALSE     FALSE   FALSE   FALSE   FALSE
    ## 5      FALSE     TRUE   TRUE    TRUE   FALSE FALSE     FALSE   FALSE   FALSE   FALSE
    ## 6      FALSE    FALSE   TRUE   FALSE    TRUE  TRUE     FALSE   FALSE   FALSE   FALSE
    ## 7       TRUE     TRUE  FALSE   FALSE    TRUE FALSE     FALSE   FALSE   FALSE   FALSE
    ## 8       TRUE     TRUE  FALSE   FALSE   FALSE FALSE      TRUE    TRUE   FALSE   FALSE
    ## 9      FALSE    FALSE   TRUE   FALSE   FALSE FALSE     FALSE   FALSE   FALSE   FALSE
    ## 10     FALSE    FALSE  FALSE   FALSE    TRUE FALSE     FALSE   FALSE   FALSE   FALSE
    ##    Horror Film-Noir Documentary  IMAX (no genres listed)
    ## 1   FALSE     FALSE       FALSE FALSE              FALSE
    ## 2   FALSE     FALSE       FALSE FALSE              FALSE
    ## 3   FALSE     FALSE       FALSE FALSE              FALSE
    ## 4   FALSE     FALSE       FALSE FALSE              FALSE
    ## 5   FALSE     FALSE       FALSE FALSE              FALSE
    ## 6   FALSE     FALSE       FALSE FALSE              FALSE
    ## 7   FALSE     FALSE       FALSE FALSE              FALSE
    ## 8   FALSE     FALSE       FALSE FALSE              FALSE
    ## 9   FALSE     FALSE       FALSE FALSE              FALSE
    ## 10  FALSE     FALSE       FALSE FALSE              FALSE

-   The `models` object is an empty list of functions, which will be
    filled during the modeling phase with functions calculating
    predictions for each different model;
-   The `parameters` object is an empty list, which will contain
    parameters required for the application of the models calculated
    during the modeling phase;
-   The `results` object is an empty list, which will be filled with
    results acchieved by each of the models and
-   The `predictions` object is a list containing empty data frames that
    will be filled with the RMSE results obtained by each model on the
    *train*, *test*, and *validation* sets.

During the creation of the models, additional information pertinent to
each entry of these data frames is included.

#### 3. Creation of functions

In order to facilitate calculations during the subsequent phases, some R
functions are created to perform tasks that are repeated multiple times:

-   The `RMSE()` function calculates the Root Mean Squared Error between
    two supplied vectors;
-   The `limitRange()` function introduces a saturation to a supplied
    vector, ensuring that each entry is within the interval determined
    by the `min` and `max` arguments and
-   The `sparse.colCenter()`, `sparse.colMeans()`, `sparse.colSums()`
    and `sparse.correlationCommonRows()` functions perform numerous
    sparse matrix manipulation tasks in a memory-conscious manner and
    will be described in more details in the section dedicated to the
    model with collaborative filtering.

------------------------------------------------------------------------

Exploratory Data Analysis
-------------------------

Before we tackle the prediction problem, we begin by doing a thorough
exploratory data analysis, with the goal of identifying trends and
features of the data that might be useful in understanding it and
possibly in constructing a prediction algorithm.

Five aspects will be explored in details in this section:

1.  Distribution of ratings
2.  Individual movies
3.  Individual users
4.  Movie genres
5.  Movie year of release and date of rating

At this stage, no attempt will be made to predict the actual ratings,
but trends identified during the exploratory data analysis will be later
used as criteria to build prediction models during the modeling section.

### Global distribution of ratings

We begin by looking at how the ratings are distributed in the *train*
set.

<img src="HX9_MovieLens_Report_files/figure-markdown_strict/histogram of ratings-1.png" width="70%" height="70%" style="display: block; margin: auto;" />

We see that the ratings range from 0.5 to 5.0 in increments of 0.5,
which is rather typical of a rating system based on number of stars. The
histogram also shows that the most common ratings are 4 and 3, and that
most integer ratings are more common than non-integer ratings.

<img src="HX9_MovieLens_Report_files/figure-markdown_strict/boxplot of ratings from sample-1.png" width="70%" height="70%" style="display: block; margin: auto;" />

This boxplot of the ratings for 30 randomly selected movies shows that
the Inter-Quartile Range (IQR) is typically the interval of +/- 0.5
points around the median. However, the majority of movies do have
ratings that cover most of the 0.5 - 5 points scale.

### Analysis of individual movies

We begin the exploration of the characteristics of individual movies by
looking at the table of the most popular movies, i.e. the ones with the
most number of ratings.

<table>
<caption>Top 10 Popular Movies</caption>
<thead>
<tr class="header">
<th style="text-align: right;">movieId</th>
<th style="text-align: left;">Title</th>
<th style="text-align: right;">Average Rating</th>
<th style="text-align: right;">Number of Ratings</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: right;">296</td>
<td style="text-align: left;">Pulp Fiction</td>
<td style="text-align: right;">4.16</td>
<td style="text-align: right;">25137</td>
</tr>
<tr class="even">
<td style="text-align: right;">356</td>
<td style="text-align: left;">Forrest Gump</td>
<td style="text-align: right;">4.01</td>
<td style="text-align: right;">24943</td>
</tr>
<tr class="odd">
<td style="text-align: right;">593</td>
<td style="text-align: left;">Silence of the Lambs, The</td>
<td style="text-align: right;">4.20</td>
<td style="text-align: right;">24432</td>
</tr>
<tr class="even">
<td style="text-align: right;">480</td>
<td style="text-align: left;">Jurassic Park</td>
<td style="text-align: right;">3.66</td>
<td style="text-align: right;">23492</td>
</tr>
<tr class="odd">
<td style="text-align: right;">318</td>
<td style="text-align: left;">Shawshank Redemption, The</td>
<td style="text-align: right;">4.46</td>
<td style="text-align: right;">22406</td>
</tr>
<tr class="even">
<td style="text-align: right;">110</td>
<td style="text-align: left;">Braveheart</td>
<td style="text-align: right;">4.08</td>
<td style="text-align: right;">20881</td>
</tr>
<tr class="odd">
<td style="text-align: right;">457</td>
<td style="text-align: left;">Fugitive, The</td>
<td style="text-align: right;">4.01</td>
<td style="text-align: right;">20859</td>
</tr>
<tr class="even">
<td style="text-align: right;">589</td>
<td style="text-align: left;">Terminator 2: Judgment Day</td>
<td style="text-align: right;">3.93</td>
<td style="text-align: right;">20752</td>
</tr>
<tr class="odd">
<td style="text-align: right;">260</td>
<td style="text-align: left;">Star Wars: Episode IV - A New Hope (a.k.a. Star Wars)</td>
<td style="text-align: right;">4.22</td>
<td style="text-align: right;">20591</td>
</tr>
<tr class="even">
<td style="text-align: right;">150</td>
<td style="text-align: left;">Apollo 13</td>
<td style="text-align: right;">3.89</td>
<td style="text-align: right;">19494</td>
</tr>
</tbody>
</table>

It is also interesting to see which movies have the highest average
ratings.

<table>
<caption>Top 10 Best Rated Movies</caption>
<thead>
<tr class="header">
<th style="text-align: right;">movieId</th>
<th style="text-align: left;">Title</th>
<th style="text-align: right;">Average Rating</th>
<th style="text-align: right;">Number of Ratings</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: right;">3226</td>
<td style="text-align: left;">Hellhounds on My Trail</td>
<td style="text-align: right;">5.00</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: right;">25789</td>
<td style="text-align: left;">Shanghai Express</td>
<td style="text-align: right;">5.00</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: right;">33264</td>
<td style="text-align: left;">Satan’s Tango (SÃ¡tÃ¡ntangÃ³)</td>
<td style="text-align: right;">5.00</td>
<td style="text-align: right;">2</td>
</tr>
<tr class="even">
<td style="text-align: right;">42783</td>
<td style="text-align: left;">Shadows of Forgotten Ancestors</td>
<td style="text-align: right;">5.00</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: right;">51209</td>
<td style="text-align: left;">Fighting Elegy (Kenka erejii)</td>
<td style="text-align: right;">5.00</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: right;">53355</td>
<td style="text-align: left;">Sun Alley (Sonnenallee)</td>
<td style="text-align: right;">5.00</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: right;">63772</td>
<td style="text-align: left;">Bullfighter and the Lady</td>
<td style="text-align: right;">5.00</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: right;">64275</td>
<td style="text-align: left;">Blue Light, The (Das Blaue Licht)</td>
<td style="text-align: right;">5.00</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: right;">26048</td>
<td style="text-align: left;">Human Condition II, The (Ningen no joken II)</td>
<td style="text-align: right;">4.83</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="even">
<td style="text-align: right;">5194</td>
<td style="text-align: left;">Who’s Singin’ Over There? (a.k.a. Who Sings Over There) (Ko to tamo peva)</td>
<td style="text-align: right;">4.75</td>
<td style="text-align: right;">4</td>
</tr>
<tr class="odd">
<td style="text-align: right;">25975</td>
<td style="text-align: left;">Life of Oharu, The (Saikaku ichidai onna)</td>
<td style="text-align: right;">4.75</td>
<td style="text-align: right;">2</td>
</tr>
<tr class="even">
<td style="text-align: right;">65001</td>
<td style="text-align: left;">Constantine’s Sword</td>
<td style="text-align: right;">4.75</td>
<td style="text-align: right;">2</td>
</tr>
</tbody>
</table>

From this previous table, we notice that the movies at the top of the
list all have very low number of ratings. That indicates that they high
averages are most likely a product of random variability. When we create
a list of the top 10 movies arranged by their ratings, but first filter
out movies with less then 100 individual ratings, we get a more
reasonable result:

<table>
<caption>Top 10 Best Rated Movies (with over 100 ratings)</caption>
<thead>
<tr class="header">
<th style="text-align: right;">movieId</th>
<th style="text-align: left;">Title</th>
<th style="text-align: right;">Average Rating</th>
<th style="text-align: right;">Number of Ratings</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: right;">318</td>
<td style="text-align: left;">Shawshank Redemption, The</td>
<td style="text-align: right;">4.46</td>
<td style="text-align: right;">22406</td>
</tr>
<tr class="even">
<td style="text-align: right;">858</td>
<td style="text-align: left;">Godfather, The</td>
<td style="text-align: right;">4.41</td>
<td style="text-align: right;">14166</td>
</tr>
<tr class="odd">
<td style="text-align: right;">527</td>
<td style="text-align: left;">Schindler’s List</td>
<td style="text-align: right;">4.37</td>
<td style="text-align: right;">18573</td>
</tr>
<tr class="even">
<td style="text-align: right;">50</td>
<td style="text-align: left;">Usual Suspects, The</td>
<td style="text-align: right;">4.37</td>
<td style="text-align: right;">17357</td>
</tr>
<tr class="odd">
<td style="text-align: right;">912</td>
<td style="text-align: left;">Casablanca</td>
<td style="text-align: right;">4.32</td>
<td style="text-align: right;">8982</td>
</tr>
<tr class="even">
<td style="text-align: right;">3435</td>
<td style="text-align: left;">Double Indemnity</td>
<td style="text-align: right;">4.32</td>
<td style="text-align: right;">1722</td>
</tr>
<tr class="odd">
<td style="text-align: right;">922</td>
<td style="text-align: left;">Sunset Blvd. (a.k.a. Sunset Boulevard)</td>
<td style="text-align: right;">4.32</td>
<td style="text-align: right;">2303</td>
</tr>
<tr class="even">
<td style="text-align: right;">1178</td>
<td style="text-align: left;">Paths of Glory</td>
<td style="text-align: right;">4.32</td>
<td style="text-align: right;">1236</td>
</tr>
<tr class="odd">
<td style="text-align: right;">904</td>
<td style="text-align: left;">Rear Window</td>
<td style="text-align: right;">4.32</td>
<td style="text-align: right;">6364</td>
</tr>
<tr class="even">
<td style="text-align: right;">1221</td>
<td style="text-align: left;">Godfather: Part II, The</td>
<td style="text-align: right;">4.31</td>
<td style="text-align: right;">9451</td>
</tr>
</tbody>
</table>

A histogram of the average movie ratings shows that movie averages are
most common in the vicinity of 3.5. However, the overall average is
actually around 3.19, which indicates that the data is skewed to the
right. This makes sense, since the upper limit of the interval of
possible ratings is closer to the average, so movies better rated than
the average are more concentrated.

<img src="HX9_MovieLens_Report_files/figure-markdown_strict/histogram of ratings per movies-1.png" width="70%" height="70%" style="display: block; margin: auto;" />

The following histogram shows the distribution of the total number of
ratings. We see that there is a large concentration in the low numbers,
with an average of 674 reviews per movie, while some have over 20
thousand ratings.

This concentration of ratings into few movies has a dramatic effect on
the average. The actual median number of ratings is much lower, at 98,
with the IQR of 24 to 453 reviews.

<img src="HX9_MovieLens_Report_files/figure-markdown_strict/histograms of number of ratings per movie-1.png" width="70%" height="70%" style="display: block; margin: auto;" />

A scatter plot of the average rating per number of ratings for the
entire `movies` object illustrates some of these points. We see in the
figure below that the top 10 best rated movies are all concentrate into
a single region and are obviously outliers. There is a clear positive
correlation between movie popularity and rating, and the most popular
movies are indeed well rated. However, the top 10 best rated movies with
over 100 ratings are somewhat evenly distributed in the range between
roughly 2 and 22 thousand ratings.

<img src="HX9_MovieLens_Report_files/figure-markdown_strict/movie ratings vs popularity-1.png" width="70%" height="70%" style="display: block; margin: auto;" />

Finally, we look at the cumulative proportion of ratings, with all
movies order from most to least popular. The figure below shows that,
from a total of 10677 movies, only around 500 are necessary to account
for half the individual ratings, while around 2800 movies account for
90% of all the ratings.

<img src="HX9_MovieLens_Report_files/figure-markdown_strict/cumulative number of ratings-1.png" width="70%" height="70%" style="display: block; margin: auto;" />

From the analysis of individual movies we see that there is a clear
connection between popularity and how a movie is rated on average. We
also see that there is considerable random noise in the set of movies
that are more obscure, with very few ratings. Finally, we see that the
ratings are very concentrated in a reduced set of very popular movies.

### Analysis of individual users

We now turn our attention to the behavior of individual users. First, we
look at the distribution of user’s individual mean ratings.

<img src="HX9_MovieLens_Report_files/figure-markdown_strict/histogram of average ratings per user-1.png" width="70%" height="70%" style="display: block; margin: auto;" />

The figure shows that average ratings are distributed according to a
roughly bell-shaped curve, centered at 3.61. The curve is slightly
skewed to the right, which agrees with the fact that the average is
closer to the top limit of 5 than the bottom limit of 0.5.

The frequency at which users rate is depicted in the histogram below:

<img src="HX9_MovieLens_Report_files/figure-markdown_strict/histogram of number of ratings per user-1.png" width="70%" height="70%" style="display: block; margin: auto;" />

We see from the histogram that the vast majority of users have rated
between 0 and 250 movies, with the average at 103. However, some users
have rated considerably more often than that, with 300 users having
rated more than one thousand movies, and 2 having rated over five
thousand.

Some basic statistics for the number of ratings per user are shown in
the table below:

<table>
<caption>Mean and quantiles for individual user’s number of ratings</caption>
<thead>
<tr class="header">
<th style="text-align: right;">0% (min.)</th>
<th style="text-align: right;">25% (1st qu.)</th>
<th style="text-align: right;">50% (median)</th>
<th style="text-align: right;">Mean</th>
<th style="text-align: right;">75% (3rd qu.)</th>
<th style="text-align: right;">100% (max.)</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: right;">7</td>
<td style="text-align: right;">25</td>
<td style="text-align: right;">50</td>
<td style="text-align: right;">103.038</td>
<td style="text-align: right;">113</td>
<td style="text-align: right;">5332</td>
</tr>
</tbody>
</table>

Next, we investigate whether users rate consistently across different
movies. The following plot shows the distribution of each user’s rating
standard deviation. Users are grouped according to their mean rating.

<img src="HX9_MovieLens_Report_files/figure-markdown_strict/standard deviation of users ratings-1.png" width="70%" height="70%" style="display: block; margin: auto;" />

The figure shows that users tend to be consistent, with the mean
standard deviation at 0.958. As one would expect, this standard
deviation is lower for users with average ratings closer to the extremes
of the scale.

Finally, we look at the cumulative percentage of ratings. The figure
below shows this cumulative percentage, with users ordered according to
their total number of ratings.

<img src="HX9_MovieLens_Report_files/figure-markdown_strict/cumulative ratings per user-1.png" width="70%" height="70%" style="display: block; margin: auto;" />

The curve shows that, while there is a concentration of the total number
of ratings in the most active users, this concentration is not as
pronounced as was observed for the most popular movies. The table below
compares how much of the full sets of movies and users are required to
account for 50% and 90% of the total ratings.

<table>
<caption>Concentration of total ratings in most popular movies and most active users</caption>
<thead>
<tr class="header">
<th style="text-align: center;">Variable</th>
<th style="text-align: center;">Total</th>
<th style="text-align: center;">Percentage to account for 50% of all ratings</th>
<th style="text-align: center;">Percentage to account for 90% of all ratings</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;">movies</td>
<td style="text-align: center;">10677</td>
<td style="text-align: center;">4.7%</td>
<td style="text-align: center;">26%</td>
</tr>
<tr class="even">
<td style="text-align: center;">users</td>
<td style="text-align: center;">69878</td>
<td style="text-align: center;">12.5%</td>
<td style="text-align: center;">58%</td>
</tr>
</tbody>
</table>

The table confirms what was observed in the plots: there is a much
higher concentration of the total ratings into the most popular movies
than there is into the most active users. This indicates that
statistical conclusions drawn from a reduced list of popular movies may
prove to have more significance in predicting results for the full set
than statistics drawn from a reduced list of most active users.

From this analysis of the individual users, we conclude that the typical
user is rather consistent in giving ratings, with the average standard
deviation at around 0.95 and the mean user average at 3.61. We also see
that the typical user is not very active, with half of the users having
less than 50 ratings in the *train* set. Finally, there is a clear
concentration of the total number of ratings into the most active users,
but it is not nearly as pronounced as observed across movies.

### Analysis of movie genres

Upon inspection of the `genres` column presented in the data, we see
that each entry is comprised of a string of characters with all genres
in which a movie fits, separated by “|”. Below we see a sample with 12
entries with relevant columns of the `movies` object:

    ## # A tibble: 12 x 3
    ##    movieId title                         genres                                 
    ##      <dbl> <chr>                         <chr>                                  
    ##  1   53956 Death at a Funeral            Comedy|Drama                           
    ##  2    1232 Stalker                       Drama|Mystery|Sci-Fi                   
    ##  3     457 Fugitive, The                 Thriller                               
    ##  4   26663 Monsieur Hire                 Crime|Romance|Thriller                 
    ##  5    5085 Carmen Jones                  Drama|Musical                          
    ##  6   26226 Hi, Mom!                      Comedy                                 
    ##  7    1183 English Patient, The          Drama|Romance|War                      
    ##  8    3648 Abominable Snowman, The       Horror|Sci-Fi                          
    ##  9   56748 Big Bad Swim, The             Comedy|Drama                           
    ## 10    2735 Golden Child, The             Action|Adventure|Comedy|Fantasy|Mystery
    ## 11   33264 Satan's Tango (SÃ¡tÃ¡ntangÃ³) Drama                                  
    ## 12   27735 Unstoppable                   Action|Adventure|Comedy|Drama|Thriller

After some data manipulation, we see that the `genres` variable contains
19 different genres, which are counted and arranged by number of
appearances in the table below:

<table>
<caption>Movie genres arranged by number of movies</caption>
<thead>
<tr class="header">
<th style="text-align: center;">#</th>
<th style="text-align: center;">Genre</th>
<th style="text-align: center;">Number of Movies</th>
<th style="text-align: center;">Average Rating</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;">1</td>
<td style="text-align: center;">Drama</td>
<td style="text-align: center;">5336</td>
<td style="text-align: center;">3.35</td>
</tr>
<tr class="even">
<td style="text-align: center;">2</td>
<td style="text-align: center;">Comedy</td>
<td style="text-align: center;">3703</td>
<td style="text-align: center;">3.12</td>
</tr>
<tr class="odd">
<td style="text-align: center;">3</td>
<td style="text-align: center;">Thriller</td>
<td style="text-align: center;">1705</td>
<td style="text-align: center;">3.18</td>
</tr>
<tr class="even">
<td style="text-align: center;">4</td>
<td style="text-align: center;">Romance</td>
<td style="text-align: center;">1685</td>
<td style="text-align: center;">3.30</td>
</tr>
<tr class="odd">
<td style="text-align: center;">5</td>
<td style="text-align: center;">Action</td>
<td style="text-align: center;">1473</td>
<td style="text-align: center;">3.06</td>
</tr>
<tr class="even">
<td style="text-align: center;">6</td>
<td style="text-align: center;">Crime</td>
<td style="text-align: center;">1117</td>
<td style="text-align: center;">3.30</td>
</tr>
<tr class="odd">
<td style="text-align: center;">7</td>
<td style="text-align: center;">Adventure</td>
<td style="text-align: center;">1025</td>
<td style="text-align: center;">3.16</td>
</tr>
<tr class="even">
<td style="text-align: center;">8</td>
<td style="text-align: center;">Horror</td>
<td style="text-align: center;">1013</td>
<td style="text-align: center;">2.80</td>
</tr>
<tr class="odd">
<td style="text-align: center;">9</td>
<td style="text-align: center;">Sci-Fi</td>
<td style="text-align: center;">754</td>
<td style="text-align: center;">2.97</td>
</tr>
<tr class="even">
<td style="text-align: center;">10</td>
<td style="text-align: center;">Fantasy</td>
<td style="text-align: center;">543</td>
<td style="text-align: center;">3.20</td>
</tr>
<tr class="odd">
<td style="text-align: center;">11</td>
<td style="text-align: center;">Children</td>
<td style="text-align: center;">528</td>
<td style="text-align: center;">3.00</td>
</tr>
<tr class="even">
<td style="text-align: center;">12</td>
<td style="text-align: center;">War</td>
<td style="text-align: center;">510</td>
<td style="text-align: center;">3.46</td>
</tr>
<tr class="odd">
<td style="text-align: center;">13</td>
<td style="text-align: center;">Mystery</td>
<td style="text-align: center;">509</td>
<td style="text-align: center;">3.33</td>
</tr>
<tr class="even">
<td style="text-align: center;">14</td>
<td style="text-align: center;">Documentary</td>
<td style="text-align: center;">481</td>
<td style="text-align: center;">3.47</td>
</tr>
<tr class="odd">
<td style="text-align: center;">15</td>
<td style="text-align: center;">Musical</td>
<td style="text-align: center;">436</td>
<td style="text-align: center;">3.27</td>
</tr>
<tr class="even">
<td style="text-align: center;">16</td>
<td style="text-align: center;">Animation</td>
<td style="text-align: center;">286</td>
<td style="text-align: center;">3.23</td>
</tr>
<tr class="odd">
<td style="text-align: center;">17</td>
<td style="text-align: center;">Western</td>
<td style="text-align: center;">275</td>
<td style="text-align: center;">3.31</td>
</tr>
<tr class="even">
<td style="text-align: center;">18</td>
<td style="text-align: center;">Film-Noir</td>
<td style="text-align: center;">148</td>
<td style="text-align: center;">3.72</td>
</tr>
<tr class="odd">
<td style="text-align: center;">19</td>
<td style="text-align: center;">IMAX</td>
<td style="text-align: center;">29</td>
<td style="text-align: center;">3.30</td>
</tr>
<tr class="even">
<td style="text-align: center;">20</td>
<td style="text-align: center;">(no genres listed)</td>
<td style="text-align: center;">1</td>
<td style="text-align: center;">3.67</td>
</tr>
</tbody>
</table>

We see that “Drama” is the most common movie genre. The table also shows
that there is a relevant difference between average movie ratings across
different genres. We can see the distribution of these average ratings
in the plot below:

<img src="HX9_MovieLens_Report_files/figure-markdown_strict/average ratings per movie genre-1.png" width="70%" height="70%" style="display: block; margin: auto;" />

As indicated in the previous table, all genres other than “IMAX” have
over 100 individual movies, which suggests that the relation between
movie genres and average movie ratings is an actual feature of the data
and not due to random variability.

We also note that each movie is not associated with a single genre, but
rather a collection of genres. In fact, 4 movies classified with 7 or 8
different genres:

<table>
<caption>Movies with 6+ different genres</caption>
<thead>
<tr class="header">
<th style="text-align: left;">Title</th>
<th style="text-align: right;"># Genres</th>
<th style="text-align: left;">Genres</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">Host, The (Gwoemul)</td>
<td style="text-align: right;">8</td>
<td style="text-align: left;">Action|Adventure|Comedy|Drama|Fantasy|Horror|Sci-Fi|Thriller</td>
</tr>
<tr class="even">
<td style="text-align: left;">Who Framed Roger Rabbit?</td>
<td style="text-align: right;">7</td>
<td style="text-align: left;">Adventure|Animation|Children|Comedy|Crime|Fantasy|Mystery</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Monster House</td>
<td style="text-align: right;">7</td>
<td style="text-align: left;">Adventure|Animation|Children|Comedy|Drama|Fantasy|Mystery</td>
</tr>
<tr class="even">
<td style="text-align: left;">Enchanted</td>
<td style="text-align: right;">7</td>
<td style="text-align: left;">Adventure|Animation|Children|Comedy|Fantasy|Musical|Romance</td>
</tr>
</tbody>
</table>

Some genres are more frequently found together. We calculate the
correlations between the columns in the `genres$wide` data frame and
show the highest and lowest pairs to demonstrate:

<table>
<caption>Movie genre pairs with highest correlation</caption>
<thead>
<tr class="header">
<th style="text-align: left;">Pair</th>
<th style="text-align: right;">Correlation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">Animation | Children</td>
<td style="text-align: right;">0.3566642</td>
</tr>
<tr class="even">
<td style="text-align: left;">Children | Musical</td>
<td style="text-align: right;">0.1819614</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Crime | Thriller</td>
<td style="text-align: right;">0.1758849</td>
</tr>
<tr class="even">
<td style="text-align: left;">Horror | Thriller</td>
<td style="text-align: right;">0.1756856</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Mystery | Thriller</td>
<td style="text-align: right;">0.1750220</td>
</tr>
<tr class="even">
<td style="text-align: left;">Children | Fantasy</td>
<td style="text-align: right;">0.1668665</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Crime | Film-Noir</td>
<td style="text-align: right;">0.1557878</td>
</tr>
<tr class="even">
<td style="text-align: left;">Adventure | Children</td>
<td style="text-align: right;">0.1495040</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Documentary | IMAX</td>
<td style="text-align: right;">0.1442926</td>
</tr>
<tr class="even">
<td style="text-align: left;">Action | Adventure</td>
<td style="text-align: right;">0.1412978</td>
</tr>
</tbody>
</table>

Not surprisingly, there is positive correlation between genres like
“animation” and “children”, as well as “mystery” and “thriller” or
“action” and “adventure”. Similar results appear when we look at the
bottom of the table, with the least correlated genres:

<table>
<caption>Movie genre pairs with lowest correlation</caption>
<thead>
<tr class="header">
<th style="text-align: left;">Pair</th>
<th style="text-align: right;">Correlation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">Children | Thriller</td>
<td style="text-align: right;">-0.2572242</td>
</tr>
<tr class="even">
<td style="text-align: left;">Crime | Fantasy</td>
<td style="text-align: right;">-0.2019309</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Animation | Thriller</td>
<td style="text-align: right;">-0.1972529</td>
</tr>
<tr class="even">
<td style="text-align: left;">Musical | Thriller</td>
<td style="text-align: right;">-0.1917192</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Action | Musical</td>
<td style="text-align: right;">-0.1883138</td>
</tr>
<tr class="even">
<td style="text-align: left;">Comedy | Thriller</td>
<td style="text-align: right;">-0.1803907</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Animation | Crime</td>
<td style="text-align: right;">-0.1641847</td>
</tr>
<tr class="even">
<td style="text-align: left;">Children | Horror</td>
<td style="text-align: right;">-0.1631741</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Romance | Sci-Fi</td>
<td style="text-align: right;">-0.1542794</td>
</tr>
<tr class="even">
<td style="text-align: left;">Adventure | Horror</td>
<td style="text-align: right;">-0.1510313</td>
</tr>
</tbody>
</table>

As you would imagine, certain pairs are not found often (though I
suspect writing a children’s thriller or an action musical would be an
interesting challenge for a movie writer).

From this analysis of the movies genres, we conclude that there is a
large difference in the frequency with which individual genres are used
to describe movies, with “Drama” and “Comedy” being the most frequent.
There is also a connection between the movie genres and the average
rating, with movies tagged with “Horror” averaging 2.80, while ones
tagged with “Film-Noir” average 3.72. We also note a significant
correlation between certain pairs of genres that are often found
together.

### Analysis of time

As a final step in the exploratory data analysis, we look at the
distribution of the year of release of each movie and how the average
ratings are distributed according to the year of release. We also
investigate how the number of years since the release of each movie at
the time of rating influences the rating.

The figure below contains a depiction of the number of movies released
at each year and the average rating for movies released at each year:

<img src="HX9_MovieLens_Report_files/figure-markdown_strict/average rating per year of release-1.png" width="70%" height="70%" style="display: block; margin: auto;" />

We see that the number of movies in the dataset increases dramatically
in the late 70s, as well as in the early 90s. We also see that the
average movie rating has a strong negative correlation with the year of
release.

It is also interesting to look at the average ratings as a function of
the number of years passed since the release of the movie at the time of
each rating. The plot below shows this information:

<img src="HX9_MovieLens_Report_files/figure-markdown_strict/average rating per years since release-1.png" width="70%" height="70%" style="display: block; margin: auto;" />

Again, we see that there is a strong connection between the movie age at
the time of rating and the value of the rating, demonstrating a tendency
of good movies to survive the test of time.

From this analysis, we conclude that there is a prevalence of newer
movies in the dataset, but that older movies (be it according to year of
release of age at the time of rating) tend to be better rated.

------------------------------------------------------------------------

Modeling
--------

We now proceed to use the hints provided by the exploratory data
analysis to construct models that predict movie ratings given by users.
We begin by constructing a reference model in which all predictions are
the global average rating calculated on the *train* set. Then, we create
increasingly complete models by applying the general approach described
below:

1.  Choose a previous model as reference
2.  Calculate residuals from the previous model applied to the *train*
    set
3.  Create a new model with an added term that explains the residuals
4.  Train parameters on the *test* set (if applicable)
5.  Evaluate performance on the *test* set

Finally, after development of a complete model resulting from the
combination of all the previous steps, we evaluate model performance on
the *validation* set in the Results section of the report.

### Naive model

We begin by constructing a simple model in which the global average
rating calculated in the *train* set (3.5125703) is given uniformly as
prediction to all ratings.

The model is depicted in mathematical notation below:

*Y*<sub>*u*, *m*</sub> = *μ* + *ϵ*
where:  
*Y*<sub>*u*, *m*</sub> = rating given by user *u* to movie *m*  
*μ* = mean global rating  
*ϵ* = random residual

The RMSE result obtained after application of this model to the *test*
set is below:

<table>
<caption>Results from application of Naive model (modelId = 1) to test set</caption>
<thead>
<tr class="header">
<th style="text-align: center;">modelId</th>
<th style="text-align: center;">RMSE</th>
<th style="text-align: center;">description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;">1</td>
<td style="text-align: center;">1.060704</td>
<td style="text-align: center;">Global average (benchmark)</td>
</tr>
</tbody>
</table>

### Simple movie effect

We proceed to include the movie effect in the model, representing the
fact that each movie has an inherent quality that makes it be qualified,
on average, above or below the global mean. The movie effect is
calculated for each movie as the mean residual obtained after
application of the naive model on the *train* set. The model with this
new parameter is this:

*Y*<sub>*u*, *m*</sub> = *μ* + *e*<sub>*m**o**v**i**e*</sub> + *ϵ*
  
where:  
*Y*<sub>*u*, *m*</sub> = rating given by user *u* to movie *m*  
*μ* = mean global rating  
*e*<sub>*m**o**v**i**e*</sub> = movie effect for movie *m*  
*ϵ* = random residual

As *e*<sub>*m*</sub> is modeled as the average points each movie
receives above the *train* set average, *μ* + *e*<sub>*m*</sub> is equal
to each individual movie average. This version of the model corresponds
to predicting that each movie is rated as the average of previous
ratings plus a random term.

Results on the *test* set are displayed below:

<table>
<caption>Results from application of Simple Movie Effect model (modelId = 2) to test set</caption>
<thead>
<tr class="header">
<th style="text-align: center;">modelId</th>
<th style="text-align: center;">RMSE</th>
<th style="text-align: center;">description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;">2</td>
<td style="text-align: center;">0.9437144</td>
<td style="text-align: center;">model 1 + simple movie effect</td>
</tr>
</tbody>
</table>

### Regularized movie effect

Although the results did improve, the fact that some movies are rated
very few times (as observed in the Exploratory Data Analysis section)
indicates that a prediction based solely on the movie average is not
adequate. In these situations, it is normally beneficial to use a
prediction that is closer to the global average for movies with few
ratings, and converges the the movie average as they receive a more
substantial number of ratings. A regularization of each movie’s mean
rating is appropriate to achieve this correction.

The model with the movie effect regularized according to the number of
ratings is represented via the equation below:

*Y*<sub>*u*, *m*</sub> = *μ* + *e*<sub>*m**o**v**i**e*</sub><sup>*r*</sup> + *ϵ*
  
where:  
*Y*<sub>*u*, *m*</sub> = rating given by user *u* to movie *m*  
*μ* = mean global rating  
*e*<sub>*m**o**v**i**e*</sub><sup>*r*</sup> = regularized movie effect  
*ϵ* = random residual

The regularized movie effect corresponding to each individual movie is
obtained via a modified version of the average deviation from the global
average, according to the equation below:

$$e\_m^r(\\lambda) = \\frac{1}{\\lambda+n\_i}\* \\sum\_{m=1}^{n\_i} (Y\_{m,i} - \\mu)$$
This indicates that the movie effect *e*<sub>*m*</sub><sup>*r*</sup>
depends on an arbitrary parameter *λ*, which can be chosen to minimize
the residual.

The figure below depicts the results obtained on the *test* set after
calculating the regularized movie effect on the *train* set with
different values of lambda, from which we choose the optimal.

<img src="HX9_MovieLens_Report_files/figure-markdown_strict/investigation of lambda for movie effect regularization-1.png" width="70%" height="70%" style="display: block; margin: auto;" />

The optimal regularization effect parameter *λ* was found to be 2.5.

From he RMSE results in the table above, it is evident that
regularization has not contributed with a substantial improvement. This
may be due to the fact that, as the dataset gets larger, instances of
movies with very few ratings become rarer and have a lesser effect on
the overall results.

The RMSE after application of this model on the *test* set is given
below:

<table>
<caption>Results from application of Regularized Movie Effect model (modelId = 3) to test set</caption>
<thead>
<tr class="header">
<th style="text-align: center;">modelId</th>
<th style="text-align: center;">RMSE</th>
<th style="text-align: center;">description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;">3</td>
<td style="text-align: center;">0.9436775</td>
<td style="text-align: center;">model 1 + regularized movie effect</td>
</tr>
</tbody>
</table>

### Simple user effect

Similarly to what was done for the movie averages, we proceed to modify
our model to account for the fact that different users tend to rate
differently. In order to estimate this effect, we calculate the
residuals after application of the previous model on the *train* set and
calculate the average residual for each user. We then add this term for
each user to get a new prediction.

As we include additional terms in the model, it is important to make
sure that predictions fall between 0.5 and 5, the minimum and maximum
possible ratings. To do so, we introduce the function `limitRange`,
defined as below:

$$limitRange\_{Y\_{min}}^{Y\_{max}}(Y) = \\begin{cases}Y\_{min} &\\text{, if } Y &lt; Y\_{min} \\\\ Y\_{max} &\\text{, if } Y &gt; Y\_{max} \\\\ Y &\\text{otherwise } \\end{cases}$$

The modified model is depicted below:

*Y*<sub>*u*, *m*</sub> = *l**i**m**i**t**R**a**n**g**e*<sub>0.5</sub><sup>5</sup>(*μ* + *e*<sub>*m**o**v**i**e*</sub><sup>*r*</sup> + *e*<sub>*u**s**e**r*</sub> + *ϵ*)

where:  
*Y*<sub>*u*, *m*</sub> = rating given by user *u* to movie *m*  
*μ* = mean global rating  
*e*<sub>*m**o**v**i**e*</sub><sup>*r*</sup> = regularized movie effect  
*e*<sub>*u**s**e**r*</sub> = simple user effect  
*ϵ* = random residual

After application of this model, we obtain the results below:

<table>
<caption>Results from application of User Effect model (modelId = 4) to test set</caption>
<thead>
<tr class="header">
<th style="text-align: center;">modelId</th>
<th style="text-align: center;">RMSE</th>
<th style="text-align: center;">description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;">4</td>
<td style="text-align: center;">0.8658639</td>
<td style="text-align: center;">model 3 + simple user effect</td>
</tr>
</tbody>
</table>

### Regularized user effect

As we did for the movie effect, we proceed to modify our model to
account for the fact that certain users have a low number of ratings and
that their average ratings may be heavily influenced by random
variability.

We evaluate several values of *λ* in the *test* set and select the one
that gives the lower value of the RMSE. In this case, the best *λ* is
4.5.

The modified model with the regularized user effect is depicted below:

*Y*<sub>*u*, *m*</sub> = *l**i**m**i**t**R**a**n**g**e*<sub>0.5</sub><sup>5</sup>(*μ* + *e*<sub>*m**o**v**i**e*</sub><sup>*r*</sup> + *e*<sub>*u**s**e**r*</sub><sup>*r*</sup> + *ϵ*)

where:  
*Y*<sub>*u*, *m*</sub> = rating given by user *u* to movie *m*  
*μ* = mean global rating  
*e*<sub>*m**o**v**i**e*</sub><sup>*r*</sup> = regularized movie effect  
*e*<sub>*u**s**e**r*</sub><sup>*r*</sup> = regularized user effect  
*ϵ* = random residual

The application of this model in the *test* set gives the results below.
We see that regularization does not introduce a great gain in
performance, which is explained by the fact the fact that we are using a
massive dataset and that ratings from users that are not very active do
not account for a significant portion of the total ratings.

<table>
<caption>Results from application of Regularized User Effect model (modelId = 5) to test set</caption>
<thead>
<tr class="header">
<th style="text-align: center;">modelId</th>
<th style="text-align: center;">RMSE</th>
<th style="text-align: center;">description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;">5</td>
<td style="text-align: center;">0.8655645</td>
<td style="text-align: center;">model 3 + regularized user effect</td>
</tr>
</tbody>
</table>

### Effect of movie genres (Drama movies)

As observed during the Exploratory Data Analysis section, there is a
connection between the movie ratings and the movie genres with which
each movie is tagged. However, this connection is already accounted for
in the previous model: since there is a one-to-one relation between a
movie and it’s tags (meaning that the tags are constant and do not
change across different ratings), this effect is already represented in
the movie average.

Regardless of that, we can investigate the residuals from the previous
model to determine whether each individual user is partial do a
particular genre. Users with a preference for a particular genre will
have higher residuals for movies tagged with that genre.

As a proof of concept, we now create a model with an added term
corresponding to the mean residual observed for each user for movies
either tagged or not tagged with a particular genre. For this, we choose
the Drama genre, which was observed to be the most frequent. We will
evaluate the residuals in the *train* set, calculate their average for
each user by grouping movies that are tagged with Drama and movies that
are not, and apply regularization by selecting the value of *λ* that
minimizes RMSE observed in the *test* set.

The corresponding model is represented as:

*Y*<sub>*u*, *m*</sub> = *l**i**m**i**t**R**a**n**g**e*<sub>0.5</sub><sup>5</sup>(*μ* + *e*<sub>*m**o**v**i**e*</sub><sup>*r*</sup> + *e*<sub>*u**s**e**r*</sub><sup>*r*</sup> + *e*<sub>*d**r**a**m**a*/*n**o**n* − *d**r**a**m**a*</sub><sup>*r*</sup> + *ϵ*)

where:  
*Y*<sub>*u*, *m*</sub> = rating given by user *u* to movie *m*  
*μ* = mean global rating  
*e*<sub>*m**o**v**i**e*</sub><sup>*r*</sup> = regularized movie effect  
*e*<sub>*u**s**e**r*</sub><sup>*r*</sup> = regularized user effect  
*e*<sub>*d**r**a**m**a*/*n**o**n* − *d**r**a**m**a*</sub><sup>*r*</sup>
= regularized drama/non-drama effect  
*ϵ* = random residual

To illustrate how this term is distributed across users, we look at a
scatter plot of the drama and non-drama effects for a sample of 500
random users:

<img src="HX9_MovieLens_Report_files/figure-markdown_strict/distribution of drama/non-drama effect-1.png" width="70%" height="70%" style="display: block; margin: auto;" />

The plot shows a clear negative correlation between the drama and
non-drama effects for each user. This is to be expected, since by
definition their average (weighted by the number of drama and non-drama
movies) is equal to each user’s mean residual.

In order to get a sense of how pronounced the preference is, look at a
plot of the spread between the drama and non-drama effects.

<img src="HX9_MovieLens_Report_files/figure-markdown_strict/spread between drama and non-drama effects-1.png" width="70%" height="70%" style="display: block; margin: auto;" />

The figure above contains the density plot for the difference between
the drama and non-drama effects for each user, with a blue line
depicting the average and the different shades of green background
depicting the intervals of one and two standard deviations from the
mean. The distribution is bell-shaped, centered at 0.008 and with a
standard deviation of 0.145. This indicates that, for 95% of users, the
difference between the average rating between movies with and without
the ‘Drama’ tag is below 0.29 points.

When we account for this additional piece in the model, we get the
following results:

<table>
<caption>Results from application of Drama/Non-Drama Effect model (modelId = 6) to test set</caption>
<thead>
<tr class="header">
<th style="text-align: center;">modelId</th>
<th style="text-align: center;">RMSE</th>
<th style="text-align: center;">description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;">6</td>
<td style="text-align: center;">0.8614085</td>
<td style="text-align: center;">model 5 + regularized drama/non-drama effects</td>
</tr>
</tbody>
</table>

### Genre effect through Principal Component Analysis (PCA)

Even though the results obtained with the previous model were an
improvement, we would like to extend to analysis for all of the
available movie genres. In theory, we could repeat the process of
calculating the residuals from one model and adding an effect for each
of the movie genres, but that would be repetitive and cumbersome. The
most appropriate way to tackle the effect of the movie genres on the
ratings is by calculating effects for all of them at the same time.

When addressing the effect of multiple variables at once, we must
consider the fact that there is a correlation between the predictors. As
observed during the exploratory data analysis, there is a correlation
between the tags associated with each of the genres. Consequently, if we
choose to calculate effects for each of the genres separately and add
them all together according to each movie’s genres, we risk “adding the
same thing twice”.

To account for this correlation, we take the matrix saved in the
`genres$wide` object, which contains columns indicating whether each
genre tag is part of each of the different genre combinations in the
dataset - and perform a Principal Component Analysis (PCA). The
resulting object is saved to the `genres$principal.components` object
and contains the principal components for the matrix, representing a
rotation to a set of coordinates that makes the columns uncorrelated. We
can think of principal components as “equivalent genres”, which are
independent from one another.

The object resulting from the PCA also contains, among other pieces of
information, the rotation matrix used to convert from the original
logical values indicating whether a movie is tagged with each of the
movie genres to the continuous values indicating how much the movie is
associated with each of the principal components.

An excerpt from the original data in the `genres$wide` object is shown
below:

    ##                          genres Action Crime Thriller Drama Sci-Fi Adventure Children
    ## 1         Action|Crime|Thriller   TRUE  TRUE     TRUE FALSE  FALSE     FALSE    FALSE
    ## 2  Action|Drama|Sci-Fi|Thriller   TRUE FALSE     TRUE  TRUE   TRUE     FALSE    FALSE
    ## 3       Action|Adventure|Sci-Fi   TRUE FALSE    FALSE FALSE   TRUE      TRUE    FALSE
    ## 4 Action|Adventure|Drama|Sci-Fi   TRUE FALSE    FALSE  TRUE   TRUE      TRUE    FALSE
    ## 5       Children|Comedy|Fantasy  FALSE FALSE    FALSE FALSE  FALSE     FALSE     TRUE
    ## 6      Comedy|Drama|Romance|War  FALSE FALSE    FALSE  TRUE  FALSE     FALSE    FALSE
    ##   Comedy Fantasy Romance   War Animation Musical Western Mystery Horror Film-Noir
    ## 1  FALSE   FALSE   FALSE FALSE     FALSE   FALSE   FALSE   FALSE  FALSE     FALSE
    ## 2  FALSE   FALSE   FALSE FALSE     FALSE   FALSE   FALSE   FALSE  FALSE     FALSE
    ## 3  FALSE   FALSE   FALSE FALSE     FALSE   FALSE   FALSE   FALSE  FALSE     FALSE
    ## 4  FALSE   FALSE   FALSE FALSE     FALSE   FALSE   FALSE   FALSE  FALSE     FALSE
    ## 5   TRUE    TRUE   FALSE FALSE     FALSE   FALSE   FALSE   FALSE  FALSE     FALSE
    ## 6   TRUE   FALSE    TRUE  TRUE     FALSE   FALSE   FALSE   FALSE  FALSE     FALSE
    ##   Documentary  IMAX (no genres listed)
    ## 1       FALSE FALSE              FALSE
    ## 2       FALSE FALSE              FALSE
    ## 3       FALSE FALSE              FALSE
    ## 4       FALSE FALSE              FALSE
    ## 5       FALSE FALSE              FALSE
    ## 6       FALSE FALSE              FALSE

We can also see an excerpt from the `genres$principal.components$x`
object, which contains the values of each of the principal components
for each movie in the dataset. The values of the components are
continuous, as opposed to the original values which where logical.

    ##           PC1          PC2         PC3         PC4         PC5         PC6         PC7
    ## 1  1.00349551  0.003621436  0.49748730  0.61938280 -0.26930415 -0.37886205  1.34364741
    ## 2  0.33032637  0.471594251  0.80738547  0.47828204  0.04356930 -0.09539502  1.15087991
    ## 3  0.81422863 -0.551347816 -0.24074062  0.06118877  0.64660798  0.09743760 -0.06494836
    ## 4  0.06850071 -1.012082459 -0.38194737  0.20887547  0.43083447 -0.19287805 -0.06549951
    ## 5  0.78925081 -0.222587165 -0.03439398 -0.15357760 -0.23793438  0.02523626 -0.07382492
    ## 6 -0.02024799  1.169497917 -0.88224028  0.35800661 -0.07928572  0.55251752 -0.12176358
    ##            PC8          PC9        PC10          PC11          PC12        PC13
    ## 1 -0.192368444  0.277132441  0.42534751  1.445321e-01  0.0106085226 0.340021354
    ## 2 -0.159675452  0.263385855  0.19135462  6.558184e-02  0.1541750581 0.382492360
    ## 3  0.032318203 -0.021048503 -0.05922673  5.188830e-04  0.0009563814 0.017746002
    ## 4 -0.046212295 -0.004455704 -0.03287435 -5.579304e-02  0.0151889011 0.035608842
    ## 5  0.108110459 -0.022334703 -0.05536344 -9.477939e-05  0.0240778548 0.001558162
    ## 6  0.005675863  0.070142334  0.17523281 -7.843739e-02 -0.1387569617 0.099984324
    ##         PC14        PC15        PC16         PC17          PC18          PC19
    ## 1 0.18917291  0.17754277 -0.11425865  0.420717408  8.216308e-05 -0.0190771562
    ## 2 0.09843175 -0.12563786  0.08213033 -0.458370669 -5.880179e-03 -0.0022170929
    ## 3 0.09800892 -0.02594786  0.05584165  0.006248834 -3.974770e-03  0.0001822967
    ## 4 0.03684341  0.13628102 -0.07711716  0.006848232  1.728377e-02  0.0014354283
    ## 5 0.04636914 -0.09177617  0.09060276  0.012174524 -1.294167e-02  0.0003711257
    ## 6 0.03688075 -0.05730162  0.05143720 -0.009178221 -6.703405e-02 -0.0009304504
    ##            PC20
    ## 1 -0.0002708846
    ## 2  0.0002805121
    ## 3  0.0001723632
    ## 4 -0.0003263937
    ## 5  0.0002995218
    ## 6  0.0002151488

Below we see the output of the `summary()` function applied to the
`genres$principal.componenents` object, which has been created using the
R function `prcomp()`. The summary depicts how much of the total
variance is explained by each of the principal components.

    ## Importance of components:
    ##                           PC1    PC2     PC3     PC4     PC5     PC6     PC7     PC8
    ## Standard deviation     0.5657 0.4936 0.37274 0.36291 0.34528 0.29156 0.28098 0.25651
    ## Proportion of Variance 0.2091 0.1592 0.09077 0.08604 0.07789 0.05554 0.05158 0.04299
    ## Cumulative Proportion  0.2091 0.3682 0.45900 0.54505 0.62293 0.67847 0.73005 0.77304
    ##                            PC9    PC10    PC11   PC12    PC13    PC14    PC15    PC16
    ## Standard deviation     0.23430 0.21974 0.20632 0.2033 0.19506 0.18766 0.17921 0.14922
    ## Proportion of Variance 0.03586 0.03155 0.02781 0.0270 0.02486 0.02301 0.02098 0.01455
    ## Cumulative Proportion  0.80890 0.84045 0.86826 0.8952 0.92011 0.94312 0.96410 0.97865
    ##                           PC17    PC18    PC19     PC20
    ## Standard deviation     0.13030 0.11397 0.05120 0.009675
    ## Proportion of Variance 0.01109 0.00849 0.00171 0.000060
    ## Cumulative Proportion  0.98974 0.99823 0.99994 1.000000

In order to include this information in the model, we begin by grouping
all the ratings in the *train* set by users and performing weighted
averages of the residuals of every rating given by each user. These
averages are weighted by the values of each of the principal components
associated with each movie, so that every user ends up with 20 scores
indicating how partial they are to each of the principal components.
These values are then regularized, so that results for users with a
lower number of ratings have reduced importance. The resulting model
follows the equation below:

$$Y\_{u,m} = limitRange\_{0.5}^5(\\mu + e\_{movie}^r + e\_{user}^r+ \\sum\_{n = 1}^{N\_{PC}}e\_{genre(PC)}^r + \\epsilon)$$

where:  
*Y*<sub>*u*, *m*</sub> = rating given by user *u* to movie *m*  
*μ* = mean global rating  
*e*<sub>*m**o**v**i**e*</sub><sup>*r*</sup> = regularized movie effect  
*e*<sub>*u**s**e**r*</sub><sup>*r*</sup> = regularized user effect  
*N*<sub>*P**C*</sub> = number of principal components  
*e*<sub>*g**e**n**r**e*(*P**C*)</sub><sup>*r*</sup> = regularized genre
effect  
*ϵ* = random residual

Here, we use all of the 20 principal components of the movie genres
tagging system, so *N*<sub>*P**C*</sub> = 20. The value of *λ* that
minimizes the residuals after application of this model in the *test*
set is 35, and the RMSE in the *test* set is displayed below:

<table>
<caption>Results from application of Genre Effect model (modelId = 7) to test set</caption>
<thead>
<tr class="header">
<th style="text-align: center;">modelId</th>
<th style="text-align: center;">RMSE</th>
<th style="text-align: center;">description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;">7</td>
<td style="text-align: center;">0.849572</td>
<td style="text-align: center;">model 5 + regularized genre effect through PCA</td>
</tr>
</tbody>
</table>

### Movie age effect

We now turn our attention to the effect that the year of release
(extracted from the movie title in the original dataset) and the date of
the rating have on the ratings. As was the case with the movie genres,
looking at the year of release in an isolated manner brings no new
information, since there is a one-to-one connection between the movie
and its age of release. Any effect strictly from the year of release
would have already been captured in the movie average rating, already
accounted for in previous models.

However, there is one aspect that can still be explored, which is the
age of the movie at the time of each rating. We get that by subtracting
each movie’s year of release from the year each rating was given.

A plot of the average residuals as a function of the movie age at time
of rating shows interesting features:

<img src="HX9_MovieLens_Report_files/figure-markdown_strict/average residuals vs movie age-1.png" width="70%" height="70%" style="display: block; margin: auto;" />

We can see that the average residual is positive for the first two years
after a movie has been release, which seems to indicate some enthusiasm
from avid users who like to watch the movies as soon as possible. There
is then a period of negative residuals up to around 15 years after
release. Finally, the average residual increases until it starts to
oscillate around zero, when it appears there is a renewed interest for
them (or at least they are considered good enough to be worth watching
and rating). For longer periods, the curve gets very noisy, since there
are considerably less movies and ratings for these periods.

Because there is a long interval of movie age for which there are few
data points, this effect benefits a lot from regularization. The value
of *λ* that minimizes the residuals in the *test* set is 400,
considerably larger than for the other phenomenons.

The model that incorporates this feature of the data is represented
below, followed by the results obtained after application on the *test*
set.

$$Y\_{u,m} = limitRange\_{0.5}^5(\\mu + e\_{movie}^r + e\_{user}^r+ \\sum\_{n = 1}^{N\_{PC}}e\_{genre(PC)}^r + e\_{age}^{r} + \\epsilon)$$

where:  
*Y*<sub>*u*, *m*</sub> = rating given by user *u* to movie *m*  
*μ* = mean global rating  
*e*<sub>*m**o**v**i**e*</sub><sup>*r*</sup> = regularized movie effect  
*e*<sub>*u**s**e**r*</sub><sup>*r*</sup> = regularized user effect  
*N*<sub>*P**C*</sub> = number of principal components  
*e*<sub>*g**e**n**r**e*(*P**C*)</sub><sup>*r*</sup> = regularized genre
effect  
*e*<sub>*a**g**e*</sub><sup>*r*</sup> = regularized movie age effect  
*ϵ* = random residual

<table>
<caption>Results from application of Movie Age Effect model (modelId = 8) to test set</caption>
<thead>
<tr class="header">
<th style="text-align: center;">modelId</th>
<th style="text-align: center;">RMSE</th>
<th style="text-align: center;">description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;">8</td>
<td style="text-align: center;">0.849096</td>
<td style="text-align: center;">model 7 + regularized movie age effect</td>
</tr>
</tbody>
</table>

### Collaborative filtering

As a final step in the modeling phase, we implement a collaborative
filtering technique to improve the predictions.

In previous sections, we investigated how each of the predictors related
directly with the ratings. Collaborative filtering, on the other hand,
looks at how predictors are indirectly related to the ratings. An
User-Based Collaborative Filter (UBCF) looks at how similar users have
rated a particular movie to come to a prediction. An Item-Based
Collaborative Filter (IBCF) looks at how the user has rated similar
movies to come to a prediction.

Both approaches required identifying similarities between either
different users or different items (movies, in our case).

Here, we prefer to use a IBCF due to the fact that there is a higher
concentration of ratings in popular movies than there is in active
users, as revealed during the exploratory data analysis phase. This
permits some simplifications - instead of calculating similarities
between every pair of movies in the dataset, we only calculate
similarities between each movie and a selection of the 1000 most popular
movies. This approach drastically reduces the dimensions of the matrices
containing these similarities.

As a measure of similarity between movies, we choose to calculate the
correlation between the residuals obtained after application of the
previous model. We calculate the correlation based solely on the users
that have rated both movies.

The corrections according to the user’s rating to a particular movie is
given by the equation below:

$$e\_{IBCF}^{u,m} = \\frac {M\_{u,m\_R} \* C\_{m\_R,m}} {\\sum\_{m\_R}^{}C\_{m\_R,m}}$$
where:  
*e*<sub>*I**B**C**F*</sub><sup>*u*, *m*</sup> = adjustment to prediction
of rating of user *u* to movie *m* according to ratings given by user
*u* to similar movies  
*M*<sub>*u*, *m*<sub>*R*</sub></sub> = ratings given by user *u* to set
of most popular movies  
*C*<sub>*m*<sub>*R*</sub>, *m*</sub> = correlation matrix between each
movie in the complete dataset and each of the most popular movies,
calculated from the set of common users between each pair of movies  
$\\sum\_{m\_R}^{}C\_{m\_R,m}$ = row sums of
*C*<sub>*m*<sub>*R*</sub>, *m*</sub>

Because the value of *e*<sub>*I**B**C**F*</sub><sup>*u*, *m*</sup> is
unique for each user/movie combination, it is impracticable to represent
the full set of values in usual matrix form. The package `Matrix` is
used to represent these values in sparse matrix form.

In order to further reduce the need for computational resources, the
sets of users and movies are partitioned and the values of
*e*<sub>*I**B**C**F*</sub><sup>*u*, *m*</sup> are calculated
sequentially for each combination of this partitions. After some
experimentation, we choose to create 7 partitions of users and 10
partitions of movies, so that the full corrections matrix is completed
in 70 steps.

With the inclusion of this features, we reach our complete model for
predicting movie ratings. It is expressed mathematically as:

$$Y\_{u,m} = limitRange\_{0.5}^5(\\mu + e\_{movie}^r + e\_{user}^r+ \\sum\_{n = 1}^{N\_{PC}}e\_{genre(PC)}^r + e\_{age}^{r} + e\_{IBCF}^{u,m} + \\epsilon)$$

where:  
*Y*<sub>*u*, *m*</sub> = rating given by user *u* to movie *m*  
*μ* = mean global rating  
*e*<sub>*m**o**v**i**e*</sub><sup>*r*</sup> = regularized movie effect  
*e*<sub>*u**s**e**r*</sub><sup>*r*</sup> = regularized user effect  
*N*<sub>*P**C*</sub> = number of principal components  
*e*<sub>*g**e**n**r**e*(*P**C*)</sub><sup>*r*</sup> = regularized genre
effect  
*e*<sub>*a**g**e*</sub><sup>*r*</sup> = regularized movie age effect  
*e*<sub>*I**B**C**F*</sub><sup>*u*, *m*</sup> = adjustment to prediction
of rating of user *u* to movie *m* according to IBCF  
*ϵ* = random residual

The results after application of this model on the *test* set are
depicted below:

<table>
<caption>Results from application of Item-Based Collaborative Filter model (modelId = 9) to test set</caption>
<thead>
<tr class="header">
<th style="text-align: center;">modelId</th>
<th style="text-align: center;">RMSE</th>
<th style="text-align: center;">description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;">9</td>
<td style="text-align: center;">0.8431708</td>
<td style="text-align: center;">model 8 + similar movies effect</td>
</tr>
</tbody>
</table>

Results
=======

The final model has been constructed by considering the rating averages
for each particular movie and user. It also considers each user’s
individual preference for the different movie genres and the effect that
the number of years since movie release has on the ratings. Finally,
ratings are adjusted according to how each user has rated similar movies
(from a list of the 1000 most popular movies). The mathematical
representation of the full model is repeated here for convenience:

$$Y\_{u,m} = limitRange\_{0.5}^5(\\mu + e\_{movie}^r + e\_{user}^r+ \\sum\_{n = 1}^{N\_{PC}}e\_{genre(PC)}^r + e\_{age}^{r} + \\epsilon)$$

where:  
*Y*<sub>*u*, *m*</sub> = rating given by user *u* to movie *m*  
*μ* = mean global rating  
*e*<sub>*m**o**v**i**e*</sub><sup>*r*</sup> = regularized movie effect  
*e*<sub>*u**s**e**r*</sub><sup>*r*</sup> = regularized user effect  
*N*<sub>*P**C*</sub> = number of principal components  
*e*<sub>*g**e**n**r**e*(*P**C*)</sub><sup>*r*</sup> = regularized genre
effect  
*e*<sub>*a**g**e*</sub><sup>*r*</sup> = regularized movie age effect  
*ϵ* = random residual

The results obtained as we applied each step in the modeling phase to
the *test* set were presented in their respective sections. Here, we
aggregate the results from all versions of model after application to
the *train*, *test*, and *validation* sets:

<table>
<thead>
<tr class="header">
<th style="text-align: center;">modelId</th>
<th style="text-align: center;">RMSE (train set)</th>
<th style="text-align: center;">RMSE (test set)</th>
<th style="text-align: center;">RMSE (validation set)</th>
<th style="text-align: center;">Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;">1</td>
<td style="text-align: center;">1.0602380</td>
<td style="text-align: center;">1.0607045</td>
<td style="text-align: center;">1.0612019</td>
<td style="text-align: center;">Global average (benchmark)</td>
</tr>
<tr class="even">
<td style="text-align: center;">2</td>
<td style="text-align: center;">0.9421948</td>
<td style="text-align: center;">0.9437144</td>
<td style="text-align: center;">0.9440443</td>
<td style="text-align: center;">model 1 + simple movie effect</td>
</tr>
<tr class="odd">
<td style="text-align: center;">3</td>
<td style="text-align: center;">0.9422340</td>
<td style="text-align: center;">0.9436775</td>
<td style="text-align: center;">0.9439736</td>
<td style="text-align: center;">model 1 + regularized movie effect</td>
</tr>
<tr class="even">
<td style="text-align: center;">4</td>
<td style="text-align: center;">0.8554970</td>
<td style="text-align: center;">0.8658639</td>
<td style="text-align: center;">0.8660464</td>
<td style="text-align: center;">model 3 + simple user effect</td>
</tr>
<tr class="odd">
<td style="text-align: center;">5</td>
<td style="text-align: center;">0.8560926</td>
<td style="text-align: center;">0.8655645</td>
<td style="text-align: center;">0.8656782</td>
<td style="text-align: center;">model 3 + regularized user effect</td>
</tr>
<tr class="even">
<td style="text-align: center;">6</td>
<td style="text-align: center;">0.8477715</td>
<td style="text-align: center;">0.8614085</td>
<td style="text-align: center;">0.8613803</td>
<td style="text-align: center;">model 5 + regularized drama/non-drama effects</td>
</tr>
<tr class="odd">
<td style="text-align: center;">7</td>
<td style="text-align: center;">0.8181879</td>
<td style="text-align: center;">0.8495720</td>
<td style="text-align: center;">0.8494644</td>
<td style="text-align: center;">model 5 + regularized genre effect through PCA</td>
</tr>
<tr class="even">
<td style="text-align: center;">8</td>
<td style="text-align: center;">0.8177398</td>
<td style="text-align: center;">0.8490960</td>
<td style="text-align: center;">0.8490089</td>
<td style="text-align: center;">model 7 + regularized movie age effect</td>
</tr>
<tr class="odd">
<td style="text-align: center;">9</td>
<td style="text-align: center;">0.7879400</td>
<td style="text-align: center;">0.8431708</td>
<td style="text-align: center;">0.8430003</td>
<td style="text-align: center;">model 8 + similar movies effect</td>
</tr>
</tbody>
</table>

We see from the table that the results obtained on the *validation* set
are very close to the ones obtained on the *test* set, which indicates
that the validation strategy (partitioninig the `edx` dataset into a
*train* and a *test* sets) was appropriate and that the tuning
parameters defined for each aspect of the model are robust and not
subject to overtraining. The results on the *train* set are considerably
better, which does indicate that they are not representative of what
should be expected when exposing the model to new datasets.

We also see from the results that the improvement to the RMSE as new
effects are incorporated into the model are very subtle and seem to
converge to around 0.84. This reflects the fact the ratings have an
intrinsic level of random variability and that there is a limit to the
level of precision that any model can attain.

However, this subtle improvements to the RMSE should not be interpreted
as insignificant performance improvements in practice. Ultimately, the
goal of these models is not to predict ratings, but rather to create
movie recommendations for each particular user. Because the objective is
to use the models to create an ordered list of movie recommendations, a
small change in the predicted ratings can have a big impact in the order
at which movies are recommended.

The table contains the top 20 recommendations given to a randomly
selected user by 3 of the models and illustrates this feature:

<table>
<caption>Movie recommendations from 3 models to a random user</caption>
<thead>
<tr class="header">
<th style="text-align: left;">Model 5: regularized user effect</th>
<th style="text-align: left;">Model 7: regularized genre effect</th>
<th style="text-align: left;">Model 9 (complete model): similar movies effect</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">More</td>
<td style="text-align: left;">More</td>
<td style="text-align: left;">Maltese Falcon, The (a.k.a. Dangerous Female)</td>
</tr>
<tr class="even">
<td style="text-align: left;">Shawshank Redemption, The</td>
<td style="text-align: left;">Dark Knight, The</td>
<td style="text-align: left;">Shawshank Redemption, The</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Who’s Singin’ Over There? (a.k.a. Who Sings Over There) (Ko to tamo peva)</td>
<td style="text-align: left;">Yojimbo</td>
<td style="text-align: left;">In a Lonely Place</td>
</tr>
<tr class="even">
<td style="text-align: left;">Human Condition II, The (Ningen no joken II)</td>
<td style="text-align: left;">Double Indemnity</td>
<td style="text-align: left;">More</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Casablanca</td>
<td style="text-align: left;">Shawshank Redemption, The</td>
<td style="text-align: left;">Dark Knight, The</td>
</tr>
<tr class="even">
<td style="text-align: left;">Double Indemnity</td>
<td style="text-align: left;">Oldboy</td>
<td style="text-align: left;">Yojimbo</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Sunset Blvd. (a.k.a. Sunset Boulevard)</td>
<td style="text-align: left;">Chinatown</td>
<td style="text-align: left;">Pather Panchali</td>
</tr>
<tr class="even">
<td style="text-align: left;">Satan’s Tango (SÃ¡tÃ¡ntangÃ³)</td>
<td style="text-align: left;">Memento</td>
<td style="text-align: left;">Chinatown</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Paths of Glory</td>
<td style="text-align: left;">Maltese Falcon, The (a.k.a. Dangerous Female)</td>
<td style="text-align: left;">Rashomon (RashÃ´mon)</td>
</tr>
<tr class="even">
<td style="text-align: left;">Rear Window</td>
<td style="text-align: left;">City of God (Cidade de Deus)</td>
<td style="text-align: left;">Oldboy</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Dark Knight, The</td>
<td style="text-align: left;">Big Sleep, The</td>
<td style="text-align: left;">Double Indemnity</td>
</tr>
<tr class="even">
<td style="text-align: left;">Lives of Others, The (Das Leben der Anderen)</td>
<td style="text-align: left;">Fight Club</td>
<td style="text-align: left;">Memento</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Yojimbo</td>
<td style="text-align: left;">Rashomon (RashÃ´mon)</td>
<td style="text-align: left;">M</td>
</tr>
<tr class="even">
<td style="text-align: left;">Wallace &amp; Gromit: A Close Shave</td>
<td style="text-align: left;">Strangers on a Train</td>
<td style="text-align: left;">Bourne Identity, The</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Big Sleep, The</td>
<td style="text-align: left;">Blade Runner</td>
<td style="text-align: left;">Big Sleep, The</td>
</tr>
<tr class="even">
<td style="text-align: left;">North by Northwest</td>
<td style="text-align: left;">North by Northwest</td>
<td style="text-align: left;">Sunset Blvd. (a.k.a. Sunset Boulevard)</td>
</tr>
<tr class="odd">
<td style="text-align: left;">M</td>
<td style="text-align: left;">Rear Window</td>
<td style="text-align: left;">City of God (Cidade de Deus)</td>
</tr>
<tr class="even">
<td style="text-align: left;">City of God (Cidade de Deus)</td>
<td style="text-align: left;">M</td>
<td style="text-align: left;">North by Northwest</td>
</tr>
<tr class="odd">
<td style="text-align: left;">General, The</td>
<td style="text-align: left;">Sunset Blvd. (a.k.a. Sunset Boulevard)</td>
<td style="text-align: left;">Rear Window</td>
</tr>
<tr class="even">
<td style="text-align: left;">Raiders of the Lost Ark (Indiana Jones and the Raiders of the Lost Ark)</td>
<td style="text-align: left;">Star Wars: Episode IV - A New Hope (a.k.a. Star Wars)</td>
<td style="text-align: left;">Star Wars: Episode IV - A New Hope (a.k.a. Star Wars)</td>
</tr>
</tbody>
</table>

Finally, the result of the RMSE after application of the complete model
to the *validation* set is approximately 0.843.

------------------------------------------------------------------------

Conclusion
==========

This project covered the construction of a statistical model to predict
movie ratings. A *train* set of approximately 7.2 million ratings was
used to create predictions with parameters tuned with a *test* set of
approximately 1.8 million ratings. Finally, model performance was
measured on a *validation* set with approximately 1 million ratings. The
Root Mean Squared Error (RMSE) was used as a metric, and a RMSE of 0.843
was reached with the complete model in the *validation* set.

Models were created in the Modeling section based on trends identified
in the Exploratory Data Analysis section. Increasingly complex models
were created by including terms that accounted for specific trends
present in the residuals from the previous model.

In a practical setting, the results of the prediction models would be
used to predict ratings to movies that have not already been
watched/rated by a particular user. Movies would be ordered according to
their predicted ratings and recommended to users. In this sense, the
prediction and its RMSE are not the actual outcome of the model.
Instead, a proper outcome would be a list of recommended movies.

Considering this fact, a list of recommendations generated by some of
the models for a random user was also presented in the Results section.
We notice that, even though the RMSE decreases very slightly with each
step in the modeling process, the list of recommended movies changes
more drastically.

One point not taken into consideration during the project is the fact
that, in a practical setting, there are sources of “voting” other than
the ratings given by users. Many times, users do not go through the
trouble of rating a movie they’ve just watched. User are also not very
consistent in givin ratings, and there is an intrinsic random component
to a rating. A potentially more illustrative indication of an user’s
preferences may be the list of movies he or she have watched, regardless
of ratings.

We also note that the strategy for partitioning the full set of ratings
may be somewhat misleading. We randomly selected proportions of the
original *MovieLens 10M* dataset, but a more relevant manner of
partitioning the dataset would be to consider the date of rating. For
instance, we could have set apart ratings from the last year in the
dataset for validation. With this approach, we wouldn’t be using future
ratings to predict past ratings.

A potential point of improvement would be to perform further training on
the collaborative filter portion of the model. The number of movies in
the set of popular movies, used as reference for calculating a matrix of
correlations and identifying similar movies was chosen from limited
experimentation with a few set of values. The sizes of the users and
movies partitions were also not fully explored to produce results with
the lowest RMSE.

This was mostly due to the high demand for computational power.
Calculating the predictions for one single set of values took nearly one
hour in a home computer, so performing training with a large set of
parameters would have been impractical. One possible solution would be
to make use of the parallel computing capabilities in the `caret`
package, which allows for the evaluation of performance for different
combinations of parameters to be evaluated in parallel using additional
processor cores.
