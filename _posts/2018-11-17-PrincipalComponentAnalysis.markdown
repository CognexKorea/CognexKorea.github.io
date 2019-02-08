---
layout: post
title:  "Deep Learning Theory / Principal Component Analysis(PCA)"
date:   2018-11-17 10:30:00
author: Alex Choi
categories: Deep-Learning
---
Hi GS&S Guys!
In this post I'm going to introduce Princial Component Analaysis (PCA), one of the most important dimensionality reduction methods.

------
## Introduction
I've looked for a bunch of learning resources on dimensionality reduction techniques. I have come across a couple resources about dimensionality reduction techniques. This topic is definitively one of the most interesting ones, it is great to think that there are algorithms that are able to reduce the number of features by choosing the most important ones that still represent the entire dataset. One of the advantages pointed out by authors is that these algorithms can improve the results of classification task.

In this post, I am going to verify this statement using a Principal Component Analysis ( PCA ) to try to improve the classification performance of a neural network over a dataset. Does PCA really improve the outcome of classification? Let’s check it out.

------
## Dimensionality Reduction Algorithms
Before go straight ahead to code, let’s talk about dimensionality reduction algorithms.

Somewhat unsurprisingly, reducing the dimension of the feature space is called “dimensionality reduction.” There are many ways to achieve dimensionality reduction, but most of these techniques fall into one of two classes:

* Feature Elimination
* Feature Extraction

**Feature elimination** is what it sounds like: we reduce the feature space by eliminating features. Advantages of feature elimination methods include simplicity and maintaining interpretability of your variables. As a disadvantage, though, you gain no information from those variables you’ve dropped.

**Feature extraction**, however, doesn’t run into this problem. Say we have ten independent variables. In feature extraction, we create ten “new” independent variables, where each “new” independent variable is a combination of each of the ten “old” independent variables. However, we create these new independent variables in a specific way and order these new variables by how well they predict our dependent variable. You might say, “Where does the dimensionality reduction come into play?” Well, we keep as many of the new independent variables as we want, but we drop the “least important ones.” Because we ordered the new variables by how well they predict our dependent variable, we know which variable is the most important and least important. But — and here’s the kicker — because these new independent variables are combinations of our old ones, we’re still keeping the most valuable parts of our old variables, even when we drop one or more of these “new” variables!

There are two principal algorithms for dimensionality reduction: [Linear Discriminant Analysis](https://en.wikipedia.org/wiki/Linear_discriminant_analysis) (LDA) and [Principal Component Analysis](https://en.wikipedia.org/wiki/Principal_component_analysis) (PCA). The basic difference between these two is that <U>LDA uses information of classes</U> to find new features in order to maximize its separability while <U>PCA uses the variance of each feature</U> to do the same. In this context, LDA can be considered a supervised algorithm and PCA an unsupervised algorithm.

------
## What is PCA?
The idea behind PCA is simply to find a low-dimension set of axes that summarize data. Why do we need to summarize data though? Let’s think about this example: We have a dataset composed by a set of properties from cars. These properties describe each car by its size, color, circularity, compactness, radius, number of seats, number of doors, size of trunk and so on. However, many of these features will measure related properties and so will be redundant. Therefore, we should remove these redundancy and describe each car with less properties. This is exactly what PCA aims to do. For example, think about the number of wheel as a feature of cars and buses, almost every example from both classes have four wheels, hence we can tell that this feature has a low variance (from four up to six wheels or more in some of rare buses), so this feature will make bus and cars look the same, but they are actually pretty different from each other. Now, consider the height as a feature, cars and buses have different values for it, the variance has a great range from the lowest car up to the highest bus. Clearly, the height of these vehicle is a good property to separate them. Recall that PCA does not take information of classes into account, it just look at the variance of each feature because is reasonable assumes that features that present high variance are more likely to have a good split between classes.

Often, people end up making a mistake in thinking that PCA selects some features out of the dataset and discards others. The algorithm actually constructs new set of properties based on combination of the old ones. Mathematically speaking, PCA performs a linear transformation moving the original set of features to a new space composed by principal component. These new features does not have any real meaning for us, only algebraic, therefore do not think that combining linearly features, you will find new features that you have never thought that it could exist. Many of people still believe that machine learning algorithms are magic, they put a thousands of inputs straight to the algorithm and hope to find all insights and solutions for theirs business. Do not be tricked. It is the job of the data scientist to locate the insights to the business through a well-conducted exploratory analysis of data using machine learning algorithm as a set of tools and not as a magic wand. Keep it in mind.

------
## When Should I Use PCA?
1. Do you want to reduce the number of variables, but aren’t able to identify variables to completely remove from consideration?
2. Do you want to ensure your variables are independent of one another?
3. Are you comfortable making your independent variables less interpretable?

If you answered “yes” to all three questions, then PCA is a good method to use. If you answered “no” to question 3, you should not use PCA.

------
## How Does the Principal Component Space Looks Like?
In the new feature space we are looking for some properties that strongly differ across the classes. Some properties that present low variance are not useful, it will make the examples look the same. On the other hand, PCA looks for properties that show as much variation across classes as possible to build the principal component space. The algorithm use the concepts of variance matrix, covariance matrix, eigenvector and eigenvalues pairs to perform PCA, providing a set of eigenvectors and its respectively eigenvalues as a result. How exactly PCA performs is a material for next posts.

So, what should we do with eigenvalues and eigenvectors? It is very simple, the eigenvectors represents the new set of axes of the principal component space and the eigenvalues carry the information of quantity of variance that each eigenvector have. So, in order to reduce the dimension of the dataset we are going to choose those eigenvectors that have more variance and discard those with less variance. As we go though the example below, it will be more and more clear how exactly it works.

Thus the algorithm is summarized as below:

1. Calculate the covariance matrix of $$ \mathbf{X} $$ of data points.
2. Calculate eigen vectors and corresponding eigen values.
3. Sort the eigen vectors according to their eigen values in decreasing order.
4. Choose first $$ k $$ eigen vectors and that will be the new $$ k $$ dimensions.
5. Transform the original $$ n $$ dimensional data points into $$ k $$ dimensions.

The covariance matrix of $$ \mathbf{X} $$ is calculated:

$$ \mathbf{C_{X}} = \frac{1}{n-1} (\mathbf{X} - \mathbf{\bar{X}})(\mathbf{X} - \mathbf{\bar{X}})^{T} $$

where $$ n $$ is the number of samples and $$ \mathbf{\bar{X}} $$ is mean vector.

It's so simple, isn't it?

------
## Detailed Algorithm
However, we need to understand the algorithm under the hood to code in detail. So for readers who want to just understand roughly, skip this part.

Before starting, you should have tabular data organized with $$ n $$ rows and $$ p + 1 $$ columns, where there is one column corresponding to your dependent variable (usually denoted $$ \mathbf{Y} $$) and $$ p $$ columns corresponding to each of your independent variables (the matrix of which is usually denoted $$ \mathbf{X} $$).

1. Separate your data into $$ \mathbf{Y} $$ and $$ \mathbf{X} $$, as defined above — we’ll mostly be working with $$ \mathbf{X} $$.<br/>
2. Take the matrix of independent variables $$ \mathbf{X} $$ and, for each column, subtract the mean of that column from each entry (This ensures that each column has a mean of zero). <br/><br/> $$ (\mathbf{X} - \bar{\mathbf{X}}) $$ <br/><br/>
3. Decide whether or not to standardize. Given the columns of $$ \mathbf{X} $$, are features with higher variance more important than features with lower variance, or is the importance of features independent of the variance? (In this case, importance means how well that feature predicts $$ \mathbf{Y} $$.) If the importance of features is independent of the variance of the features, then divide each observation in a column by that column’s standard deviation. (This, combined with step 2, standardizes each column of $$ \mathbf{X} $$ to make sure each column has mean zero and standard deviation 1.) Call the centered (and possibly standardized) matrix $$ \mathbf{Z} $$.<br/>
4. Take the matrix $$ \mathbf{Z} $$, transpose it, and multiply the transposed matrix by $$ \mathbf{Z} $$. (Writing this out mathematically, we would write this as $$ \mathbf{Z^{T}Z} $$.) The resulting matrix is the covariance matrix of $$ \mathbf{Z} $$, up to a constant.<br/>
5. Calculate the eigenvectors and their corresponding eigenvalues of $$ \mathbf{Z^{T}Z} $$. This is quite easily done in most computing packages — in fact, the eigendecomposition of $$ \mathbf{Z^{T}Z} $$ is where we decompose $$ \mathbf{Z^{T}Z} $$ into $$ \mathbf{PDP^{-1}} $$, where $$ \mathbf{P} $$ is the matrix of eigenvectors and $$ \mathbf{D} $$ is the diagonal matrix with eigenvalues on the diagonal and values of zero everywhere else. The eigenvalues on the diagonal of $$ \mathbf{D} $$ will be associated with the corresponding column in $$ \mathbf{P} $$ — that is, the first element of $$ \mathbf{D} $$ is $$ \lambda_1 $$ and the corresponding eigenvector is the first column of $$ \mathbf{P} $$. This holds for all elements in $$ \mathbf{D} $$ and their corresponding eigenvectors in $$ \mathbf{P} $$. We will always be able to calculate $$ \mathbf{PDP^{-1}} $$ in this fashion. (Bonus: for those interested, we can always calculate $$ \mathbf{PDP^{-1}} $$ in this fashion because $$ \mathbf{Z^{T}Z} $$ is a symmetric, positive semidefinite matrix).<br/>
6. Take the eigenvalues $$ \lambda_{1} $$, $$ \lambda_{2} $$ , ... , $$ \lambda_{p} $$ and sort them from largest to smallest. In doing so, sort the eigenvectors in $$ \mathbf{P} $$ accordingly. (For example, if $$ \lambda_{2} $$ is the largest eigenvalue, then take the second column of $$ \mathbf{P} $$ and place it in the first column position.) Depending on the computing package, this may be done automatically. Call this sorted matrix of eigenvectors $$ \mathbf{P^{*}} $$. (The columns of $$ \mathbf{P^{*}} $$ should be the same as the columns of $$ \mathbf{P} $$, but perhaps in a different order.) Note that these eigenvectors are independent of one another.<br/>
7. Calculate $$ \mathbf{Z^{*}} = \mathbf{ZP^{*}} $$. This new matrix, $$ \mathbf{Z^{*}} $$, is a centered/standardized version of $$ \mathbf{X} $$ but now each observation is a combination of the original variables, where the weights are determined by the eigenvector. As a bonus, because our eigenvectors in $$ \mathbf{P^{*}} $$ are independent of one another, each column of $$ \mathbf{Z^{*}} $$ is also independent of one another.

------
## Example with Python Code
Let's understand PCA by a Python code example.

<br/>
#### Prerequisites
* Python: ver.3.x
* [Pandas Library](https://pandas.pydata.org/)
* [Scikit-learn Library](https://scikit-learn.org/)
* [Matplotlib Library](https://matplotlib.org/)

<br/>
#### Download Data Files
Click the links below to download csv data files, which should be under `dataset` path.

* Download [movies.csv](../../../../assets/posts/2018-11-17-PrincipalComponentAnalysis/dataset/movies.csv)
* Download [ratings.csv](../../../../assets/posts/2018-11-17-PrincipalComponentAnalysis/dataset/ratings.csv)

`movies.csv` is a movies dataset which has three features(columns), `movieId`, `title` and `genres`.

<img src="{{ site.baseurl }}/assets/posts/2018-11-17-PrincipalComponentAnalysis/01.png">

`ratings.csv` is a users' movie rating dataset which has four features - `userId`, `movieId`, `rating` and `timestap`.

<img src="{{ site.baseurl }}/assets/posts/2018-11-17-PrincipalComponentAnalysis/02.png">

<br/>
#### Import Libraries
Let's import necessary Python libraries.

{% highlight python %}
# Load dependencies
import pandas as pd
import numpy as np
from sklearn.preprocessing import StandardScaler
import matplotlib.pyplot as plt
from sklearn.decomposition import PCA as sklearnPCA
{% endhighlight %}

<br/>
#### Step 1 : Load and Standardize Data
`read_csv()` in Pandas library is called to read `.csv` files into Pandas Dataframe `movies` and `ratings` and then `timestamp` ferature(column) is removed from `ratings` because it is not necessary in this analysis.

{% highlight python %}
###############################################################################
# Step 1: Load and Standardize Data
###############################################################################

# Load movie names and movie ratings
movies = pd.read_csv('./dataset/movies.csv')
ratings = pd.read_csv('./dataset/ratings.csv')
ratings.drop(['timestamp'], axis=1, inplace=True)
{% endhighlight %}

Pandas' `Series.map()` function is very useful for replacing from one column to another column. This function is used here to replace from `movieId` to `title`.

{% highlight python %}
ratings.movieId = ratings.movieId.map(replace_name)
{% endhighlight %}

The argunment `replace_name` is a function to be used as `map()` operation:

{% highlight python %}
'''
    @ function : replace_name
'''
def replace_name(x):
    return movies[movies['movieId']==x].title.values[0]
{% endhighlight %}

So now we can check that the dataframe `ratings` should look like this:

<img src="{{ site.baseurl }}/assets/posts/2018-11-17-PrincipalComponentAnalysis/03.png">

As expected `movieId` is now actual `title`.

{% highlight python %}
M = ratings.pivot_table(index=['userId'], columns=['movieId'], values='rating')
m = M.shape
df1 = M.replace(np.nan, 0, regex=True)
X_std = StandardScaler().fit_transform(df1)
{% endhighlight %}

Using `pivot_table` function `ratings` dataframe structure is arranged by `userId` and `movieId`.
