---
title: Exploring Language in Classic Fiction with Data and R
date: 2023-04-19T12:01:00+00:00
layout: single
permalink: /2023/04/analysing-language-fiction/
classes: wide
toc: true
---

## Introduction

In this post, I will present an exploratory data analysis and visualisation of natural language data from a collection of 26 fiction books by six English-language authors. The data was downloaded from the [Project Gutenberg](https://www.gutenberg.org/) website [[1]](#references). For the purposes of this analysis, we treat each book as a document, and the collection of 26 books as a corpus. The text content of each book was pre-processed to remove irrelevant text added by Project Gutenberg.

The analysis is performed using [R](https://www.r-project.org/). As part of the analysis, I applied dimensionality reduction techniques such as [Principal Component Analysis](https://en.wikipedia.org/wiki/Principal_component_analysis) (PCA), [Multidimensional Scaling](https://en.wikipedia.org/wiki/Multidimensional_scaling) (MDS), and [t-SNE](https://en.wikipedia.org/wiki/T-distributed_stochastic_neighbor_embedding). I also used clustering methods such as [K-means](https://en.wikipedia.org/wiki/K-means_clustering) and [Hierarchical clustering](https://en.wikipedia.org/wiki/Hierarchical_clustering).

## Initial Exploratory Data Analysis

For an initial exploratory analysis of the data, a subset of the corpus containing 2 documents was selected (hence, the ’Corpus Subset’). The Corpus Subset comprises: _Great Expectations_, by Charles Dickens, and _The Sign of the Four_, by Sir Arthur Conan Doyle. Statistics of the Corpus Subset texts are presented in Table 1.

![Table1](/assets/img/analysing_language_table1.jpg){: .center-image }

The statistics in Table 1 give some insight into the writing style of the documents. For example, we see that Dickens’ sentences were longer than Doyle’s, containing 19.2% more words on average. Dickens’ book is also longer, containing 4 times as many words as Doyle’s. This difference in word count and sentence length possibly reflects the nature and era of the texts and their intended audiences: Dickens’ text is a relatively older novel that depicts and critiques the social and cultural landscape of 19th Century England (requiring more words and more complex sentences) whereas Doyle’s work is of the entertainment genre (detective mystery).

Figure 1 presents the top ten words in each Corpus Subset document based on tf-idf (the definition of tf-idf used is the one implemented by bind tf idf in tidytext [[2]](#references)). Pre-processing was performed to remove the ‘stop’ words (based on a list of stop words in the `tidytext` package [[2]](#references)). From Figure 1, we can quickly see the key characters and themes of each book. For example, ‘Holmes’ is the name of the main character in ‘The Sign of Four’, and ‘Joe’ is the name of one of the main characters in ‘Great Expectations’. 

![Figure1](/assets/img/analysing_language_fig1.png){: .center-image }

Similarly, Figure 2 shows the most common words in each of the subset documents, in the form of word clouds, and gives insight into the key characters in the documents.

![Figure2](/assets/img/analysing_language_fig2.png){: .center-image }

Figure 3 provides another view into the language used in this Corpus Subset: the bigraph shows common bigrams (based on frequency) and their interconnections. There is a large cluster at the top, centered on the words ‘miss’ and ‘dear’; possibly reflecting the formal language used in the books, both of which were published in the 19th century.

![Figure3](/assets/img/analysing_language_fig3.png){: .center-image }

## Similarity of Language

We now turn our attention to the full data set containing the corpus of all 26 books, which is analysed to assess the similarity of language used across the corpus. The data was first pre-processed in the following way:

1. Each document in the corpus is tokenized into bigrams
2. The bigrams are filtered to remove any bigrams that contain at least one stop word, producing p bigrams. In this corpus, p is \~145,000.
3. The [_tf-idf_](https://en.wikipedia.org/wiki/Tf%E2%80%93idf) of the remaining bigrams is calculated (using `bind_tf_idf` from `tidytext` [[2]](#references))
  * tf-idf is used here rather than raw term frequency count because we want to account for varying document lengths
4. A data matrix (**DataMatrix**) is constructed, converting the data from wide format to long format
  * This matrix has dimensions 26 × p. That is, 26 rows (one for each document), and p columns (one for each bigram). 
  * Each cell in the matrix contains the corresponding tf-idf value. The value in **DataMatrix<sub>i,j</sub>** is the tf-idf of bigram j in document i.

### Distance as Similarity

A distance matrix is then calculated from the **DataMatrix**, and visualised in Figure 4; it shows that document 13 in the corpus (_The Eyes Have It_, by Dick) is most distance to the rest, followed by document 23 (_The Missing Will_ by Christie). In this analysis, we consider distance in the tf-idf vector space as a signal of similarity of language - i.e. documents whose tf-idf vectors (represented in **DataMatrix**) that are close in that space use similar language.

![Figure4](/assets/img/analysing_language_fig4.png){: .center-image }

### Dimensionality Reduction

Next, Dimensionality Reduction is performed on the DataM atrix. Reduced dimensions are necessary for reducing the computational cost of the subsequent clustering operations, as well as facilitating visualisation. Dimensionality Reduction (down to two dimensions) is performed using three techniques (1) [Principal Component Analysis](https://en.wikipedia.org/wiki/Principal_component_analysis) (PCA), (2) [Multidimensional Scaling](https://en.wikipedia.org/wiki/Multidimensional_scaling) (MDS), and (3) PCA followed by [t-SNE](https://en.wikipedia.org/wiki/T-distributed_stochastic_neighbor_embedding); the results of which are visualised in Figures 5, 6, and 7 respectively. Importantly, these methods preserve ’closeness’ (points that are close in the higher-dimensional space are close in the lower-dimensional space).

Notes on the implementation: In method (1), the first two principal components were found to explain only 18% of the variance in the data. In method (3), PCA is performed first as t-SNE is computationally expensive, and using it directly is impractical on a dataset of this size. Finally, the two outlier points mentioned earlier - documents 13 and 23 - were removed to improve the visualisations by ’zooming in’ on the main cluster points.

![Figure5](/assets/img/analysing_language_fig5.png){: .center-image }

![Figure6](/assets/img/analysing_language_fig6.png){: .center-image }

![Figure7](/assets/img/analysing_language_fig7.png){: .center-image }

### Clustering

In order to group the documents by similarity of language, _k-means_ clustering was performed on the three reduced-dimension data sets. Figures 8, 9, and 10 show the resultant clusters (book titles are replaced by the authors’ initials in Figures 8 and 9, for better visualisation). In each case, the values for _K_ were selected using a combnination the [elbow method](https://en.wikipedia.org/wiki/Elbow_method_(clustering)) (using `fviz_nbclust` from the `factoextra` R package) and visual inspection of the resulting clusters. We can see that the third approach in Figure 11 (PCA followed by t-SNE followed by K-Means) results in the clearest separation of clusters; we see some of the works by the same author are grouped to together, e.g. three of Dickens’ works appear together in the middle red cluster, and two of Austen’s works appear together in the pink cluster (bottom left).

![Figure8](/assets/img/analysing_language_fig8.png){: .center-image }

![Figure9](/assets/img/analysing_language_fig9.png){: .center-image }

![Figure10](/assets/img/analysing_language_fig10.png){: .center-image }

Hierarchical Clustering was also performed on the documents, in the high-dimensional space. Figure 12 shows the resultant dendrogram. Towards the left of the diagram, we see a cluster for The Eyes Have It (document 13, the outlier, from earlier), suggesting this book’s language is quite different from the other books. Towards the right we see clusters for Doyle’s books, The Sign of the Four and A Study in Scarlet.

![Figure11](/assets/img/analysing_language_fig11.png){: .center-image }

## Conclusion

In conclusion, this report has demonstrated that a number of techniques can be applied to analyse and visualise unstructured textual data and identify patterns. While _some_ patterns do emerge when clustering (i.e. subsets of the same author’s works appearing together), in all methods there are limitations, as shown by the lack of intuitively-understood clusters. 

Prior to performing clustering, this report’s author had some expectations of the number of clusters that would possibly emerge in this corpus: perhaps six clusters reflecting the six authors, or three clusters reflecting the three genres that might broadly categorise the corpus (_Detective Mystery_ - Christie and Doyle, _Sci-Fi_ - Wells and Dick, _Victorian Life_ - Dickens and Austen), or clusters around publication dates (intuitively: language changes over time). Further analysis and exploration should pursue uncovering these deeper patterns.

---

## References

1. Project Gutenberg. (n.d.). Retrieved April 2023, from [www.gutenberg.org](https://www.gutenberg.org/).
2. Silge, J., & Robinson, D. (2016). [Tidytext: Text mining and analysis using tidy data principles](https://joss.theoj.org/papers/10.21105/joss.00037). _R. Journal of Open Source Software, 1(3), 37._

---

Note: This post is based on a report written for an assignment as part of my degree in _Machine Learning & Data Science_ at [Imperial College London](https://www.imperial.ac.uk). The assignment was part of _Exploratory Data Analysis & Visualisation_ module taught by the inspiring, knowledgable and supportive Dr. J Martin (all credit due for the assignment setting, the goals and objectives of the analysis, and the selection and provision of the dataset).