---
title: Sentiment Analysis with scikit-learn, nltk and word2vec
date: 2023-11-29T12:02:00+00:00
layout: single
permalink: /2023/11/sentiment-classification/
classes: wide
toc: true
---

## Introduction

This post presents a process of performing [sentiment analysis](https://en.wikipedia.org/wiki/Sentiment_analysis) the [_Starbucks Reviews Dataset_](https://www.kaggle.com/datasets/harshalhonde/starbucks-reviews-dataset) from [Kaggle](https://www.kaggle.com/). The goal was to classify reviews as either Positive or Negative. Two models were developed for this purpose, and their performances compared.

The accompanying code and data repository can be found [here](https://github.com/ma489/sentiment-classification-starbucks).

## Data

The ‘Starbucks Reviews Dataset’ contains data from online customer reviews of Starbucks coffee shop locations. The dataset contains 850 samples containing (among other features) a text review and a numeric rating (on a scale of 1-5).

First this data was cleaned up to remove incomplete samples: Only 705 of the 850 samples contained a numeric rating, and of those only 703 also contained an actual text review. This set of 703 reviews and ratings was used for modeling.

The distribution of the numeric ratings is given in Figure 1. We can see that the distribution of ratings is right-skewed (more negative ratings). A modelling choice was made to set a threshold of _3 and above_ to be a _Positive_ rating, and _below 3_ to be a _Negative_ rating - these would be the class labels. Figure 2 shows the resulting distribution of sentiment classes (Positive/Negative) - with a clear class imbalance.

The text reviews were then pre-processed, with the following steps applied to each document:

- Remove hyperlinks
- Make all words lower case
- Remove new line characters
- Remove characters that are not whitespace and not in English alphabet
- Remove words that are in the stopwords list provided by the [`nltk`](https://www.nltk.org/) package
- Remove the word ‘Starbucks’
- Perform stemming, using the `PorterStemmer` from [`nltk`](https://www.nltk.org/)

![Fig1](/assets/img/sentiment_classification_hists1-2.png){: .center-image }

## Modelling

The reviews dataset was used as follows:

- The text reviews were vectorized to create the independent variables
- The numeric ratings were converted to sentiment classes, these would be the class labels (the dependent variable ’Y’)

The reviews dataset was then split into training and test sets, with a 75%/25% split (with shuffling).

Two models were developed, both based on `RandomForestClassifier` from [scikit-learn](https://scikit-learn.org/stable/). A Random Forest Classifer was chosen as it is robust to datasets with class imbalance (as is the case here). Where the two models diverge, however, is in the choice of text vectorization method:
- Model 1 uses a `TfidfVectorizer`
- Model 2 uses a [`word2vec`](https://en.wikipedia.org/wiki/Word2vec) implementation from the [`gensim`](https://radimrehurek.com/gensim/) package.

Figure 3 shows a visualisation of one of the decision trees from Model 1, up to depth 5.

Note: when the models are applied to the test set, out-of-vocabulary terms are ignored.

![Fig3](/assets/img/sentiment_classification_model1.png){: .center-image }

## Results

The two models were applied to the test set and evaluated. Table 1 compares the models based on a number classification evaluation metrics: Accuracy, Precision, Recall, and f1-score. Macro averages are provided for those latter 3 metrics - since there is a class imbalance in the data, this should be taken into account when evaluating the models’ performance. The first model, `RandomForestClassifer` with `TfidfVectorizer`, performed better across all metrics. Figures 4 and 5 show the respective confusion matrices.

![Fig4-5](/assets/img/sentiment_classification_table1fig4-5.png){: .center-image }

---

Note: This post is based on a report written for an assignment as part of my degree in _Machine Learning & Data Science_ at [Imperial College London](https://www.imperial.ac.uk). The assignment was part of _Unstructured Data Analysis_ module taught by Dr A. Calissano (all credit due for the assignment setting, the goals and objectives of the modelling, and the selection and provision of the dataset).
