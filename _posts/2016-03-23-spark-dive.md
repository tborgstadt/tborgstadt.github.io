---
layout: post
title: Tune $50K Salary Estimators using Spark and Explore MLlib
description: A look at Spark powered Scikit Learn grid search and intro to Spark MLlib Tree classifiers
tags: [classification, docker, mllib, pyspark, scikit-learn, spark-sklearn]
comments: True
---
This is purely an academic project to begin exploring Spark. 

The objectives of the project:

1. Configure and use Jupyter Notebook with Spark
2. Use [databricks](https://databricks.com/){:target="_blank"} spark-sklearn package to boost SciKit Learn's grid search
3. Implement [Spark MLlib tree module](http://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#module-pyspark.mllib.tree){:target="_blank"} classifiers

* TOC
{:toc}

### Environment

Although I'm familiar with installing a Spark cluster, I opted to use a [Docker](https://www.docker.com/){:target="_blank"} stack [all-spark-notebook](https://github.com/jupyter/docker-stacks/tree/master/all-spark-notebook){:target="_blank"} made available by the Jupyter project for my local environment. This was my first time using [Docker](https://www.docker.com/){:target="_blank"}, and I must admit it is an impressive tool. I will be looking at it further.

For my Amazon environment, I opted for an EMR 5 node cluster with "all" apache products installed. I then installed anaconda, configured Jupyter, and installed the DataBricks package manually. 

I did have a problem with the databricks spark-sklearn package in the AWS environment. This needs additional troubleshooting so I have included only the Jupyter notebook from my local Docker environment at the end.

### Previous Work

In a previous project I evaluated performance of a number of different SciKit Learn classifiers on the following data set. The top two classifiers from that project will be used for basis of this project.

#### Data Set

The data set was the Adult Census dataset from the [UCI Machine Learning Repository](https://archive.ics.uci.edu/ml/datasets/Adult){:target="blank"}. My objective is to predict the minority class, or those people that make **more than $50K** annually based on a number of different demographic features from the census.

The dataset is imbalanced 3 to 1 in favor of the majority class, or those that **make $50K or less**.

#### Recommended Estimator

A Bagged Linear Support Vector was recommended because it provided the best predictability of the minority class as measured by recall. A Gradient Boosting Classifier was the second best but actually quite poor predictor of the minority class however, it is included here to see if it can be tuned further.

### Experiments and Results

For the experiments I performed, the Linear SVC and Gradient Boosting Classifer were extensively grid searched using the spark-sklearn package from [databricks](https://databricks.com/){:target="_blank"}.

<figure><a><img src="/images/sparkdive1.jpg"></a>
<figcaption></figcaption>
</figure>

First, a small improvement of the Bagged Linear Support Vector from the previous was made. This was accomplished with a small increase in the linear intercept that the additional grid search was able to find.

Second, I incorporated two conceptual feature selection algorithms for consideration. The first takes the top 60% of the most important features as identified by a random forest. The second, uses a K­means clustering classifier with 14 clusters in this case, and takes the top 5 features from each cluster to identify the top features. I did not get to bag these estimators over the smaller feature list, however, I suspect there are a number of correlated features considering the performance of these estimators is only slightly less than the boosted - both evident with the Linear Support Vector and Gradient Boost Classifiers.

I tried a manifold approach to feature selection by using SciKit Learn's Locally Linear Embedding (LLE) model but the module has a memory leak once several hundred observations are encountered. The code is included in the notebook for reference. Apparently the bug is a known issue.

Lastly, the [Spark MLlib tree module](http://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#module-pyspark.mllib.tree){:target="_blank"} classifiers are included. They performed quite well although none were tuned - all results reflect default parameters. The Random Forest did quite well and I'm anxious to spend some time tuning it further.

### Notebook

* [Exploring PySpark Notebook](/notebooks/Spark-Dive.html){:target="_blank"}.

That about wraps it up.

