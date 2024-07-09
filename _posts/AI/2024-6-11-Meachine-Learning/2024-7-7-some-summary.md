---
layout: post
title: 【Meachine Learning】07 some summary of Meachine Learning
categories: [AI, Meachine Learning]
tags: [summary]
math: true
---

## 1 cost function

| model               | cost function               | formula                                                                                                                                                                                                                                 |
| ------------------- | --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| linear regression   | squared error cost function | $$J(w,b) = \frac{1}{2m} \sum\limits_{i = 0}^{m-1} (f_{w,b}(x^{(i)}) - y^{(i)})^2$$                                                                                                                                                      |
| logistic regression | loss function               | $$loss(f_{\mathbf{w},b}(\mathbf{x}^{(i)}), y^{(i)}) = (-y^{(i)} \log\left(f_{\mathbf{w},b}\left( \mathbf{x}^{(i)} \right) \right) - \left( 1 - y^{(i)}\right) \log \left( 1 - f_{\mathbf{w},b}\left( \mathbf{x}^{(i)} \right) \right)$$ |

## 2 demo for different algorithms

### 2.1 linear regression demo

| features                                                                              | target                             | code                                                                                                                          |
| ------------------------------------------------------------------------------------- | ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| - population of city                                                                  | - restaurant profit in target city | [predict profit by population](https://github.com/yc913344706/ai-code/blob/main/LinearRegression/population_profit.ipynb)     |
| - Size (sqft) <br/> - Number of Bedrooms <br/> - Number of floors <br/> - Age of Home | - Price (1000s dollars)            | [predict house price by some features](https://github.com/yc913344706/ai-code/blob/main/LinearRegression/house_predict.ipynb) |

### 2.2 logistic regression demo

| features                            | target                          | code                                                                                                                                 |
| ----------------------------------- | ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| - Exam 1 score <br/> - Exam 2 score | - Admitted <br/> - Not admitted | [predict admission chance by exam score](https://github.com/yc913344706/ai-code/blob/main/LogisticRegression/admission_chance.ipynb) |
| - test 1 score <br/> - test 2 score | - microchip accept <br/> - microchip reject             | [predict microchip inspect result by test score](https://github.com/yc913344706/ai-code/blob/main/LogisticRegression/microchip_QA.ipynb)         |
