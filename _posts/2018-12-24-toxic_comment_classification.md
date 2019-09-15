---
layout: post
title: Toxic Comments on Wikipedia - Data Exploration and Classification
published: true
---
How I went about analyzing Wikipedia comments and their toxicity. **The Jupyter notebook can be found [here](https://github.com/vamsi-aribandi/kaggle_toxic_comment_challenge/blob/master/kaggle_toxic_comment_challenge.ipynb).**
<!--break-->

## Background
This task was assigned to me to showcase my competency, and I had one week to finish it. More important than the end result was how I went about my data exploration, and what conclusions I could draw. The Jupyter Notebook linked above is self-explanatory, so hopefully this post will complement what is already written there.

## Introduction
The dataset used here is from [Kaggle](https://www.kaggle.com/c/jigsaw-toxic-comment-classification-challenge/data). I used [Google Colaboratory](https://colab.research.google.com/) to run my code, which has its own limitations, so from the beginning I ruled out any RNN-based solution (remember, I had a week to complete the entire pipeline). Even if I managed to gain confidence working with RNNs (say, an LSTM), training one on Colaboratory would have been a nightmare.

I decided to use this as an opportunity to familiarize myself with popular libraries like [scikit-learn](https://scikit-learn.org/stable/) and [pandas](https://pandas.pydata.org/). Although I had some experience with them before, I was still quite slow at using them, so this was a good opportunity to get better at implementing data science pipelines.

## First Attempt
After reading about word embeddings like [Word2Vec](https://ai.google/research/pubs/pub41224) and [gloVe](https://nlp.stanford.edu/projects/glove/), I was looking for an excuse to use them in some way. The question was, how could a sentence be represented using its words' embeddings? My first attempt was to simply represent a sentence as the average (i.e. length normalized sum) of its words' Word2vec embeddings, and see what happened when we applied a Logistic Regression Model to it. Not only were the results sub par, but it took so long to compute these sentence embeddings that it wasn't worth the time to optimize this approach. Querying the pre-trained model for each word of each of the 160k sentences was too much for Colaboratory to handle.

A better approach, as I had gathered online, might have been to compute a weighted average of the words' vectors, with the weight of a word being its TF-IDF score and its vector being its Word2vec representation. Think about how similar this is to simply training upon the TF-IDF representation of a sentence. The TF-IDF representation of a document can be seen as the vector sum of the TF-IDF score of a word multiplied with the word's vector representation, over all words in the document. The word's vector representation is the one-hot-encoded representation of the word (1 in the word's column, 0 in all other columns). If we replace this vector representation with one from a pre-trained model, we are essentially doing the same thing, but this vector representation actually has information about the meaning of the word ("boy" and "girl" vectors are quite close to each other in the space of the pre-trained model, but are orthogonal in the TF-IDF space).

Unfortunately I did not have the time to test whether I would get better results using the above sentence representation. The time it took for feature preparation using pre-trained models led me to the following final model.

## The Final Model
My final approach was to train a Logistic Regression Model upon the TF-IDF embeddings of the sentences, with some extra details like oversampling and more features. I was familiar with manually representing documents using the TF-IDF scheme from my Information Retrieval course, but I underestimated just how fast scikit-learn's implementation was (see: [TfidfVectorizer](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.TfidfVectorizer.html)). Feature creation was now a breeze, and the entire data science pipeline was surprisingly fast.

For more information on the final model (oversampling, more features), refer to the Jupyter Notebook linked above.

## Conclusions
Developing this data science pipeline taught me a lot. Scikit-learn is an incredibly powerful library and developers should probably check if it has a required feature before looking elsewhere. Imbalanced-learn is a good option to over/undersample your data. It was also my first time working with Jupyter Notebooks. Working with advanced scikit-learn features like Pipelining (see the Jupyter Notebook above for more details) was also a great learning experience.
