---
layout: post
title: 【Meachine Learning】07 some summary of Meachine Learning
categories: [AI, Meachine Learning]
tags: [demo summary]
math: true
---

## 1 cost function

| model               | cost function               | formula                                                                                                                                                                                                                                 |
| ------------------- | --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| linear regression   | squared error cost function | $$J(w,b) = \frac{1}{2m} \sum\limits_{i = 0}^{m-1} (f_{w,b}(x^{(i)}) - y^{(i)})^2$$                                                                                                                                                      |
| logistic regression | loss function               | - $$loss(f_{\mathbf{w},b}(\mathbf{x}^{(i)}), y^{(i)}) = (-y^{(i)} \log\left(f_{\mathbf{w},b}\left( \mathbf{x}^{(i)} \right) \right) - \left( 1 - y^{(i)}\right) \log \left( 1 - f_{\mathbf{w},b}\left( \mathbf{x}^{(i)} \right) \right)$$ <br/> - $$J(\mathbf{w},b) = \frac{1}{m}  \sum\limits_{i=0}^{m-1} \left[ -y^{(i)} \log\left(f_{\mathbf{w},b}\left( \mathbf{x}^{(i)} \right) \right) - \left( 1 - y^{(i)}\right) \log \left( 1 - f_{\mathbf{w},b}\left( \mathbf{x}^{(i)} \right) \right) \right] + \frac{\lambda}{2m}  \sum\limits_{j=0}^{n-1} w_j^2$$ |

## 2 numpy operation

### 2.1 dot product and transpose

- ![15-dot-project-demo](/assets/images/meachine-learning/linear-regression/15-dot-project-demo.gif)
- [matrix multiplication](https://blog.csdn.net/STRVE/article/details/106739349)
- [more about (\*, np.multiply, np.dot, np.matmul, and @)](https://mkang32.github.io/python/2020/08/30/numpy-matmul.html)

### 2.2 boardcast and add

#### 2.2.1 boardcast

boardcast index:

![Numpy Boardcast](/assets/images/meachine-learning/numpy/C2_W1_Assign1_BroadcastIndexes.PNG)

boardcast intuition:

![Numpy Boardcast intuition](/assets/images/meachine-learning/numpy/C2_W1_Assign1_Broadcasting.gif)

Boardcast Matrix:

![Numpy Boardcast Matrix](/assets/images/meachine-learning/numpy/C2_W1_Assign1_BroadcastMatrix.PNG)

#### 2.2.2 add

Boardcast VectorAdd:

![Numpy Boardcast VectorAdd](/assets/images/meachine-learning/numpy/C2_W1_Assign1_VectorAdd.PNG)


### 2.3 slice

- [numpy array slice](https://blog.csdn.net/weixin_43629813/article/details/101122997)


## 3 demo for different algorithms

### 3.1 linear regression demo

| features                                                                              | target                             | code                                                                                                                          |
| ------------------------------------------------------------------------------------- | ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| - population of city                                                                  | - restaurant profit in target city | [predict profit by population](https://github.com/yc913344706/ai-code/blob/main/LinearRegression/population_profit.ipynb)     |
| - Size (sqft) <br/> - Number of Bedrooms <br/> - Number of floors <br/> - Age of Home | - Price (1000s dollars)            | [predict house price by some features](https://github.com/yc913344706/ai-code/blob/main/LinearRegression/house_predict.ipynb) |

### 3.2 logistic regression demo

| features                            | target                          | code                                                                                                                                 |
| ----------------------------------- | ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| - Exam 1 score <br/> - Exam 2 score | - Admitted <br/> - Not admitted | [predict admission chance by exam score](https://github.com/yc913344706/ai-code/blob/main/LogisticRegression/admission_chance.ipynb) |
| - test 1 score <br/> - test 2 score | - microchip accept <br/> - microchip reject             | [predict microchip inspect result by test score](https://github.com/yc913344706/ai-code/blob/main/LogisticRegression/microchip_QA.ipynb)         |
