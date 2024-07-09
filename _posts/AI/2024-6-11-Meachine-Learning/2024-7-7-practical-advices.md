---
layout: post
title: 【Meachine Learning】05 practical advices
categories: [AI, Meachine Learning]
tags: [convergence, learning curve, learning rate]
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

- 0.001 0.01 1
- 3x

![02-choose-learning-rate](/assets/images/meachine-learning/practical-advice/02-choose-learning-rate.png)

## 4. underfitting and overfitting

### 4.1 generalization

Technically we say that you want your learning algorithm to generalize well, which means to make good predictions even on brand new examples that it has never seen before.

### 4.2 demo of underfitting and overfitting

let's see some demo:

- underfitting and overfitting in linear regression:
  ![03-overfit-linear-regression](/assets/images/meachine-learning/practical-advice/03-overfit-linear-regression.png)
- underfitting and overfitting in logistic regression:
  ![04-overfit-logistic-regression](/assets/images/meachine-learning/practical-advice/04-overfit-logistic-regression.png)

### 4.3 concept of bias and variance

when underfitting:

- This preconception that the data is linear causes it to fit a straight line that fits the data poorly, leading it to underfitted data.
- this also called `high bias`.

when overfitting:

- Because even though it fits the training set very well, it has fit the data almost too well, hence is overfit.
- It does not look like this model will generalize to new examples that's never seen before.
- this also called `high variance`.

### 4.4 why high variance

The intuition behind overfitting or high-variance is that the algorithm is trying very hard to fit every single training example.

It turns out that if your training set were just even a little bit different, say one house was priced just a little bit more little bit less, then the function that the algorithm fits could end up being totally different.

If two different machine learning engineers were to fit this fourth-order polynomial model, to just slightly different datasets, they couldn't end up with totally different predictions or highly variable predictions.

That's why we say the algorithm has high variance.

### 4.5 how to solve overfitting/high variance?

#### 4.5.1 get more training data

![05-against-overfitting01](/assets/images/meachine-learning/practical-advice/05-against-overfitting01.png)

#### 4.5.2 feature selection

MARK: wait to complete.

#### 4.5.3 regularization

what is regularization? it looks like we want to penalize the model for having too many large weights with the parameter `lambda`.

like the following img, we add a parameter `lambda` to the cost function. the larger the lambda, if we want minimum the cost function, then the weights will be smaller.

![regularization intuition](/assets/images/meachine-learning/practical-advice/05-against-overfitting03.png)

So, when lambda:

- too big: model will underfit;
- too small: model will overfit;

**cost function**

- linear regression
  - without regularization: $$J(w,b) = \frac{1}{2m} \sum\limits_{i = 0}^{m-1} (f_{w,b}(x^{(i)}) - y^{(i)})^2$$ 
  - with regularization: $$J(w,b) = \frac{1}{2m} \sum\limits_{i = 0}^{m-1} (f_{w,b}(x^{(i)}) - y^{(i)})^2 + \frac{\lambda}{2m}  \sum\limits_{j=0}^{n-1} w_j^2$$ 

- logistic regression:
  - without regularization: $$ J(\mathbf{w}.b) = \frac{1}{m} \sum\limits_{i=0}^{m-1} \left[ (-y^{(i)} \log\left(f_{\mathbf{w},b}\left( \mathbf{x}^{(i)} \right) \right) - \left( 1 - y^{(i)}\right) \log \left( 1 - f_{\mathbf{w},b}\left( \mathbf{x}^{(i)} \right) \right)\right]$$
  - with regularization: $$J(\mathbf{w},b) = \frac{1}{m}  \sum\limits_{i=0}^{m-1} \left[ -y^{(i)} \log\left(f_{\mathbf{w},b}\left( \mathbf{x}^{(i)} \right) \right) - \left( 1 - y^{(i)}\right) \log \left( 1 - f_{\mathbf{w},b}\left( \mathbf{x}^{(i)} \right) \right) \right] + \frac{\lambda}{2m}  \sum\limits_{j=0}^{n-1} w_j^2$$

**gradient descent**

- linear regression and logistic regression:
  - gradient descent approach: 
    - $$\begin{align*}& \text{repeat until convergence:} \; \lbrace \newline \; & b := b -  \alpha \frac{\partial J(\mathbf{w},b)}{\partial b} \newline       \; & w_j := w_j -  \alpha \frac{\partial J(\mathbf{w},b)}{\partial w_j}  \; & \text{for j := 0..n-1}\newline & \rbrace\end{align*}$$
  - partial derivatives: 
    - $$\frac{\partial J(\mathbf{w},b)}{\partial b}  = \frac{1}{m} \sum\limits_{i = 0}^{m-1} (f_{\mathbf{w},b}(\mathbf{x}^{(i)}) - \mathbf{y}^{(i)})$$
    - $$\frac{\partial J(\mathbf{w},b)}{\partial w_j}  = \frac{1}{m} \sum\limits_{i = 0}^{m-1} (f_{\mathbf{w},b}(\mathbf{x}^{(i)}) - \mathbf{y}^{(i)})x_{j}^{(i)} + \frac{\lambda}{m} w_j$$
- the difference between the two is the $f_{\mathbf{w},b}(\mathbf{x}^{(i)})$
  - linear regression: $$f_{\mathbf{w},b}(\mathbf{x}^{(i)}) = \mathbf{w} \cdot \mathbf{x}^{(i)}+b$$
  - logistic regression: $$f_{\mathbf{w},b}(\mathbf{x}^{(i)}) = \frac{1}{1+e^{-(\mathbf{w} \cdot \mathbf{x^{(i)}} + b)}}$$

**code demo**

- [logistic regression demo for regularization](https://github.com/yc913344706/ai-code/blob/main/LogisticRegression/microchip_QA.ipynb)

### 4.6 how to solve underfitting/high bias?

#### 4.6.1 feature mapping

One way to fit the data better is to create more features from each data point. In the provided function `map_feature`, we will map the features into all polynomial terms of $x_1$ and $x_2$ up to the sixth power.

$$\mathrm{map\_feature}(x) = 
\left[\begin{array}{c}
x_1\\
x_2\\
x_1^2\\
x_1 x_2\\
x_2^2\\
x_1^3\\
\vdots\\
x_1 x_2^5\\
x_2^6\end{array}\right]$$

As a result of this mapping, our vector of two features has been transformed into a 27-dimensional vector. 

![08-feature-mapping-demo-6-power](/assets/images/meachine-learning/feature-engineering/08-feature-mapping-demo-6-power.png)



