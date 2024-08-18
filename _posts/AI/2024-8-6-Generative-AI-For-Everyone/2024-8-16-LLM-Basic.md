---
layout: post
title: 【Generative AI】01 Generative AI Basic
categories: [AI, Generative AI]
tags: [Basic]
---

## 1 what is generative ai

Generative AI, refers to AI, or artificial intelligence systems that can produce high-quality content, specifically text, images and audio.

### 1.1 AI tools landscape

Let's start by looking at where generative AI fits within the AI landscape. 

![01-ai-tools-landscape](/assets/images/llm-for-everyone/01-ai-tools-landscape.png)

One of the most important tools in AI is supervised learning, which turns out to be really good at labeling things.

Second to it that started to work really well only fairly recently is generative AI. 

### 1.2 GenAI vs supervised learning

it turns out generative AI is built using supervised learning.

Large-scale supervised learning remains important today, but this idea of **very large models for labeling things** is how we got to generative AI today. 

So, GenAI uses supervised learning to repeatedly predict what is the next word.

![02-how-llm-works](/assets/images/llm-for-everyone/02-how-llm-works.png)

That's how large language models work; they're trained to repeatedly predict the next word.

It turns out that many people, perhaps including you, are already finding these models useful for day-to-day activities at work to help with writing, to find basic information, or to be a thought partner to help think things through.

## 2 GenAI can do what

What is Generative AI good for? One of the reasons that question is a bit hard to answer is because AI is a **general-purpose technology**.

But let's take a look at what a general purpose technology really means.

other general-purpose technologies:
- electricity
- internet

Let's discuss more broadly a framework for what tasks LLMs can do.

### 2.1 writing task

base a small set of instructions, generate a long piece of text.

- optimize your words
- brainstorming companion
  - brainstorm 5 creative names for peanut butter cookies
  - brainstorm ideas for increasing cookie sales
- translation
- answering questions
  - and if you give them access to information specific to a company, they can help members of your team find information that they need.
- writing copy
  - writing a press release
  - When writing a prompt, you find that if you can give the large language model more context or more background information, then it will write more specific and better copy for you. 
    - Otherwise, you will get a generic copy

### 2.2 reading task

base a long piece of text, generate a short piece of text.

- proof reading
- summarizing an article
  - web interface
  - software application:
    - summarize all the communication of call center
  - If you can think of a task where you wish you had someone that could read a piece of text, and just say a few things or give a few quick indications of what was in that piece of text, that could be a good candidate for a reading task to get an LLM to do for you.
- choose the department which email should routing
- reputation monitoring
  - restaurant review sentiment analysis
    - llm to recognize one review, output the result
    - and, another dashboard software to show change


### 2.3 chatting bot

continuous conversation with a human, maybe you, maybe your **customer**.

如果有很多对话有相似的特性，那么就可以考虑是否可以 **开发/使用定制的LLM** 来帮助自动化这些对话。

- partner for inference or brainstorm.

- customer service chatbot
- specialized chatbot
  - Some of these bots can also interface with the rest of a company's software system and take actions such as to put in an order for a cheeseburger to be delivered.
- IT service chatbot
  - A bot like this that needs to be send a text message to verify identity and actually help reset the password. This is a bot that would need to be empowered, to actually take action in the world such as to get a text message to be sent to someone.


## 3 two LLM application

- web interface-based application
- software-based LLM application
  - a software-based LLM application will need access to information about your specific company's info or data, whereas a general large language model on the Internet probably doesn't have that information.

![03-llm-application](/assets/images/llm-for-everyone/03-llm-app-type.png)

上面这张图，已经体现了一些 software-based LLM application 的关键使用技巧：

- 对接其他应用:
  - reading 根据评论内容，分析评论是正面的还是负面的。
  - chatbot 根据用户的输入，生成汉堡订单。
- RAG（Retrieval Augmented Generation，检索增强生成）:
  - writing 通过检索私有数据库，生成答案。

总的来看：
- web interface-based application，指的是不需要开发应用，不需要对接非公开的数据库，只需要在网页上输入内容，然后得到结果。
- software-based LLM application，指的是需要开发应用，需要对接非公开的数据库，需要自己开发应用，然后通过接口调用 LLM。 


## 4 more about software-based LLM application

对于 writting, reading, chatting 类型的LLM应用，当我们基于它进行定制化开发时，我们可以实现很多定制化、行业化的功能。

比如：

- writing task
  - fine tune：可以让输出的内容，使用张三的语气，或者用生成小说。
- reading task
  - 对接其他应用: 可以阅读用户的评论，输出是正面的还是负面的，并将结果存储，用其他软件来进行 汇总报表展示，这样就可以知道用户对产品的评价情况。
- chatting bot
  - fine tune：可以让输出的内容，使用张三的语气，或者用易经里的话来进行内容生成，与用户聊天。

我们来看具象的例子：

评论识别，报表展示评论的情感，生成报告。

![04-llm-software-app-demo](/assets/images/llm-for-everyone/04-llm-software-app-demo.png)

汇总所有的语音通话记录，生成报告。

![04-llm-software-app-demo2](/assets/images/llm-for-everyone/04-llm-software-app-demo2.png)

根据用户的聊天，来下订单

![04-llm-software-app-demo3](/assets/images/llm-for-everyone/04-llm-software-app-demo3.png)

不同类型的chatbot

![06-specialization-chatbot](/assets/images/llm-for-everyone/06-specialization-chatbot.png)

## 5 LLM can and cannot do

### 5.1 LLM can do

#### 5.1 LLM as a fresh college grad

If you're trying to figure out what prompting an LLM can do, here's one question that I found to provide a useful mental framework. Which is I'll ask myself,

> can a fresh college graduate following only the instructions in the prompt to complete the tasks you want? 

Here's another example, can a fresh college grad write a press release without any information about the COO or your company? 

- Well, this fresh college grad just graduated from college. They only just met you and don't know anything about you or your business, and so the best they could do is maybe write a really generic and not quite satisfying press release like this. 
- But on the flip side, if you were to give them more context about your business and about the COO, then we can ask, can this fresh college grad write a press release given the basic relevant context? 
- And I think they may be able to do that decently well, and so too, can the large language model. 

This rule of thumb of **asking what a fresh college grad can do** is an imperfect rule of thumb, there are things college graduates can do that LLMs cannot and vice versa. But I found this to be **a useful starting point** for thinking through what LLMs can and cannot do. 

#### 5.2 llm as a reasoning engine

- LLMs have a lot of general knowledge, but they don't know everything
- By providing relevant context in the prompt, we ask an LLM toread a piece of text, then process it to get an answer
- We're using it as a reasoning engine to process informationrather than using it as a source of information


### 5.2 llm limitation

#### 5.2.1 knowledge cutoffs. 

An LLM's knowledge of the world is frozen at the time of his training.

but now, it seems that llm can connect internet?

#### 5.2.2 hallucination

A second limitation of LLMs is that they will sometimes just make things up, and we call these hallucinations.

Hallucinations can have serious consequences. 

#### 5.2.3 length limitations

LLMs also have a technical limitation in:
- input length
- output length
- context length: input + output

#### 5.2.4 not work well with unstructured data

one major limitation of generative AI is that they do not currently work well with structured data.

In contrast, to structured data, generative AI tends to work best with unstructured data. Structured data refers to tabular data of the soil you would store in a spreadsheet, whereas unstructured data refers to text, images, audio, video. 

#### 5.2.5 output bias even harmful content

large language models can bias output and can sometimes output toxic or other harmful speech. 

And so if you're using an LLM in an application where such biases could cause harm, I would use care in how we prompt and apply the LLM to make sure we don't contribute to such undesirable biases. 


Finally, some LLMs can also occasionally output toxic or other harmful speech.

Fortunately, all the major large language providers have been working hard on the safety of these models, and so most models have gotten much safer over time.

## 6 tips for prompting

### 6.1 be detailed and specific

![05-prompt-detail](/assets/images/llm-for-everyone/05-prompt-detail.png)

### 6.2 guide the model to think through its anwser

![05-prompt-guide-step01](/assets/images/llm-for-everyone/05-prompt-guide-step01.png)

![05-prompt-guide-step02](/assets/images/llm-for-everyone/05-prompt-guide-step02.png)


### 6.3 experiment and iterate

- 没有适用于任何人、任何场景的完美的prompt。
- 适合自己场景的 prompt，需要持续优化得到。
- 如何优化：
  - 输入更多上下文、更具体的要求
  - 引导 LLM 按步骤去得到你的目标

![05-prompt-iterate-optimize](/assets/images/llm-for-everyone/05-prompt-iterate-optimize.png)

关于 initial prompt: 
- 不要过度看重 initial prompt ，不会因为 initial prompt 不太理想，你就会得到多么坏的结果。多尝试、多迭代就好了。
- 而且，initial prompt的回答，还会帮助你重新思考、整理甚至调整你的想法

two caveats:
- 小心 LLM 过度自信的回答
- 当你决定采用回答时，一定要double checking

## 7 more about GenAI

### 7.1 image generation

#### 7.1.1 diffusion model

Image generation today is mostly done via a method called a diffusion model.

Diffusion models have learned from huge numbers of images found on the Internet or elsewhere. 

It turns out that at the heart of a diffusion model is supervised learning.

#### 7.1.2 diffusion model step intuition

- The first step is to take this image, and gradually add more and more noise to it.

![07-image-add-noise](/assets/images/llm-for-everyone/07-image-add-noise.png)

- and the train step is take a reverse order.

![07-image-generation](/assets/images/llm-for-everyone/07-image-generation.png)

then use supervised learning for million, billion images, then get a model.

I'm illustrating this process using four steps of adding noise on the previous slide, and four steps of removing noise on this slide. 

But in practice, maybe about 100 steps will be more typical for a diffusion model. 

#### 7.1.3 diffusion model + text step intuition

This algorithm will work for generating pictures completely at random. 

But we want to be able to control the image it generates by specifying a prompt to tell it what we want it to generate.

the key is add 'prompt' to each step.

- add noise with text

![08-image-txt-add-noise](/assets/images/llm-for-everyone/08-image-txt-add-noise.png)

- image generation with text

![08-image-txt-generation](/assets/images/llm-for-everyone/08-image-txt-generation.png)





