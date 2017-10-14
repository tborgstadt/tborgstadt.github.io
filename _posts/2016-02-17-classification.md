---
layout: post
title: Predicting Salaries Greater than $50K
description: Apply and tune multiple classification models in search of "the best"...
tags: [classification, docker, neural networks, pybrain, theanets, rep, scikit-learn, ]
comments: True
---
This is an academic exercise in applying and tuning numerous classification models in order to predict individual salaries greater than $50K given demographics from a census dataset. We will consider classifiers from scikit-learn, pybrain, and theanets.

* TOC
{:toc}

#### Data Set

The [Adult Census](http://archive.ics.uci.edu/ml/datasets/Adult){:target="_blank"} dataset from UCI Machine Learning Repository was donated by Ronny Kohavi and Barry Becker of Data Mining and Visualization Silicon Graphics. The data consists of various demographic information collected by the US Census. The classification task is to predict whether an individual makes over $50K per year income. 
 
The dataset is provided in two sets, one for training and one for testing. For this project, the two sets were put back together for preprocessing and EDA before applying a 60/40 training/testing split. 
 
After preprocessing the total number of records was 48,841. The feature set grew from 15 to 109 after categorical features were converted to binary (True/False, 1/0). The dataset was scaled before split into training and testing sets.
 
This exercise is a two class classification, meaning either the person makes more than $50K per year, or they make $50K or less. The class distribution of the dataset is the following: 

* 24% - make greater than $50K  (value=1, class of interest is the minority class) 
* 76% - make $50K or less       (value=0) 
 
The dataset is imbalanced meaning there is significant difference between number of people making more than $50K and those that do not. Furthermore, the class of interest is the minority class which presents challenges in optimizing prediction of people making more than $50K. 

#### Environments

Two environments were utilized for experiements. The first environment was standard Anaconda running on Ubuntu 14.04. The Anaconda environment was used primarily for scikit-learn classifiers.

The second environment was installed using a Docker image and was utilized for the neural networks portion of the exercise. The Docker image consists of the [The Reproducible Experiment Platform (REP)](http://yandex.github.io/rep/index.html#){:target="_blank"}. I used the estimators module which contains wrappers that follow scikit-learn interface style for three neural network machine learning libraries - pybrain, neurolab, and theanets. The REP environment provides interoperatbilty of these neural network classifiers with scikit-learn boosting algorithms. For example, it is very easy to boost a pybrain classifier with scikit-learn adaboost algorithm.

#### Experiment Process
 
The process for tuning classifiers was iterative. Each classifier was run with default parameter values as a baseline and then tuned with grid search until the best estimator for each was identified. The result of each experiment (including grid search parms) was recorded in a csv file for reference and analysis for tuning the next step.

The following classification algorithms were used. Note number of experiments for each and total build time was captured.

<figure><a><img src="/images/class1.png"></a>
<figcaption></figcaption>
</figure>

#### Scikit-Learn Results

Below is ROC curves for top 5 performing estimators in the Anaconda environment. Based on ROC alone, Gradient Boost and Bagged Random Forest outperform the others.

<figure><a><img src="/images/class2.png"></a>
<figcaption></figcaption>
</figure>

#### Neural Nets Results

Below are ROC curves for experiments conducted in the REP environment. Gradient Boost was run in this environment to compare and agree with the same in the Anaconda environment. PyBrain and Theanet neural networks deserve addtional tuning with various network configurations, but clearly we can see differences in tuning parameters. For example, the Theanet­30­30 is a two hidden layer network with 30 neurons in each hidden layer. It has default learning parameter of .1. By comparison, the Theanet­30­30­.2 is the same network with .2 learning parm.

Note the **Boosted Theanet-30-30-.5** is example of how REP classes can accommodate boosting the Theanet classifier with scikit-learn AdaBoost boosting algorithm.

<figure><a><img src="/images/class3.png"></a>
<figcaption></figcaption>
</figure>

#### The Challenge and Conclusion

The challenge with this dataset is the fact that is imbalanced. The overall accuracy of the model is of secondary importance as we desire to predict the minority class. In other words, optimizing recall to improve predictibility of the minority class should be our objective.

The Bagged Linear SVC is able to best predict the minority class (those that make $50K or more) with 85 percent recall. It also has a good ROC score. The neural nets performed next best but were actually quite poor in predicting the minority class. The discrepency of the recall versus the cross validation of recall on the neural nets is an additional concern.

<figure><a><img src="/images/class4.png"></a>
<figcaption></figcaption>
</figure>

ROC Score Key: 

* .90­ to 1.0 = excellent (A) 
* .80­ to .90 = good (B) 
* .70­ to .80 = fair (C) 
* .60­ to .70 = poor (D) 
* .50­ to .60 = fail (F)

A Bagged Linear Support Vector is recommended because it provides the best predictability of the minority class as measured by recall.

#### Notebooks

* [Adult Census Scikit Learn Experiments](/notebooks/AdultCensus-Sklearn.html){:target="_blank"}
* [Adult Census Neural Net Experiments](/notebooks/AdultCensus-NN.html){:target="_blank"}

