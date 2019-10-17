---
layout: post
title: Interesting Papers II - Exploring Contextual Vectors
published: true
---

An overview of the paper “[Dissecting Contextual Word Embeddings: Architecture and Representation](https://aclweb.org/anthology/D18-1179/)”, presented at EMNLP 2018 by researchers at Allen AI and the University of Washington.
<!--break-->
All images and tables in this post are from their paper.

## Motivation
Contextual word representations from pre-trained bidirectional language models (biLMs) have recently shown to improve performance across various NLP tasks compared to word embeddings. However, the kind of patterns learned by biLMs and why they were so effective at representing words contextually had not been studied yet.

\[\sum_{k=1}^N \{log(p(t_k | t_1, t_2, ... t_{k-1})) + log(p(t_k | t_{k+1}, t_{k+2}, ... t_{N}))\}\]

The authors aim to study contextual representations learned by neural biLMs (independent of their architecture) to understand the patterns they learn, whether they contain hierarchies of features, and how effective they are at learning the structure of language.

## Bidirectional language models

biLMs model the probability distribution of what word would be present in a blank given the sequence of words preceding and following the blank. To put it formally, they maximize the sum of log-likelihoods of language models (LMs) in forward and backward directions:

Neural biLMs/LMs must use a context-insensitive word representation for their first layer (in this case, a [character-to-word encoder](https://arxiv.org/abs/1505.00387) and a Softmax layer as their final layer (so that the model outputs a probability distribution). In between, each hidden layer of state-of-the-art biLMs is the concatenation of the forward and backward LMs’ respective hidden states. The word embedding layer is fully connected to its next layer and the Softmax layer is fully connected to its preceding layer. However, in between, each hidden layer has separate weights for the forward and backward LM respectively. In other words, the forward LM updates half of the hidden neurons’ weights and the backward LM updates the other half during their respective backpropagation passes, but both LMs update the embedding and Softmax layers’ weights.

### Forming a Contextual Representation 
Once a biLM is trained, any of its hidden layers can be used as a contextual representation of a word. However, to better leverage the representations of every hidden layer (since each layer generally learns patterns at a different level of abstraction), [ELMo](https://arxiv.org/abs/1802.05365) was introduced. ELMo considers a word’s representation to be the weighted average of all of its hidden layer representations, and the weights are parameters to be optimized for a specific task.

### Different biLM architectures
The authors considered 3 different biLM architectures: LSTMs, gated CNNs, and Transformers.

## Evaluation as Word Representations
The authors compared the quality of biLM representations as ELMo-like contextual vectors with context-insensitive GloVe embeddings across NLP tasks. Natural language inference ([The MultiNLI dataset](https://www.nyu.edu/projects/bowman/multinli/)), semantic role labeling ([The OntoNotes 5.0 Dataset](https://catalog.ldc.upenn.edu/LDC2013T19)), constituency parsing ([The Penn Treebank](https://catalog.ldc.upenn.edu/LDC99T42)), and named entity recognition (NER) ([The CoNLL 2003 NER task](https://www.clips.uantwerpen.be/conll2003/ner/)) are all well-known benchmark NLP tasks, and the authors swapped out GloVe embeddings with biLM contextual embeddings the tasks’ respective state-of-the-art models. They found that the contextual embeddings consistently outperformed the GloVe embeddings on every task, regardless of biLM architecture.
![Model performance across benchmarks tasks using different embeddings](https://raw.githubusercontent.com/vamsi-aribandi/vamsi-aribandi.github.io/master/images/IP_II/benchmarks_table.png)

## Exploring Properties of Contextual Vectors
Coming to the most interesting part of the paper, the authors examine patterns learned by biLMs, and focus on those that are independent of architecture. Interestingly, they find that biLMs learn a hierarchy of features ([like how CNNs do so for images](https://arxiv.org/abs/1311.2901)) which varies with network depth - morphological at the embedding level, local syntax at lower levels, and semantic relationships (like coreference) at upper layers.

### Contextual Similarity
The authors find that using nearest neighbors with cosine similarity as a distance metric, it can be observed that lower layers capture mostly local information while top layers capture longer range relationships between words. Intra-sentence similarity can be observed between pairs of words in a sentence. For example, using lower-layer vectors, “the Russian government” has its words clustered together; and in the higher layers, all verbs have high similarity among each other suggesting that the biLMs capture part-of-speech information.

Since the context vectors abruptly change at syntactic boundaries of sentences (the words of phrases generally cluster together as stated above), a good representation of a span (a sequence of tokens) might also be possible to formulate. The authors concatenate the hidden states of the first and last context vectors of a span with their difference and dot products to form such a representation. Upon visualizing such span representations, they are clustered by chunk-type (verb-phrase, adjective-phrase, etc.), so it can be inferred that the span representations capture elements of syntax.

The authors also hypothesize that using these contextual vectors, unsupervised coreference resolution should be possible, as contextual representations of coreferential mentions should be similar. They show that simple arithmetic manipulations of contextual vectors perform comparably to the upper-bound of the coreference resolution task (52%-57% and 64% respectively).

### Context Independent Word Representation
Traditional word embeddings (like GloVe) are good at representing semantic relationships like “Moscow:Russia :: Beijing:China”, as well as syntactic relationships like “apparent:apparently :: slow:slowly”. However, the authors make the interesting observation that although the word embedding layer of biLMs capture syntactic relationships better than context-independant word embeddings, they are remarkably worse when it comes to semantic relationships.
![How different embeddings perform on semantic and syntactic relation matching](https://raw.githubusercontent.com/vamsi-aribandi/vamsi-aribandi.github.io/master/images/IP_II/semantic_syntactic_relations.png)

### Understanding Contextual Information
The authors examine their models results in parts-of-speech (POS) tagging (as a subset of the NER benchmark task) and Constituency Parsing more closely, and find that the results support the claim that all layers of the biLM learn syntax. Interestingly, the layers that perform well for Constituency parsing are generally at or above those that perform well for POS tagging, as can be seen in the below image. This is an important observation to make, since it supports the claim that biLMs learn a hierarchy of features. Put simply, it means that lower levels learn local syntax required for POS tagging; and higher levels learn patterns across wider context required for constituency parsing. Similarly, even higher layers perform well for coreference resolution, further supporting the claim.

![Layer performance for tasks](https://raw.githubusercontent.com/vamsi-aribandi/vamsi-aribandi.github.io/master/images/IP_II/benchmarks_layers_performance.png)

The distribution of layers’ weights in the ELMo word embeddings (specific to each task) is evidence that further supports the above claim. As expected, NER task weights are skewed towards lower levels. The authors note that the middle layers are the most transferable, with higher levels being more specific for language modeling.

![Layer distribution across tasks for ELMo embeddings](https://raw.githubusercontent.com/vamsi-aribandi/vamsi-aribandi.github.io/master/images/IP_II/benchmarks_layers_distribution.png)

## Personal Opinions
As of writing this article, I have yet to come across a more interesting NLP paper. Maybe I am biased because it aligns closely with my research interests (as of now), but I greatly appreciate the motivation behind the study and the way the authors explained the results. The parts of the paper that I appreciate the most are those that illustrate how different layers of biLMs perform differently across NLP benchmark tasks and relate that to the hierarchy of features learned by biLMs.