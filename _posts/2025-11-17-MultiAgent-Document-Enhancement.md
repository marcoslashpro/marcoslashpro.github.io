---
layout: post
permalink: multi_agent_document_enhancement
title: Multi Agent Document Enhancement
---

* TOC
{:toc}


START


One of the main features of **CareerKit** as it stands right now is the **multi-agent document enhancement** system(**MADE**) that allows for fine-grained tailoring of **CV** and **Cover Letter(CVL)** for specific, user-required job listings. Let’s dive deep into how this works.

## System Dependencies

In order to make this work, there are certain requirements that must be met, in order to satisfy the system dependencies:

- **Job Listing & Company Information**: As it should be obvious, we need to know what job the user is interested in, what is the job about, what are the job requirements and who is the company behind this job. In **CareerKit** this is in itself handled by a couple of different autonomous agents:

    - **Listing-formatter**: whose only goal is to read the job listing and produce structured output that can be then used by the MADE system. This information will be very valuable in order to be able to align user information to the specific job listing and is made of fields such as the job requirements.

    - **Company-researcher**: Which is in and of itself a **sequential multi-agent** system that runs **deep-research** on the company in order to extract as much valuable information as possible, and present it in a friendly markdown format. The final research contains important fields such as company values and company policies, that can be later used by the **MADE** system to align the user’s CV not only to the specific job listing, but rather to the company itself.

- **User Information**: We need to have access to the user’s information in order to be able to know how the user’s skills, education and experience align with the user-provided job listing. In **CareerKit** we have access to this information in two ways:

    - The user can manually fill out some forms, in which he can describe work experience, or a project that he has worked on, skills gained along the way and much more; or

    - The user can offload this process to another **autonomous agent**, by only uploading an existing curriculum and letting the system handle the rest.

### Enhancing Input Quality

As we know, **LLMs**, the brainpower behind these agents, are **stochastic machines**, meaning that even given twice the same input they might not give back the same output. There are techniques that can be used in order to reduce this probability of this scenario, which are in use by **CareerKit** in order to guarantee reliable output(and consequently input to the **MADE** system). 

These techniques include:

- Lowering the model’s `temperature`, which represents  the model’s probability of choosing a lower-ranked output token; and

- Lowering a special parameter named `top-p`, also known as *nucleus-sampling*, which controls the output diversity of the model.

However, this does not make the system immune to another known factor of **LLMs**, usually described in the mainstream as: 

> *Garbage in, garbage out*
> 

This is to say that, if the information that are given to the **MADE** system are not very **descriptive**, **badly articulated**, or **irrelevant**, the system will always struggle to produce reliable and relevant output.

For this reason, in the future, **CareerKit** will also implement a **smart-enhancer agent**, effectively acting as a middle man that, on user’s request, will enhance specific sections of the user’s information, rendering trivial the process of feeding valuable information to the **MADE** system that will, in turn, produce more valuable outputs; stay tuned for that!

Now that we have the dependencies out of the way, let’s give a look at the system architecture.

## MADE System Architecture

The multi-agent flow used for this system is known as **orchestrator flow,** which can be simply visualized as:

![image.png](/assets/images/multi_agent_document_enhancement/orchestrator_flow.png)

Here we see that we have a **manager**(or **orchestrator** if we’re being fancy) agent that has authority over **n** sub-agents that report directly to him.

I have chosen this agentic architecture because it makes it very easy to offload tasks and also allow to run them in parallel, let’s see how this agentic flow is used in **CareerKit**’s **MADE** system.

### Manager Agent

The **manager agent** is the brainpower of the **MADE** system, being the only agent that has **thinking capabilities**.

This allows it to reason on the given inputs, and compare user information against the job listing and company information in order to determine what should be **included**, what should be **highlighted** and what should instead be **left out**.

He then proceeds to simultaneously kick off both sub-agents in **parallel**, by giving to each one of them a very descriptive task, containing: **useful guidelines** on what to include, **why** and **how**.

Various techniques here are in place in order to guarantee that no information is fabricated on the spot only to satisfy the specific job listing requirements, **contextual grounding** becomes crucial. By **contextual grounding** I refer to a known technique used in order to reduce model’s hallucination. In order to know more about this, please refer to [this](https://docs.claude.com/en/docs/test-and-evaluate/strengthen-guardrails/reduce-hallucinations) **anthropic** guide on the topic. In the **MADE** system context, this technique is being put in place by forcing the model to explicitly include **information context**(the user’s information) when delegating tasks to sub-agents.

By having the model directly refer to the initial information, I have noticed a positive **95% drop in hallucinations**. 

### CV & CVL Enhancer

These guys are the operational part. Each one of them is in charge of a single document and their only responsibility is to iterate over it by satisfying the tasks that has been given to the them by the manager agent.

They only respond when the **manager agent** calls them, and they produce, or modify, parts of the document they are assigned to and they also output a **change summary** that is returned to the **manager model** that can be used to validate the success of the iteration.

Nice! But how does this *actually* work? Let’s dive in!

## MADE System Ops

A first, naive implementation of the system had the sub-agents output a full draft on every iteration. This was incredibly inefficient for different reasons, mainly because, after a couple of iterations the context of the manager would be completely **bloated**, token costs would be **sky-high** and information would be loss due to **context overflow**.

I saw that this was **not** maintainable and therefore decided to steer in a different direction. 

In order to make this improve, I had to make sure of two things:

1. **No** context bloating; and
2. **No** incredibly long outputs on every iteration.

Once identified and clearly stated the problems, the solution became trivial:

1. **Do not** fight the context, rather **embrace it**. By this I mean, make the drafts part of the context, but only include the latest and most relevant iteration of each, in order to avoid context bloating;
2. Allow each sub-agent to only output the **relevant changes** to be made and **programmatically** update the draft, this will save you a lot in the long run, trust me.

With this in place, the system can be visualized from birds-eye view like this:

![image.png](/assets/images/multi_agent_document_enhancement/system_birds_eye.png)

It can be read like this:

- The **manager** examines the context, containing user’s information, company research and document drafts(if any);

- Once the **manager** has a clear idea of what needs to be done, it delegates tasks to the sub-agents;

- The sub-agents are kicked off in parallel, they both execute their given task;

- The documents are updated with the output of their respective agents and a live feedback of the changes are given back to the **manager;**

- Once the manager deems the documents ready, they are used to populate real `.docx` files which are then prepared and shipped to the frontend through a `websocket` connection;

- On the frontend the user can then either accept the given drafts, or reject them and provide feedback that can be used by the manager in order to polish the drafts;

- Loop until a final product is reached or the connection is close.

This keeps the costs low, the system fast(thanks to parallelization) and the user will not receive a black box at the end, but he is a crucial part of the system that dictates if something is ready or not, and in the latter case, how it can be improved.

A major challenge of this system has been improving it. It felt like a lot of the times the system would get worst rather than getting better after every change, but how can we actually tell, while iterating over the system, if we’re making improvements or not?

## System Reliability and Evaluation Metrics

The only reliable way to know if by modifying your system you are making any progress or not is by setting up valuable and accurate evals. I know it is complicated, and it was probably the most complicated part of the system and still a WIP actually, but being able to run the MADE system with mock company and user information, pass the run output to a `LLMJudge` and have a detailed `JSON` output that looks something like this:

```python
[
	{
		id: case-name,
		duration: seconds,
		input_tokens: int,
		output_tokens: int,
		total_run_tokens: int,
		input_tokens_cost: Decimal,
		output_tokens_cost: Decimal,
		total_run_cost: Decimal,
		is_hallucinated: bool,
		evaluation_report: str
	}
]
```

Is a pain to setup but offers you some real insights on how the system is behaving and if it is actually improving at all. Without even considering the fact that this can be easily wired up in your production system and allows you to have contextually grounded insights from your live application. Very much worth the hassle, if you were asking me. Look, I do not know how you build your agentic applications, but if you are interested in this concept please give a look at [`pydantic-ai`](https://ai.pydantic.dev/) and [`pydantic-eval`](https://ai.pydantic.dev/evals/).

Let me know if you’d like to hear more on the topic and I would be happy to dive a lot deeper!

## Conclusion

I want to stress that it is not by magic that a system improves, but it is by consistency, thoughtful iteration and reliable evaluation metrics that each change made leads to an improved product, rather than disastrous abomination.

That’s it, thank you for reading, and if you have not already tried **CareerKit** I invite you do so [here](http://your-careerkit.com).

I would also like to say that I am currently on the job hunt myself, and if you have enjoyed what you read and think that there might be something valuable here, do not hesitate to contact me at [**tambascomarco35@gmail.com**](mailto:tambascomarco35@gmail.com).

If you guys have tried **CareerKit**, feel free to use the same email as above to send feedback, good or bad, they’re always welcome.

Oh last thing, let me know in the comments below if you would like me to deep dive into evaluations for AI systems, I would love to do so!

Until next time!