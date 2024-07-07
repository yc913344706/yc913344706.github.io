---
layout: post
title: 【Meachine Learning】04 About Feature Engineering
categories: [AI, Meachine Learning]
tags: [Feature Engineering]
math: true
---

## 0 background

some definition about feature engineering:

- the definition of feature engineering is the process of selecting, manipulating, and transforming raw data into features that better represent the underlying problem to the predictive model.

- Depending on what insights you may have into the application, rather than just taking the features that you happen to have started off with sometimes by defining new features, you might be able to get a much better model. That's feature engineering.

- it's a problem about how you can choose or engineer the most appropriate features for your learning algorithm.

let's try to put all problem about feature to this one blog.

## 1 feature scaling

### 1.1 why feature scaling is important?

feature scaling that will enable gradient descent to run much faster.

if not scaling:

- gradient descent will be slow.
- we must analyze the the range, get the suitable vector w, so it will be more quickly.

if scaling: it will be faster.

let's see why:

1. normaly the size of x_1 bigger, the size of w_1 will be smaller.
   ![01-w-x-is-inversely-proportional](/assets/images/meachine-learning/feature-engineering/01-w-x-is-inversely-proportional.png)
2. the contour plot will looks like this:
   ![02-feature-range-with-contour](/assets/images/meachine-learning/feature-engineering/02-feature-range-with-contour.png)
3. if we can scale the feature, then we can let contour plot looks like this:
   ![03-features-scaling-concept](/assets/images/meachine-learning/feature-engineering/03-features-scaling-concept.png)

### 1.2 how to do feature scaling?

there are three ways to do feature scaling:

1. dividing by maxium
   ![04-FS01-dividing-by-maxium](/assets/images/meachine-learning/feature-engineering/04-FS01-dividing-by-maxium.png)
2. mean normalization

   - $$x_i := \dfrac{x_i - \mu_i}{max - min} $$

     ![04-FS02-mean-normalization](/assets/images/meachine-learning/feature-engineering/04-FS02-mean-normalization.png)

3. z-score normalization

   - $$\mu_j = \frac{1}{m} \sum_{i=0}^{m-1} x^{(i)}_j$$
   - $$\sigma^2_j = \frac{1}{m} \sum_{i=0}^{m-1} (x^{(i)}_j - \mu_j)^2$$
   - $$x^{(i)}_j = \dfrac{x^{(i)}_j - \mu_j}{\sigma_j}$$

     ![04-FS03-z-score-normalization](/assets/images/meachine-learning/feature-engineering/04-FS03-z-score-normalization.png)

### 1.3 when to do feature scaling?

But, there are some cases that we don't need to do feature scaling:

![05-FS-only-necessary](/assets/images/meachine-learning/feature-engineering/05-FS-only-necessary.png)

There's almost never any harm to carrying out feature re-scaling. When in doubt, I encourage you to just carry it out.

### 1.4 code demo

[feature scaling code demo](https://github.com/yc913344706/ai-code/blob/main/FeatureEngineer/FeatureScaling.ipynb)

## 2. design new features

### 2.1 demo

Depending on what insights you may have into the application, rather than just taking the features that you happen to have started off with sometimes by defining new features, you might be able to get a much better model. That's feature engineering.

such as: You may have an intuition that the area of the land is more predictive of the price, than the frontage and depth as separate features.

![demo of house price prediction](/assets/images/meachine-learning/feature-engineering/06-design-new-feature-demo1.png)

### 2.2 polynomial features

why we need polynomial features? it may because we want let the model more fit the data.

just like the following picture, it obviously that the straight line is not a good fit for the data. we need high order polynomial to fit the data.

![07-polynomial-regression-intuition](/assets/images/meachine-learning/feature-engineering/07-polynomial-regression-intuition.png)

but how to find a good order?

MARK: wait to complete.
