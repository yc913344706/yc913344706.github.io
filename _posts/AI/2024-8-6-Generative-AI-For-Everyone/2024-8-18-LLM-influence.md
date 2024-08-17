---
layout: post
title: 【Generative AI】03 Generative AI Influence
categories: [AI, Generative AI]
tags: [influence]
---

## 1 which can be automated by AI

it turns out that from a technical and business perspective, it's more useful to think of AI not as automating `jobs`, but as automating `tasks`.

And it turns out most jobs involve a collection of many tasks. 

### 1.1 AI automation framework

There's a framework that had originated in economics due to Erik Brynjolfsson, Tom Mitchell, and Daniel Rock for analyzing the work tasks for possible automation using AI.

And if you work in a company with, say, many customer service representatives, the first step to analyzing the potential for using generative AI would be to understand for your business what are the tasks that the representatives in your company do. 

After that, we can then take a look at these different tasks and try to assess their potential for generative AI to either hope or augment, or to automate these tasks. 

![20-ai-automate-tasks-not-jobs](/assets/images/llm-for-everyone/20-ai-automate-tasks-not-jobs.png)

But after an analysis like this and I'll go in a little bit into the specifics of how to carry out this analysis, you might then decide that answering customer chat queries and keeping records of customer interactions have the highest potential, and therefore, focus your efforts on those two tasks. 

So, here is a question, which tasks can be automated by AI?

### 1.2 AI can automate it?

we have seen [Generative AI can do what](https://yc913344706.github.io/posts/LLM-Basic/#2-genai-can-do-what) before, now let's have a re-think about it.

![21-evaluate-AI-potential](/assets/images/llm-for-everyone/21-evaluate-AI-potential.png)

more about business value:
- And so the questions I would ask to frame up my thinking on business value would be things like how much time is spent on this task? So how much time savings can we actually realize?
- Second, I'd also ask, does doing this task significantly faster, cheaper, or more consistently using AI create substantial value?
- rather than analyzing employees tasks, consider also analyzing the tasks of your customers.

> Tips: 
> - GenAI, Supervised Learning, Unsupervised Learning, Reinforcement Learning, these are all the AI techniques.
> - So, when we think the potential of AI, we should think about the all AI techniques.

#### 1.2.1 augmention or automation

Now, the opportunity for generative AI could be either augmentation or automation. 

**augmention**

By augmentation, I mean, we can use AI to help a human with a task. 

So if we're not sure yet if the generative AI would give good answers, then recommending a response could speed up the people doing the work, but not fully automated, and this would be an example of augmentation.

**automation**

And automation would be if we have an AI system fully automatically perform a task.

What I see in many applications is that businesses will sometimes start with augmentation to maybe let a human double check or finalize the output before it is used. 

But then as you gain trust and gain confidence in the output of the generative AI, then the user interface can be adapted to make the process more and more efficient for humans and to then gradually shift to higher and higher degrees of augmentation and perhaps eventually, to full automation. 

### 1.3 Job vs task

I found that for many job roles, people have a mental picture of that iconic task that uniquely defines that job role. 

For example, computer programmers write code. Doctors maybe see patients. Lawyers go to court to argue court cases. 

And I think that when people think about AI opportunities, often it's instinctual to say, can AI do that most iconic role, or that most iconic tasks of that job? 

But I found that when we actually systematically analyze the tasks that a particular job is made up of, the best opportunities may or may not be that first initial instinct. 


And in many different job roles, the best potential for AI may not be the most obvious first task you might think of.

demo:
- [software developer task](https://www.onetonline.org/link/summary/15-1252.00)
- ![augment or automate customer task](/assets/images/llm-for-everyone/23-augment-customer-task.png)

## 2 AI Benefit

First, because AI is indeed a automated process, so it will:

- faster: save task time
- cheaper: save task cost

And second, some AI system can create more value:

- just like a recommendation system

Third, maybe most important, it may create new workflows. Let's have a look.

### 2.1 AI can create new workflows

Let's see a significant example:

![22-llm-create-workflow-intuition](/assets/images/llm-for-everyone/22-llm-create-workflow-intuition.png)

In this example, even though it seems like generative AI can be used just for cost saving to help the marketer be much more efficient in the process of writing and pushing copy. 

> We can also take advantage of this efficiency in a totally different way and write and push and:
> - test many more versions of the website copy.
> - This leads to other changes as well to the workflow of the marketer so that this way hopefully the marketer can deliver much more compelling marketing campaigns. 

This type of re-engineering of workflow is quite common once both generative AI or other AI tools are inserted into a system. 

Because when one task is automated or augmented, it often makes sense to **re-think what are the other tasks that need to be done together** in order to deliver a valuable work product.

## 3 teams to build GenAI

If you're building an LLM-based application, it's often possible to get started with a pretty small team. I definitely see companies start to experiment with even a one-person team, such as a software engineer who has learned some prompting. Or machine learning engineer who's learned a bit about prompting OMs.

But I've seen many other configurations also work well, such as a software engineer who's learned prompting and a product manager, or two generally enthusiastic people that know a bit about how to write software and they're willing to learn how to use these tools to build new and exciting applications. 

## 4 automation potential across sectors

The results from this video may be less directly actionable for a particular business, but maybe this will help you to think through and try to forecast some of the large macro changes that may take place over time.


So McKinsey had carried a study on the potential of AI automation with and without generative AI. And we re-plotted the McKinsey data to show the impact of generative AI only on automation, leaving out other forms of AI such as supervised learning. 

Some of the sectors impacted include, educator and workforce training, business and legal professions, STEM professionals and so on. 

And one remarkable thing about this data is that there are sectors that were not highly exposed to automation before generative AI. But with the rise of generative AI is now seeing a much greater potential of automation or augmentation. 

And so depending on what sector or sectors you either work in or work with, perhaps this type of analysis may give you a sense of what may happen in industries that you touch as well. 

![24-GenAI-influence-industry-sector](/assets/images/llm-for-everyone/24-GenAI-influence-industry-sector.png)

If you look at the top few lines on this graph, one theme that pervades both this as well as other studies is that, it looks like **a lot of the impact of generative AI, will be on knowledge workers**. Meaning, workers who generate value primarily through their knowledge, including their expertise, their critical thinking, and their interpersonal skills. 

This is in contrast to, say, workers that create value mainly from performing physical tasks rather than knowledge tasks. 


## 5 concerns about AI

### 5.1 whether it might amplify humanity's worst impulses.

- fine tune
- RLHF

### 5.2 job loss

#### 5.2.1 why Not a single one of my radiologist friends has lost their job to AI.

- First, interpreting X-rays turns out to be harder than it looked back then, though we are making rapid progress. 

- But second and more important, it turns out that radiologists do a lot more than just interpret X-ray images. According to O*NET, radiologists do about 30 different tasks, one of which is interpreting X-rays and other medical images, but they do many other tasks. And it has been difficult so far for AI to do all of these tasks at human level. 

#### 5.2.2 AI will bring more than it destroy

- Mind you, I don't mean to minimize the challenge of helping many people adopt AI or the suffering of the much smaller number of people whose jobs will disappear. Or our responsibility to make sure people affected have a safety net and an opportunity to learn new skills. 

- But every wave of technology, from the steam engine to a chassis to the computer, has created far more jobs than it destroyed. As I mentioned earlier this week, in most waves of innovation, businesses wound up focusing on growth, which has unlimited potential rather than cost savings. So AI will bring a huge amount of growth and create many, many new jobs in the process. 

#### 5.2.3 will AI kill us all

- Some were concerned about a bad actor using AI to destroy humanity, say, by creating a bioweapon.

- Others were worried about AI inadvertently driving humanity to extinction. Similar to how humans have driven many other species to extinction through simple lack of awareness that our actions could lead to that outcome. 

#### 5.2.4 summary

Finally, if we look at the real risks to humanity, such as climate change leading to massive depopulation of parts of the planet, or hopefully not the next pandemic. Or even much lower chance, but another asteroid striking the planet and wiping us out like the dinosaurs. 


So I know that there are different views on this right now. I think that AI will be a key part of our response to such challenges. 

But my view is that if we want humanity to survive and thrive for the next thousand years, AI increases the odds of us successfully getting there. 

Computers are already smarter in some narrow dimensions than any human. But AI continues to improve so fast that many people find it hard to predict exactly what it will be like in a few years. 

I think the root cause of some of these concerns, including extinction risks, is that many people are unsure when AI will reach Artificial General Intelligence, or AGI. Meaning, AI that could do any intellectual task that human can. 


## 6 AGI

I don't think there's any fundamental laws of physics that prevents us from ever creating AGI, which I think will actually be very valuable to human society. But we'll still need some significant technical breakthroughs to get there. 


One of the hard things about getting to AGI is that it benchmarks artificial intelligence against human intelligence. Artificial intelligence and biological intelligence have progress along two very different paths. 

## 7 Responsible AI

![25-dimensions-of-responsible-ai](/assets/images/llm-for-everyone/25-dimensions-of-responsible-ai.png)

One of the challenges of these dimensions, or these principles is that the implementation is not always straightforward.

For example, I think at least a couple thousand years now, humanity has been debating what is ethical and what is not ethical. 

There is unfortunately no clear mathematical definition of ethical versus unethical behavior. Although, of course, there are many clear cut cases as well. 

But that's why for individuals, organizations, even countries to adopt responsible AI, there are certain emerging best practices to hope, have the discussion and debate that will lead to better and more responsible decisions, even when sometimes the right thing to do could be ambiguous. 

For many projects, seeking a diverse set of opinions, as well as speaking with people that could be quite different than myself has allowed my team to understand better the impact of an AI system and led us to make better decisions.

![26-tips-for-responsible-AI](/assets/images/llm-for-everyone/26-tips-for-responsible-AI.png)


## 8 build-a-more-intelligent-world

### 8.1 artificial intelligence is cheap

Intelligence is the power to apply knowledge and skills to make good decisions.

We invest years of our lives and trillions of dollars on education, all to develop our ability to make better decisions. It costs a lot to feed, educate, and train a wise human being. 

So **human intelligence is expensive**. That's why today only the wealthiest people can afford to hire large amounts of intelligence by hiring people.

But unlike human intelligence, artificial intelligence can be made cheap. 

AI has the potential to give every individual the ability to hire intelligence at low cost, so that you no longer need to worry about that huge bill for visiting a doctor or getting an education and you can hire an army of smart, well informed staff to think things through for you. 

![27-intelligent-world](/assets/images/llm-for-everyone/27-intelligent-world.png)

### 8.2 AI problems will recede

AI is the new electricity with the potential to revolutionize all industries and all corners of human life. The fear of AI today is similar to the fear of electricity when it was new. 

Back then, people were terrified of electrocution or of electricity sparking devastating fires. 

Today, electricity still has dangers, but I don't think any of us would give up light, heat, and refrigeration for fear of electrocution. 

AI today still has flaws and there are cases where it will cause harm. But generative AI is an exciting leap forward in how much intelligence we can bring to the world. 

We're also improving the technology rapidly. As we continue to improve the technology and also continue to build up more use cases, AI will contribute to longer, healthier, and more fulfilling lives worldwide. 

As the technology improves, the problems of AI that alarm us today will recede. 

### 8.3 we need more intelligence

Looking beyond AI, the world has many problems such as climate change, pandemics, war, and many others, and many of these problems need urgent solutions. 

**To solve them, we will need all the intelligence,** including all the artificial intelligence we can master. 




