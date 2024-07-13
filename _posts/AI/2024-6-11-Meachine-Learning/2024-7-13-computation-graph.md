---
layout: post
title: 【Meachine Learning】09 Computation Graph
categories: [AI, Meachine Learning]
tags: [Computation Graph, Back Propagation]
math: true
---

## 1 usage of computation graph

The computation graph is a key idea in deep learning, and it is also how programming frameworks like TensorFlow, automatic compute derivatives of your neural networks.

## 2 intuition

What I'd like to do is show how the computation of the cause function J can be computed step by step using a computation graph. 

![01-computation-graph01](/assets/images/meachine-learning/computation-graph/01-computation-graph01.png)

What we've just done is build up a computation graph. This is a graph, not in a sense of plots with x and y axes, but this is the other sense of the word graph using computer science, which is that this is a set of nodes that is connected by edges or connected by arrows in this case. 

This computation graph shows the forward prop step of how we compute the output a of the neural network. 

It turns out that if a computation graph has n nodes, meaning your n of these boxes and p parameters, so we have two parameters in this case. 

![01-computation-graph02-forward-back-prop](/assets/images/meachine-learning/computation-graph/01-computation-graph02-forward-back-prop.png)

This procedure allows us to compute all the derivatives of j with respect to all the parameters in roughly n plus p steps, rather than n times p steps. If you have a neural network with, said, 10,000 nodes and maybe 100,000 parameters. This would not be considered even a very large neural network by modern standards. Being able to compute the derivatives and 10,000 plus 100,000 steps, which is punch and 10,000 is much better than meeting 10,000 times 100,000 steps, which would be a billion steps. 

![01-computation-graph03-back-prop-nice-for-derivative](/assets/images/meachine-learning/computation-graph/01-computation-graph03-back-prop-nice-for-derivative.png)

The backpropagation algorithm done using the computation graph gives you a very efficient way to compute all the derivatives. That's why it is such a key idea in how deep learning algorithms are implemented today. 

![01-computation-graph04-complex-nn](/assets/images/meachine-learning/computation-graph/01-computation-graph04-complex-nn.png)

In this video, you saw how the computation graph takes all the steps of the calculation needed to compute the output of a neural network a as well as the cost function j. Takes a step-by-step computations and breaks them into the different nodes of computation graph.