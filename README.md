# Fabio Oliveira's blog & projects & rants

_About me: a dude delving into Data Science & Machine Learning and documenting his exploits_   
_contact info: [email](mailto:fabio.afdo@gmail.com), [LinkedIn](https://www.linkedin.com/in/fabioarbacholiveira/), [GitHub](https://github.com/fabio-a-oliveira)_

<!---
***

_DD/MM/YYYY (updated DD/MM/YYYY)_

## Running list of short and sweet DS/ML analogies    

PCA
BN
Nesterov --->

***

_Nov 26, 2020_
## [Mechanisms-of-Action prediction](https://fabio-a-oliveira.github.io/2020-11-26%20HX9_CYO_Report.html)

A statistical model for predicting the Mechanism of Action of a drug based on gene expression and cell viability data.

A _**mechanism of action (MoA)**_ is a label attributed to an agent to describe its biological activity.  By being able to properly identify a molecule’s MoA, it can subsequently be used in a targeted manner to obtain a desired cell response.

The final model is a weighted average of predictions from several individual models: Logistic Regression, K-nearest neighbors, Naive Bayes with loess smoothing, Support Vector Machine and Multi-class Penalized Mixture Discriminant Analysis.   

Created as part of the capstone project for the [HarvardX Data Science Professional Certificate](https://www.edx.org/professional-certificate/harvardx-data-science) and inspired by a [Kaggle competition](https://www.kaggle.com/c/lish-moa).

[Read...](https://fabio-a-oliveira.github.io/2020-11-26%20HX9_CYO_Report.html)  

***

_Sep 21, 2020_
## [A model for predicting movie ratings](https://fabio-a-oliveira.github.io/2020-09-21%20HX9_MovieLens_Report.html)

A model for predicting movie ratings on the [MovieLens](https://grouplens.org/datasets/movielens/) dataset. A series of models of increased complexity are proposed considering user and movie average ratings, user preferences for particular movie genres, movie age at the time of rating, and finally the application of Item-Based Collaborative Filtering.

Created as part of the capstone project for the [HarvardX Data Science Professional Certificate](https://www.edx.org/professional-certificate/harvardx-data-science).    

A very simple web-based recommendation system based on a minimalist version of the model is hosted at [https://fabio-a-oliveira.shinyapps.io/MovieRecommenderApp/](https://fabio-a-oliveira.shinyapps.io/MovieRecommenderApp/).   

<iframe src="https://fabio-a-oliveira.shinyapps.io/MovieRecommenderApp/" width="100%" height="500">
</iframe>


[Read...](https://fabio-a-oliveira.github.io/2020-09-21%20HX9_MovieLens_Report.html)

***
<!---
_DD/MM/YYYY_
## Maximum likelihood, overfitting and romance

Overfitting is the price we pay for answering the wrong question.   

Read...

***
--->

_Aug 4, 2020_
## [Stock data reduction via identification of “pivot points”](https:/fabio-a-oliveira.github.io/2020-08-04_stock_data_simplification.html)

A naive encoding of time series data, created before I knew anything about recurrent neural networks. A combination of a 1D conv-net for encoding and a recurrent network for inference would have been much better suited to the task. Cool visualizations though!   

<img src="2020-08-04_animation_sequential_loop.gif" width="80%" class="center">

[Read...](https:/fabio-a-oliveira.github.io/2020-08-04_stock_data_simplification.html)   

***

_Jul 30, 2020_
## [Gender bias in research funding](https://fabio-a-oliveira.github.io/2020-07-30%20Research%20Funding.html)

An investigation on the data from a [2015 PNAS research paper](https://www.pnas.org/content/112/40/12349) (used as a pretext for practicing R-Markdown and data visualization with ggplot2).   

<img src="2020-07-30%2001.png" width="80%" class="center">

[Read...](https://fabio-a-oliveira.github.io/2020-07-30%20Research%20Funding.html)




