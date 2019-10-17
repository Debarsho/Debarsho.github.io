---
layout: post
title: Interesting papers I - The Sentiment Neuron
published: true
---

An overview of the paper “[Learning to Generate Reviews and Discovering Sentiment](https://arxiv.org/abs/1704.01444)”, by researchers at OpenAI.
<!--break-->
**Since the authors have already written a wonderful [blog post about the paper](https://openai.com/blog/unsupervised-sentiment-neuron/), this post is not a complete overview of their work**. Rather, it serves to convey my thoughts and opinions about the paper.

## Paper Summary
The authors trained a character-level RNN language model (i.e., a model that outputs a probability distribution of which character might come after a given chunk of text) on a corpus of Amazon reviews, and evaluated its learned representation for sentiment analysis. Here, the “representation” of any textual document refers to a vector of dimension 4096 that comes from the penultimate layer of the RNN language model. The study reports that a linear model trained on such representations of the [Stanford Sentiment Treebank](https://nlp.stanford.edu/sentiment/treebank.html) (using the language model trained on the Amazon reviews) performs better than most other state-of-the-art models. While this is a notable result that showcases the potential of unsupervised representation learning, it is not the most interesting part of the paper.

  

Upon inspecting the representations, they found that one neuron contained most of the sentiment signal. Not only does this account for the excellent accuracy achieved by a linear model trained on such a representation, it means that the trained model could be used for much more than just representing a chunk of text.

  

The authors show that since it is known which neuron contains most of the sentiment signal in the neural language model, fixed-sentiment synthetic reviews could be generated from the language model. Text can be generated from any language model, but by manually fixing the value of the sentiment neuron (1 for positive sentiment and -1 for negative sentiment) and leaving everything else alone, reviews of varying sentiment could be generated. The comparison that the authors draw between positive and negative generated reviews (keeping all else constant except the value of the sentiment neuron) is worth taking a look at on their [blog post](https://openai.com/blog/unsupervised-sentiment-neuron/).

## Personal Opinions
Ever since learning of word embeddings while trying to grasp the basics of Deep Learning for NLP, I was intrigued by how Deep Learning could be used to learn representations of sentences. This paper feeds into this interest of mine, so it was an easy decision to read and review it. Although I found this paper intriguing and impactful when I first read it, its significance has diminished in my mind after reading it a few more times, for the following reasons:

### The study is restricted to amazon reviews. 
A critical review of anything is implicitly meant to convey the reviewer’s “sentiment”. Since the dataset used by the authors contains pieces of text whose sole purpose is to convey a reviewer’s feelings about a product, I feel that it should not come as such a surprise that the language model learned a representation with a neuron containing most of the sentiment signal. Similarly, it should not be a surprise that the model performed well on a popular sentiment analysis dataset. I would have liked to see an exploration of similar representations for different tasks on different datasets. For example, if the representations from the language model trained on Amazon reviews were shown to achieve good results for, say, categorizing news articles (or anything not analogous with Amazon reviews), I would be very impressed!

### The study is restricted to a character-level LSTM model.
From the first line of the paper’s abstract, the authors “explore the properties of byte-level recurrent language models”. I would like to have seen an exploration of representations learned from language models in general. Whether a similar sentiment neuron is present in a word-level RNN model or other architectures like Gated CNNs or Transformers, and whether anything can be inferred from their respective results on sentiment analysis tasks would make this paper more complete in my opinion. At the very least, the authors could have conducted similar experiments using a word-level RNN, which would have been trivial given that they already conducted the experiments for a character-level RNN.

## TL;DR
The authors show how a character-level RNN language model can learn a sentiment-disentangled representation that works well for sentiment analysis, but don’t generalize further than that. While the study is interesting, it is also disappointing due to the missed potential of expanding their study to cover other neural language model architectures and other applications of the representations learned by them.