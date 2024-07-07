---
layout: post
title: 【Meachine Learning】05 practical advices
categories: [AI, Meachine Learning]
tags: [practical advices]
math: true
---

## 1. feature engineering

[feature engineering]({% link _posts/AI/2024-6-11-Meachine-Learning/2024-7-7-feature-engineering.md %})

## 2. check model convergence

### 2.1 how make sure that gradient descent is working well?

1. plot the cost function J, which is calculated on the training set.

- This curve is also called a learning curve.
- If J ever increases after one iteration, that means either Alpha is chosen poorly, and it usually means Alpha is too large, or there could be a bug in the code.

### 2.2 way to decide when your model is done training

1. convergence of learning curce

- By 400 iterations, it looks like the curve has flattened out. This means that gradient descent has more or less converged because the curve is no longer decreasing.
- By the way, the number of iterations that gradient descent takes a conversion can vary a lot between different applications.

2. automatic convergence test

- threshold epsilon, such as 0.001 or 10^-3.
- If the cost J decreases by less than this number epsilon on one iteration, then you're likely on this flattened part of the curve that you see on the left and you can declare convergence.
- I usually find that choosing the right threshold epsilon is pretty difficult. I actually tend to look at graphs like this one on the left, rather than rely on automatic convergence tests.

## 3. about learning rate

multiple learning curve show if learning rate too big.

![different large learning rate](/assets/images/meachine-learning/practical-advice/01-learning-rate-too-large.png)

debugging tip:
- with a small enough learning rate.
- if with that, J doesn't decrease on every single iteration, but instead sometimes increases, then that usually means there's a bug somewhere in the code. 
- and caution is that, this is just a debug method!!!

so, how to choose learning rate step:
- 0.001  0.01  1
- 3x

![02-choose-learning-rate](/assets/images/meachine-learning/practical-advice/02-choose-learning-rate.png)





