<!---
# Fabio Oliveira's blog

_About me: an Engineer with background in the Auto and Aerospace industries, now delving into Data Science & Machine Learning and documenting his exploits_    

_reach out: [email](mailto:fabio.afdo@gmail.com), [LinkedIn](https://www.linkedin.com/in/fabioarbacholiveira/), [GitHub](https://github.com/fabio-a-oliveira)_

***
--->

_May 3, 2021_

## [Beyond CTRL+F: using word embeddings to implement a semantic search](https://fabio-a-oliveira.github.io/2021-05-02_Semantic_Search.html)

Trying to find a passage within a long document can be very annoying when we don't recall it exactly. We have to either search for a single word and browse through what can be dozens of matches, or we can try to narrow our search down by trial-and-error with different combinations of words.

With word embeddings, we can do much better! We can use them to create a dense vector that encodes the ___meaning___ of a desired passage, and then search the document for an excerpt that matches our desired meaning.

In [this article](https://fabio-a-oliveira.github.io/2021-05-02_Semantic_Search.html), I illustrate the concept by searching the classic _Pride and Prejudice_ for passages that correspond to either simplified or translated parts of the book.

<a href="https://fabio-a-oliveira.github.io/2021-05-02_Semantic_Search.html"><img src="assets/2021-05-02_austen03.jpg" width="100%"></a>

[Read...](https://fabio-a-oliveira.github.io/2021-05-02_Semantic_Search.html)   

***

_April 24, 2021_
## [A simplistic app to showcase a couple of NLP applications to aviation regulations](https://pywebio-nlp-demo.herokuapp.com/)

Aviation is a heavily regulated industry. There are tons of regulations and advisory material both at the global level (stemming mostly from ICAO, the UN's arm for civil aviation) and at the local level (from each country's own civil aviation authorities). Even though there is a huge effort put into interoperability, this sea of regulations is still far from standardized.

This vast variety makes the field perfect for the application of Natural Language Processing techniques.

This simple app showcases two such applications. In the first, the user is prompted to choose a requirement (either from a list taken from Brazilian regulations or from a free text input field) and a model tries to find the corresponding requirement in the US regulations. In the second, the user can select a requirement (either from the US or Brazilian regulations or from a free text input field) and a model determines whether that rule applies to the aircraft or to the operator.

In both applications, requirements are restricted to those applied to airlines, but the concept can easily be extended to text of any nature.

You can visit the web app [at this link](https://pywebio-nlp-demo.herokuapp.com/). You can also check the app code [here](https://github.com/fabio-a-oliveira/PyWebIO_NLP_demo), or the code for the comparison technique and classification model [here](https://github.com/fabio-a-oliveira/NLP_Regulations).

<a href="https://pywebio-nlp-demo.herokuapp.com/"><img src="2021-04-24_PyWebIO_NLP_demo.png" width="100%"></a>

[Visit...](https://pywebio-nlp-demo.herokuapp.com/)

***

_April 1, 2021_
## [Generating aviation regulations with RNNs](https://fabio-a-oliveira.github.io/2021-04-01_14-CFR-FAA.html)

Over the last decade, I spent of lot of time ensuring that aircraft, operations and airmen comply with civil aviation regulations from around the globe. I thought this topic would be an interesting pick for exploring the application of Recurrent Neural Networks to the creation of original text.   

In <a href="https://fabio-a-oliveira.github.io/2021-04-01_14-CFR-FAA.html">this article</a>, I explain how I trained a stack of Gated Recurrent Units on the full corpus of FAA regulations and discuss the results of using the model to generate new text following the same style. You can also see a teaser in the block below or visit [this link](https://raw.githubusercontent.com/fabio-a-oliveira/14-CFR-FAA/main/generated_text/generated_text___1000000_chars__2021-03-27__21-15-26.txt) for a 1-million characters sample of original regulatory text.

<a href="https://fabio-a-oliveira.github.io/2021-04-01_14-CFR-FAA.html"><img src="2021-04-01_generated_47_80.png" width="100%"></a>   

[Read...](https://fabio-a-oliveira.github.io/2021-04-01_14-CFR-FAA.html)    

***

_March 10, 2021_
## [Generating original music in the style of Bach's four-part chorales](https://fabio-a-oliveira.github.io/2021-03-10_Bach_chorales.html)

I'm sure by now everyone has seen examples of AI applications where an algorithm is left to train for some time on a large corpus of text or music and can then be used to create new content following the same style. This project is my take on training a recurrent neural network on the full set of JS Bach's four-part chorale pieces and then using it to create some new, original music.   

<a href="https://fabio-a-oliveira.github.io/2021-03-10_Bach_chorales.html">This article</a> gives a high-level description of the project and showcases some examples generated by the trained network. If you want to skip the details and just listen to some music, these are two of my favorite pieces:

<script src="https://cdn.jsdelivr.net/combine/npm/tone@14.7.58,npm/@magenta/music@1.21.0/es6/core.js,npm/focus-visible@5,npm/html-midi-player@1.1.1"></script>

<midi-player
  src="2021-03-10_new_chorale__2021-03-08__12-26-39__051__.mid"
  sound-font = "https://storage.googleapis.com/magentadata/js/soundfonts/sgm_plus">
</midi-player>

<midi-player
  src="2021-03-10_new_chorale__2021-03-08__12-26-39__059__.mid"
  sound-font = "https://storage.googleapis.com/magentadata/js/soundfonts/sgm_plus">
</midi-player>

[Read...](https://fabio-a-oliveira.github.io/2021-03-10_Bach_chorales.html)   

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

A very simple web-based recommendation system based on a minimalist version of the model is hosted at [https://fabio-a-oliveira.shinyapps.io/MovieRecommenderApp/](https://fabio-a-oliveira.shinyapps.io/MovieRecommenderApp/). You can also use it in the frame below:       

<iframe src="https://fabio-a-oliveira.shinyapps.io/MovieRecommenderApp/" width="100%" height="600">
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
## [Stock data reduction via identification of “pivot points”](https://fabio-a-oliveira.github.io/2020-08-04_stock_data_simplification.html)

A naive encoding of time series data, created before I knew anything about recurrent neural networks. A combination of a 1D conv-net for encoding and a recurrent network for inference would have been much better suited to the task. Cool visualizations though!   

<img src="2020-08-04_animation_sequential_loop.gif" width="80%" class="center">

[Read...](https://fabio-a-oliveira.github.io/2020-08-04_stock_data_simplification.html)   

***

_Jul 30, 2020_
## [Gender bias in research funding](https://fabio-a-oliveira.github.io/2020-07-30_Research_Funding.html)

An investigation on the data from a [2015 PNAS research paper](https://www.pnas.org/content/112/40/12349) (used as a pretext for practicing R-Markdown and data visualization with ggplot2).   

<img src="2020-07-30%2001.png" width="80%" class="center">

[Read...](https://fabio-a-oliveira.github.io/2020-07-30_Research_Funding.html)


<!---


<script src="https://cdn.jsdelivr.net/combine/npm/tone@14.7.58,npm/@magenta/music@1.21.0/es6/core.js,npm/focus-visible@5,npm/html-midi-player@1.1.1"></script>

original code:
<midi-player
  src="new_chorale__2021-03-08__12-26-40__120__.mid"
  sound-font visualizer="#myVisualizer">
</midi-player>

<midi-player
  src="test42.mid"
  sound-font = "https://storage.googleapis.com/magentadata/js/soundfonts/sgm_plus">
</midi-player>

<midi-player
  src="test43.mid"
  sound-font = "https://storage.googleapis.com/magentadata/js/soundfonts/sgm_plus">
</midi-player>

instrument 51
<midi-player
  src="test51.mid"
  sound-font = "https://storage.googleapis.com/magentadata/js/soundfonts/sgm_plus">
</midi-player>

instrument 54
<midi-player
  src="test54.mid"
  sound-font = "https://storage.googleapis.com/magentadata/js/soundfonts/sgm_plus">
</midi-player>

instrument 55
<midi-player
  src="test55.mid"
  sound-font = "https://storage.googleapis.com/magentadata/js/soundfonts/sgm_plus">
</midi-player>

instrument 71
<midi-player
  src="test71.mid"
  sound-font = "https://storage.googleapis.com/magentadata/js/soundfonts/sgm_plus">
</midi-player>

instrument 79
<midi-player
  src="test79.mid"
  sound-font = "https://storage.googleapis.com/magentadata/js/soundfonts/sgm_plus">
</midi-player>

instrument 94
<midi-player
  src="test94.mid"
  sound-font = "https://storage.googleapis.com/magentadata/js/soundfonts/sgm_plus">
</midi-player>

empty sound-font
<midi-player
  src="test94.mid"
  sound-font = "">
</midi-player>

--->
