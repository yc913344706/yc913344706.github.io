---
layout: post
title: 【Meachine Learning】01 Concept
categories: [AI, Meachine Learning]
---

## 1. Demo

| **Application**                          | **Description**                                                                                                                                                                        |
| ---------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Web Search**                           | Search engines like Google, Bing, and Baidu use machine learning to rank web pages for relevant search results.                                                                        |
| **Photo Tagging**                        | Apps like Instagram and Snapchat use machine learning to recognize and label friends in your uploaded pictures.                                                                        |
| **Movie Recommendations**                | Streaming services use machine learning to recommend movies and shows based on your viewing history (e.g., recommending similar movies after watching Star Wars).                      |
| **Voice to Text and Virtual Assistants** | Machine learning is used in voice-to-text features on phones and virtual assistants like Siri and Google Assistant for tasks like writing text messages, playing music, and searching. |
| **Email Spam Detection**                 | Email services use machine learning to flag and filter out spam emails, such as those claiming you've won a million dollars.                                                           |
| **Industrial Applications**              | Machine learning optimizes wind turbine power generation to address climate change and enhances industrial processes.                                                                  |
| **Healthcare Diagnostics**               | Machine learning assists doctors in making accurate diagnoses in hospitals.                                                                                                            |
| **Factory Inspections**                  | Computer vision, a subset of machine learning, is used in factories to inspect products for defects during the assembly line process.                                                  |

## 2. Application

It turns out that there are a few basic things that we could program a machine to do, such as how to find the shortest path from a to b, like in your GPS. But for the most part, we just did not know how to write an explicit program to do many of the more interesting things, such as perform web search, recognize human speech, diagnose diseases from X-rays or build a self-driving car.

The only way we knew how to do these things was to have a machine learn to do it by itself.

In fact, I find it hard to think of any industry that machine learning is unlikely to touch in a significant way now or in the near future.

Even though machine learning is already creating tremendous amounts of value in the software industry, I think there could be even vastly greater value that has yet to be created outside the software industry in sectors such as retail, travel, transportation, automotive, materials manufacturing, and so on.

Because of the massive untapped opportunities across so many different sectors, today there is a vast unfulfilled demand for this skill set. That's why this is such a great time to be learning about machine learning.

If you find machine learning applications exciting, I hope you stick with me through this class.

## 3. Definition

Arthur Samuel: He defined machine learning as the field of study that gives computers the ability to learn without being explicitly programmed.

Samuel's claim to fame was that back in the 1950s, he wrote a checkers playing program.

The amazing thing about this program was that Arthur Samuel himself wasn't a very good checkers player.

What he did was he had programmed the computer to play maybe tens of thousands of games against itself.

By watching what social support positions tend to lead to wins and what positions tend to lead to losses the checkers plane program learned over time what are good or bad suport positions by trying to get a good and avoid bad positions, this program learned to get better and better at playing checkers because the computer had the patience to play tens of thousands of games against itself.

It was able to get so much checkers playing experience that eventually it became a better checkers player than also, Samuel himself.

Arthur Samuel's definition was a rather informal one.

but let's see first, the two main types of machine learning are supervised learning and unsupervised learning.

## 4. Supervised Learning

I think 99 percent of the economic value created by machine learning today is through one type of machine learning, which is called supervised learning.

The key characteristic of supervised learning is that you give your learning algorithm examples to learn from. That includes the right answers, whereby right answer, I mean, the correct label y for a given input x, and is by seeing correct pairs of input x and desired output label y that the learning algorithm eventually learns to take just the input alone without the output label and gives a reasonably accurate prediction or guess of the output.

![03-supervised-learning-is-a-to-b-mapping](/assets/images/ai-for-everyone/01-what-is-ai/03-ai-supervised-learning.png)

### 4.1 regression

a specific example: predict house price

The task of the learning algorithm is to produce more of these right answers, specifically predicting what is the likely price for other houses like your friend's house. That's why this is supervised learning.

To define a little bit more terminology, this housing price prediction is the particular type of supervised learning called regression.

### 4.2 classification

Using a patient's medical records your machine learning system tries to figure out if a tumor that is a lump is malignant meaning cancerous or dangerous.

It turns out that in classification problems you can also have more than two possible output categories.

So to summarize classification algorithms predict categories. Categories don't have to be numbers. It could be non numeric for example, it can predict whether a picture is that of a cat or a dog. And it can predict if a tumor is benign or malignant. Categories can also be numbers like 0, 1 or 0, 1, 2.

But what makes classification different from regression when you're interpreting the numbers is that classification predicts a small finite limited set of possible output categories such as 0, 1 and 2 but not all possible numbers in between like 0.5 or 1.7.

### 4.3 summary

So to recap supervised learning maps input x to output y, where the learning algorithm learns from the quote right answers.

The two major types of supervised learning our regression and classification.
In a regression application like predicting prices of houses, the learning algorithm has to predict numbers from infinitely many possible output numbers.
Whereas in classification the learning algorithm has to make a prediction of a category, all of a small set of possible outputs.

## 5. unsupervised learning

Whereas in supervised learning, the data comes with both inputs x and input labels y, in unsupervised learning, the data comes only with inputs x but not output labels y, and the algorithm has to find some structure or some pattern or something interesting in the data.

we call it unsupervised because we're not trying to supervise the algorithm. To give some "right answer" for every input, instead, we asked the our room to figure out all by yourself what's interesting. Or what patterns or structures that might be in this data, with this particular data set.

### 5.1 clustering

clustering algorithm: it takes data without labels and tries to automatically group them into clusters.

demo:

- google news
- clustering genetic or DNA data
- automatically group your customers

### 5.2 anomaly detection

anomaly detection: find unusual events.

- demo: fraud detection

### 5.3 dimensionality reduction

dimensionality reduction: it lets you take a big data-set and almost magically compress it to a much smaller data-set while losing as little information as possible.
