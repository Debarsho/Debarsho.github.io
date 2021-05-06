---
layout: post
title: Brain Decoding - A Transductive Ensemble Learning approach
published: true
---
An approach to decoding MEG signals of the brain, involving stacked generalization and covariate shift. This project was done by Skanda Vaidyanath, Sachin Panneer Selvam, and I. The code can be found [here](https://github.com/vamsi-aribandi/brain_decoding_kaggle).
<!--break-->

## Introduction

The human brain is something the scientific community has tried to understand for as long as anyone can remember. Although modern data like EEG and MEG signals give us some insight into the activity of one's neurons, until now they have been superficial at best when it comes to trying to understand what someone is thinking about. Could the power of machine learning help us understand someone's thoughts, and "decode" their brain? Nature has a great introduction to the concept [here](https://www.nature.com/news/brain-decoding-reading-minds-1.13989), which includes some interesting studies from UC Berkeley. In this project,  we try and analyze MEG signal data to take a small step in the direction towards understanding one's thoughts from sensor data. Our methodologies are inspired by Olivetti et. al.'s [work](https://ieeexplore.ieee.org/document/6858538).

## The Dataset
The dataset is from Kaggle's DecMeg2014 challenge, and the official train data consists of 16 participants’ trials. The official test dataset’s labels are unavailable, so they have been ignored for this project. Each participant was shown around 600 images, and an image could either have been a clear human face or a scrambled, blurry face. One trial (the act of showing a participant an image) lasts 1.5 seconds or 375 timesteps, and each timestep consists of readings from 306 MEG sensors across the surface of the participant’s head. Out of the 1.5s, the first 0.5s is a gap between images where nothing is shown, and the remaining 1 second is the duration that the image is shown to the participant. More details about the dataset can be read about [here](https://www.kaggle.com/c/decoding-the-human-brain/data). One trial can be thought of as a 375x306 matrix of values. The task is simply a binary classification problem, the task being to tell whether a participant has just seen a scrambled face or a clear human face.

## Intuition behind the approach
When it comes to how one's brain signals vary with respect to how sensors are placed around their head, each person's readings differ significantly from each other's. In statistical terms, each person's brain signals follow a different distribution. This is a problem, as machine learning models perform poorly when they are trained on data points sampled from a certain distribution but tested on those sampled from another distribution. For example, If a binary (is/is not) cat classifier was trained on a set where all positive instances were black cats, but tested on one where most positive instances were gold cats, there is a low chance the classifier would've had acceptable test performance. A practical obstacle was identified here - how can we truly understand humans' brains, when each brain follows a different distribution of signal passing? It's not like we can gather MEG signals from everyone on earth! The best we can do is try and keep this fact in mind while going about building a classifier. This leads to an approach utilizing transductive ensemble learning.

Transductive Learning, as opposed to inductive learning, is the idea that if we want to maximize performance on a test set, then why aim to build a generalized classifier (an inductive learner)? Why not instead build a classifier that excels in the domain of the test set? For the cat example, this would be tantamount to trying to build a classifier that is good at recognizing gold cats while training upon black cats.

Ensemble learning is far simpler. Fundamentally, it is just using multiple learners instead of one to approximate whatever function is to be learned. A simple example would be to train a logistic regression model, a random forest, and an SVM, and then predict whatever class the majority of the three classifiers predict.

## The Ensemble part - Stacked Generalization
Stacked Generalization (or stacking) involves training a model upon the outputs of various other learners. Here, our intention was to take advantage of the fact that if only one person's data was considered, individual classifiers performed well. Since the models efficiently learned the distributions of individuals' signals, why not use each individual's model to classify a new individual's data? Simply put, we trained a separate classifier upon each participant's data and built a dataset where the features were the outputs of the 16 participants' classifiers.

Stacked generalization made sense because incorporating more classifiers with good performance upon their respective distributions should make the final meta-learner more robust. There is scope for patterns to be learned from many individual participant's data, while also avoiding the ugly prospect of training a classifier upon data consisting of data points drawn from multiple samples. For example, if participant X's data generally aligns with that of a large portion of people, the meta-learner will learn to give more importance (a weight with larger magnitude) to A's classifier's predictions. If participant Y's data is unique enough that it doesn't really align with many other's data, the meta-learner would assign Y's classifier's predictions less importance.

## The Transductive part - Covariate Shift
Covariate shift is officially defined as the phenomenon that occurs when the distributions of inputs (queries) change but the conditional distribution of outputs (answers) is unchanged. This essentially means that the distribution of the training data is different than the test data. That means what we learn from the training set will not be very useful to predict information about the test set. While Stacked generalization helped us build a single meta-learner for the task, any test dataset will almost certainly follow a different distribution than any train participant's data, which ensemble learning has not addressed. The methods used to solve this particular problem are called covariate shift methods. Covariate shift methods allow us to take advantage of the fact that although we can't use the labels of the test dataset to help our classifier, we could still try and model the distribution from which the test data points were sampled. The idea behind it is to use information in the test dataset (not the labels, that's cheating!) to build a model that will perform better on the test dataset. One particular method is based on the idea of sample reweighting. The method reweights the samples in the training data according to how likely it is to be found in the test data. To rephrase, it gives every training example a weight corresponding to how similar it is to data points in the test dataset, i.e. how likely it is to be drawn from the test dataset's distribution.

To use the cat example we spoke about earlier, if we are trying to predict something about brown cats while training on black cats, we weight the cats in the training set that are brownish higher than the other cats.

## Experiments and Results
The first step was to build a classifier for each of the separate patients. We took the 375x306 matrix and flattened it and passed it through a Logistic Regression Model. We also figured that the first 0.5s of the data did not make a big difference to the classification task since it was the gap in between showing images, and hence tried to remove the first 0.5s in an effort to reduce the dimensionality and the complexity of the data. The accuracy of the individual models were as follows.

| Patient | Accuracy w/ first 0.5s | Accuracy w/out first 0.5s |
| :-: | :-: | :-: |
| 1 | 0.80 | 0.81 |
| 2 | 0.76 | 0.78 |
| 3 | 0.78 | 0.79 |
| 4 | 0.83 | 0.82 |
| 5 | 0.79 | 0.78 |
| 6 | 0.82 | 0.80 |
| 7 | 0.83 | 0.82 |
| 8 | 0.86 | 0.85 |
| 9 | 0.75 | 0.80 |
| 10 | 0.74 | 0.78 |
| 11 | 0.70 | 0.71 |
| 12 | 0.76 | 0.78 |
| 13 | 0.86 | 0.86 |
| 14 | 0.89 | 0.89 |
| 15 | 0.90 | 0.90 |
| 16 | 0.86 | 0.86 |

The average accuracy for the 16 models before we dropped the first 0.5s was 80.8125% and the average accuracy after we dropped the first 0.5s was 81.4375%. So there is a slight improvement.

The accuracy of the individual models are quite good, but the aim is to build a single model that works for all the patients. This is where we try Stacked Generalization. We build a new dataset to build a new model for all patients combined. This new dataset contains the data points for all patients and all instances but the features are the outputs from the individual classifiers we built earlier. For example, for patient 1, for each data point, we have the outputs of classifiers 2-16 and for classifier 1, we have a NaN. We fill in the NaNs with column means later. We then use this dataset to try and predict the results for the combined dataset. We tried Stacked Generalization with a Voting Classifier (Logistic Regression, SVM and Random Forest). Although the motivation for using the Voting Classifier was that each algorithm would learn different patterns, the performance of it was not significantly better than just using Logistic Regression. For each patient, the model was trained on the remaining patients in the dataset and then tested considering the patient dataset as a test set.

| Patient | Accuracy w/ first 0.5s | Accuracy w/out first 0.5s |
| :-: | :-: | :-: |
| 1 | 0.65 | 0.67 |
| 2 | 0.65 | 0.64 |
| 3 | 0.60 | 0.59 |
| 4 | 0.66 | 0.64 |
| 5 | 0.66 | 0.66 |
| 6 | 0.58 | 0.58 |
| 7 | 0.58 | 0.59 |
| 8 | 0.64 | 0.66 |
| 9 | 0.67 | 0.66 |
| 10 | 0.63 | 0.62 |
| 11 | 0.64 | 0.61 |
| 12 | 0.62 | 0.65 |
| 13 | 0.58 | 0.60 |
| 14 | 0.66 | 0.66 |
| 15 | 0.65 | 0.66 |
| 16 | 0.56 | 0.56 |

The results for the combined dataset are, as expected, not as good as the individual models. This is because the distributions of the patients are so different and it is difficult to learn a single distribution that works for all the patients. But ideally, covariate shift would alleviate this problem. It tries to learn on distribution A and tries to make predictions on distribution B. The covariate shift technique has already been explained in detail above so we will just present the results obtained from covariate shift and stacked generalization here.

| Patient | Accuracy w/ first 0.5s | Accuracy w/out first 0.5s |
| :-: | :-: | :-: |
| 1 | 0.65 | 0.67 |
| 2 | 0.64 | 0.65 |
| 3 | 0.60 | 0.60 |
| 4 | 0.64 | 0.63 |
| 5 | 0.66 | 0.67 |
| 6 | 0.59 | 0.58 |
| 7 | 0.58 | 0.61 |
| 8 | 0.64 | 0.66 |
| 9 | 0.66 | 0.67 |
| 10 | 0.63 | 0.63 |
| 11 | 0.63 | 0.62 |
| 12 | 0.63 | 0.63 |
| 13 | 0.59 | 0.61 |
| 14 | 0.66 | 0.65 |
| 15 | 0.64 | 0.63 |
| 16 | 0.57 | 0.56 |

The mean accuracy for just stacked generalization without dropping the first 0.5s was 62.6875% and after dropping the first 0.5s was 62.8125%. The mean accuracy for the stacked generalization and covariate shift experiment before and after dropping the first 0.5s were 62.5625% and 62.9375% respectively. The numbers are more or less similar in both cases and surprisingly it seems like covariate shift is not making a huge difference. This was a completely unexpected result and we definitely expected covariate shift techniques to help us improve our accuracy but this was not the case. Our intuition is that the dimensionality of the flattened data is simply too high for simple models like Logistic Regression to learn effective representations in order for covariate shift to effectively distinguish between different individuals' signals.

## Conclusions and next steps
While the intuitive approach of transductive learning did not fare well, we believe there is potential for modern signal processing techniques tied with deep learning algorithms like RNNs and CNNs to learn better representations of these MEG signals. Stay tuned for part two of this project!
