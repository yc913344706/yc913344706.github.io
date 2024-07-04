---
layout: post
title: 【Meachine Learning】03 Linear Regression General
categories: [AI, Meachine Learning]
tags: [Linear Regression, Cost Function, Gradient Descent, Multiple Feature]
math: true
---

## background

In general, the number of features is more than the number of samples. just like the following example:

![13-multiple-features](/assets/images/meachine-learning/linear-regression/13-multiple-features.png)

So how to solve this problem? at first, let's take a look at the **Vectorization**.

## Vectorization

What is vectorization?

Vectorization is a method of computing in which we use a vector (a list of numbers) to represent a set of data.

just like the following image, we have multiple features, and multiple value for multiple featrues, so we can use a vector to represent them.

![14-vectorization](/assets/images/meachine-learning/linear-regression/14-vectorization.png)

At here, we use the `dot project`, which is a method of computing in which we use a vector (a list of numbers) to represent a set of data.

<img src="/assets/images/meachine-learning/linear-regression/15-dot-project-demo.gif" width=800>

And, we archieve that by `Numpy`, it's a powerful library for scientific computing.

> more doc:
>
> - [more about (\*, np.multiply, np.dot, np.matmul, and @)](https://mkang32.github.io/python/2020/08/30/numpy-matmul.html)
> - [matrix multiplication](https://blog.csdn.net/STRVE/article/details/106739349)
> - [numpy array slice](https://blog.csdn.net/weixin_43629813/article/details/101122997)

## linear regression code with vectorization

[linear regression code with vectorization](https://github.com/yc913344706/ai-code/blob/main/LinearRegression/house_predict.ipynb)

## Normal equation

Before moving on from this video, I want to make a quick aside or a quick side note on an alternative way for finding w and b for linear regression. This method is called the normal equation.

Whereas it turns out gradient descent is a great method for minimizing the cost function J to find w and b, there is one other algorithm that works only for linear regression and pretty much none of the other algorithms you see in this specialization for solving for w and b and this other method does not need an iterative gradient descent algorithm.

Called the normal equation method, it turns out to be possible to use an advanced linear algebra library to just solve for w and b all in one goal without iterations.

Some disadvantages of the normal equation method are; first unlike gradient descent, this is not generalized to other learning algorithms, such as the logistic regression algorithm that you'll learn about next week or the neural networks or other algorithms you see later in this specialization.

The normal equation method is also quite slow if the number of features and this large.

Almost no machine learning practitioners should implement the normal equation method themselves but if you're using a mature machine learning library and call linear regression, there is a chance that on the backend, it'll be using this to solve for w and b.

If you're ever in the job interview and hear the term normal equation, that's what this refers to. Don't worry about the details of how the normal equation works. Just be aware that some machine learning libraries may use this complicated method in the back-end to solve for w and b.

But for most learning algorithms, including how you implement linear regression yourself, gradient descents offer a better way to get the job done.

## Wait Research Questions

some questions not answered yet, because our purpose is to use AI:

1. normal equation.
