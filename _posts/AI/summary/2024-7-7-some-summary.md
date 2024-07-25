---
layout: post
title: 【Meachine Learning】07 some summary of Meachine Learning
categories: [AI, Meachine Learning]
tags: [demo summary, numpy operation, sympy]
math: true
---

## 1 cost function

| model               | cost function               | Neural Network Name |
| ------------------- | --------------------------- | ------------------- |
| linear regression   | squared error cost function |  [MeanSquaredError](https://www.tensorflow.org/api_docs/python/tf/keras/losses/MeanSquaredError)                   |
| logistic regression | loss function               |  [BinaryCrossentropy](https://www.tensorflow.org/api_docs/python/tf/keras/losses/BinaryCrossentropy)                   |

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

## 3 code demo for different algorithms

### 3.1 linear regression demo

| features                                                                              | target                             | code                                                                                                                          |
| ------------------------------------------------------------------------------------- | ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| - population of city                                                                  | - restaurant profit in target city | [predict profit by population](https://github.com/yc913344706/ai-code/blob/main/LinearRegression/population_profit.ipynb)     |
| - Size (sqft) <br/> - Number of Bedrooms <br/> - Number of floors <br/> - Age of Home | - Price (1000s dollars)            | [predict house price by some features](https://github.com/yc913344706/ai-code/blob/main/LinearRegression/house_predict.ipynb) |

### 3.2 logistic regression demo

| features                            | target                                      | code                                                                                                                                     |
| ----------------------------------- | ------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| - Exam 1 score <br/> - Exam 2 score | - Admitted <br/> - Not admitted             | [predict admission chance by exam score](https://github.com/yc913344706/ai-code/blob/main/LogisticRegression/admission_chance.ipynb)     |
| - test 1 score <br/> - test 2 score | - microchip accept <br/> - microchip reject | [predict microchip inspect result by test score](https://github.com/yc913344706/ai-code/blob/main/LogisticRegression/microchip_QA.ipynb) |

### 3.3 neural network demo

- [binary_handwritten_digit_recognization](https://github.com/yc913344706/ai-code/blob/main/NeuralNetwork/binary_handwritten_digit_recognization.ipynb)
- [multiple_handwritten_digit_recognization](https://github.com/yc913344706/ai-code/blob/main/NeuralNetwork/multiple_handwritten_digit_recognization.ipynb)

### 3.4 decision tree demo

- [diagnose Context Cardiovascular diseases by multiple features](https://github.com/yc913344706/ai-code/blob/main/DecisionTree/DecisionTree_RandomForest_XGBoost.ipynb)


### 3.5 intuition for model evaluate

- [model evaluate for: linear regression](https://github.com/yc913344706/ai-code/blob/main/LinearRegression/RealDemo01_Linear.ipynb)
- [model evaluate for: Neural Network: linear regression](https://github.com/yc913344706/ai-code/blob/main/LinearRegression/RealDemo01_NN_Linear.ipynb)
- [model evaluate for: Neural Network: logistic regression](https://github.com/yc913344706/ai-code/blob/main/LogisticRegression/RealDemo01_NN_Logistic.ipynb)

### 3.6 Clustering demo

- K-Means:
  - [ClusteringIntuition](https://github.com/yc913344706/ai-code/blob/main/Clustering/ClusteringIntuition.ipynb)
  - [KMeans01_ImageCompression](https://github.com/yc913344706/ai-code/blob/main/Clustering/KMeans01_ImageCompression.ipynb)
- Anomaly detection:
  - [detect computer server anomalies](https://github.com/yc913344706/ai-code/blob/main/Clustering/DetectServerAnomaly.ipynb)

## 4 derivative

遇到复杂计算找python绝对不让你失望，sympy是一个Python的科学计算库，用一套强大的符号计算体系完成诸如多项式求值、求极限、解方程、求积分、微分方程、级数展开、矩阵运算等等计算问题。

doc:

- https://blog.csdn.net/cj151525/article/details/95756847


## 100 other doc summary

### 100.1 mathmatics

- [pupulation mean](https://wumbo.net/formulas/population-mean/)

### 100.2 markdown

- [mathjax/katex/latex in markdown](https://freeopen.github.io/mathjax/)
- [MathJax markdown demo](https://timefy.github.io/posts/2fa412f3.html)

### 100.3 concepts in ML

- [batch, batch size, iteration/step](https://blog.csdn.net/lfs666666/article/details/88135618)

