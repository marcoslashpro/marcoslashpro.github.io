---
layout: post
permalink: simple_rl_roadmap
title: A Simple RL Roadmap
summary: "A small summary of the post, it might actually be longer than a small summary, it could be as long as one wants to be completely honest, it does not matter to me nor to the rendering engine, we all are happy. Me of the future can write whatever he wants, me of today feels like a hacker, and the program... just does what he needs to do, until AI becomes our program and then we are doomed, but hey! I hallucinate more than ChatGPT!"
---

**Reinforcement Learning** (**RL** for short) is a truly fascinating and rapidly evolving field of **Machine Learning**. From agents beating world champions at complex games like **Go and Atari**, to enabling **autonomous intelligent machines** to learn and **interact** with the real world, the possibilities RL brings to the table are **incredible** (and, yes, somewhat scary!). But what exactly is it?

The most famous comparison used to explain RL is often teaching **a dog a new trick**. In order to get your dog to learn the trick you want, you break it down into smaller steps or _“sub-tricks.”_ You slowly guide them, giving feedback each time they do something right, until they can perform the full trick you intended.

**At a high level**, **RL** works in a similar way. You _“instruct”_ an agent with a goal or _“trick”_ it needs to learn, and you provide it with small _“treats”_ (**rewards**) every time it takes an action that leads towards achieving that goal. Through this process of **trial, error, and reward**, the agent learns the optimal sequence of actions to take in different situations… just the thought of this learning process in machines is **amazing**!

So, **why is RL considered a key to the future of AI**? Because it’s a powerful paradigm for tackling problems where **decision-making** is dynamic and requires adapting to changing circumstances. From autonomous systems like self-driving cars and robotics to complex optimization problems in finance, logistics, and so _much more_, RL offers a **framework for creating intelligent agents** that can learn and operate effectively in complex, unpredictable environments. As we build more sophisticated systems that need to interact with the real world, RL’s ability to **learn directly from experience** becomes incredibly valuable.

But, as they say, **not all that glitters is gold**.

While RL is a **core part of modern ML**, it can feel relatively new and complex when you’re just starting out, and the entry level is often considered **quite high**. There are significant challenges involved, such as:

- **Environment Creation**: Building the right simulated environment where an agent can learn effectively can be extremely complex.

- **Reward Structuring**: Designing a clear and effective reward system that guides the agent to learn exactly what you intend is a very tough challenge.

- **Integration Complexity**: Making all the different components — the agent, the environment, the learning algorithm — fit together and work correctly can feel daunting.


If you’re a newbie curious about diving into this exciting field and want to build a solid foundation in the **basic theory**, I’ve put together a simple roadmap based on **my own journey**. It covers the fundamental concepts I found crucial to understand when first getting started.

I created this **roadmap** because I wanted to put something out there that I wish I’d found myself when I was learning. It’s designed to guide you through the initial steps of **understanding RL**.


> Special thanks to [this OpenAI paper](https://spinningup.openai.com/en/latest/), [this youtube video](https://youtu.be/sJIFUTITfBc?si=hXFhfAQFtQF3M9fI)
>


### MDP (Markov-Decision-Process)

This is the base of any kind of **RL algorithm** (**model-based** or **model-free**) is the Markov-Decision-Process. RL *problems* are *modeled* as MDPs. Algorithms then *solve* these MDPs. Model-based algorithms explicitly learn or use the MDP's transitions and rewards; model-free algorithms learn optimal behavior without explicitly learning these models.

It provides a **structure** to understand the **sequential decision-making problem under uncertainty**(which might come from the **environment**, the **agent decision-making process**, or **both**.) 

The components that make up a **MDP** are:

- **Markov Property:** In which we state that the conditions of the future states are given **only** by the current state and **not** the previous ones.

- **Agent Decision Making:** The agent, given a policy, takes an action that aims to maximize the performance, quantifiable thanks to either a **Value function,** or a **Quality function.**

The core concepts of an **MDP** are represented in this tuple: $$(S,A,P,R)$$:

- $$S$$: The **State** ($$s_t$$) represents the information given to the agent at a given moment. This information, in a **MDP must** contain all of the information relevant to predicting the future.

- $$A$$: The **Action** is the *mechanism* that allows the **agent** to transition between **states** of the **environment**.
The **action to take** is derived from the **a set of actions**, which represents all of the possible actions that can be taken from a given state $$s_t$$, represented as $$A(s_t)$$. The action taken at a given state $$s_t$$ is represented as $$a_t$$ and it is sampled by the **policy** of the agent represented as $$\pi$$. But we’ll talk more about policies later.

- $$P$$: Is the **transition probability function.** The **transition model** of the environment.
It represents the probability of transitioning to the next state $$s^\prime$$, given the current state $$s_t$$, and a taken action $$a_t$$.
    
    $$
    P(s^\prime\|s_t,a_t)
    $$
    
    This **transition model** might be given to the agent, or, if not, the agent will have to learn it, in order to predict the likelihood of a next state $$s_{t+1}$$, given the current state $$s_t$$ and a taken action $$a_t$$. The probability of the next state will therefor be:
    
    $$
    s_{t+1}\sim P(\cdot\|s_t,a_t)
    $$
    
- $$r$$: The **reward function**. This is **Reward model.** It describes the **reward** of performing an action $$a_t$$, in a state $$s_t$$. It can be expressed as:
    
    $$
    r=R(s_t,a_t,s_{t+1})
    $$
    
    The **reward model** $$R$$ accounts for the next state in which we will end up ($$s_{t+1}$$, given by the **transition model** described above), by taking an action ($$a_t$$) at a state ($$s_t$$).
    

So, coming out of what we have just learned, it should be clear that a **MDP RL problem should have:**

$$
P(s^\prime\|s_t,a_t)=\text{Pr}(s_{t+1}=s^\prime\|s_t=s,a_t=a)

$$

- This, as discussed before, is the **Transition Model**, which gives us the probability of moving to a next state ($$s^\prime$$) given the current state ($$s_t=s$$) and a taken action ($$a_t =a$$).

$$
R(s_t,a_t,s^\prime)
$$

- The **Reward Model** instead gives us the reward of transitioning to that new *realized* state $$s^\prime$$.

### Goal

I believe that stating the goal of RL will help us better understand how the different components fit together. We can briefly describe the goal as:

> *Find a **policy** that **maximizes** the **expected cumulative reward** (**return**) given by taking the **best actions** at any **given state**.*
> 

I know it might not make so much sense now, since we have introduced new components which we have not discussed before(or maybe just briefly), but this high-level map is the road that we have to follow, not let’s find the gear we need in order to traverse it.

To express it mathematically:

$$
\pi^*=argmax_\pi J(\pi)
$$

Where:

- $$\pi^*$$: Is the optimal policy

- $$J(\pi)$$: Is the value that we are trying to maximize, the **expected cumulative reward** (or **expected return**, for short).

We can then see, that the **policy** ($$\pi$$) that maximizes the value of the **expected cumulative reward** ($$J(\pi)$$) will be our ***optimal* policy** ($$\pi^*$$).

### Cumulative Reward (return)

The follow-up question is:

> *How do we calculate the expected cumulative reward (return)?*
> 

To answer that question, let’s first clarify the types of **return:**

- **finite-horizon return**
    
    Which can be mathematically stated as:
    
    $$
    G_t=R_{t+1}+R_{t+2}...R_{T}=\sum_{k=t+1}^TR(s_k,a_k,s^\prime)
    $$
    
- **infinite-horizon return**
    
    $$
    G_t=R_{t+1}+R_{t+2}+...=\sum_{k=t+1}^\infty R(s_k,a_k,s^\prime)
    $$
    

Notice the time step index $$k=t+1$$, to understand it, imagine: We state at a first state $$s_t$$, from which we take no reward, which we start to obtain at the next time index, there for $$k$$(the current time index)is equal to the time index plus 1.


To put these simply, we can say: 

> The **return**, or **cumulative reward** is equal to the sum of all of the **rewards** ($$R(s^\prime,s_t,a_t)$$), in a **time-horizon** which can be either **finite**($$t$$ → $$T$$) or **infinite** ($$t$$ → $$\infty$$)
> 

**Furthermore**, a common choice in **RL** is to **discount the future rewards.**

To **discount the future rewards**, we need to introduce a new **parameter** $$\gamma$$.

This parameter is a small positive number (often between 1 and 0, or $$\in (0,1)$$), and it’s used to both make the equation **mathematically easier**, and also it has some conceptual sense, “**Money now is better than money later”**.

If we wanted to introduce the discount factor $$\gamma$$, we would:

$$
G_t=\sum_{k=t}^{\infty}\gamma^kR(s_{t+k},a_{t+1},s_{t+k+1})
$$

NOTE: The value of the **discount factor**, **decreases** the longer in the future the **reward** is.

Now we know how to calculate the **actual** **cumulative reward (return)**, Let’s pluck this new obtained definition in the **goal** stated before:

$$
\pi^*=argmax_\pi J(\pi)=argmax_\pi E_\pi [G_t]
$$

If you’ve followed through, this should match even better the definition given before, which, to freshen up, is:

> *Find a **policy** that **maximizes** the **expected cumulative reward** (**return**) 
(*$$E_\pi[G_t]$$) *given by taking the **best actions** at any **given state**.*
> 


Now, after identifying the **models,** and the **goal** of **RL,** we can march forward towards the next step which is:

> *Finding the **optimal policy** that maximizes the **expected cumulative reward***
> 

### Policy

The role that the **policy** plays in **RL** is best described as follows:

> The **policy** is the **agent’s brain**. It instructs the **agent** on what **action** to take in order to **maximize** our **expected return.**
> 

But, what is a **policy?**

Simply put, the role of a **policy** is to map a given state ($$s$$) to an **action** ($$a$$).

Two main types of **policies** are frequently used:

- **Deterministic**
    
    When working with a **deterministic policy,** we can only map **one action** to any given state ($$s$$).
    
    $$
    a=\pi(S_t=s)
    $$
    
- **Stochastic**
    
    When a **policy is stochastic** instead, it maps the probability of **every action** to any given state ($$s$$).
    
    $$
    \pi(a\|s)=\text{Pr}(A_t=a\|S_t=s)
    $$
    

As for every time that we perform model training, **we have no idea of what the *optimal* policy is**, and finding the ***optimal* policy** ($$\pi^*$$) is exactly the **goal of the algorithms** that we are going to introduce later, based on the the **oh-so-scary**: **Value functions.**

### Dynamic Programming

For the sake of the argument, let’s imagine that we have to calculate the **expected cumulative reward** of a **policy** on a very simple game, i don’t know, let’s say…**chess**.

In the wonderful game of chess there are **$$10^{80}$$ pieces-to-board** combination, which is a huge number. *How could we ever calculate all the possible state-action pairs and their rewards*? It would be **too computationally expensive**, and not feasible at all to train an algorithm on given circumstances. And this is where the magic of **dynamic programming** comes into play.

From WikiPedia:

> *In both contexts [dynamic programming] it refers to simplifying a complicated problem by breaking it down into simpler sub-problems in a recursive manner. While some decision problems cannot be taken apart this way, decisions that span several points in time do often break apart recursively. Likewise, in computer science, if a problem can be solved optimally by breaking it into sub-problems and then recursively finding the optimal solutions to the sub-problems, then it is said to have optimal substructure.*
> 

I believe we can clearly see how this would come in useful for us working on a chess problem. Instead of calculating all of the possible combinations on the board, we could instead break the huge combinations of move into smaller, more manageable sub-problems, and then tackling those recursively.

How do we do this? Well, this is what the **Bellman Equations are for.**

### Value Functions and the Bellman Equations

## Value functions

**Value functions** are **the only way** that we have in order to **evaluate** a given **policy.**

Now, what do we mean by “**evaluating**”?

To evaluate a given policy we have to calculate the **expected cumulative reward**, starting from a specific state $$s$$, and then acting accordingly to the policy forever after. These are called **state-value functions.**

In some case, we might also want to not only take into consideration a specific starting state, but also an **initial action** $$a$$, these are called **action-state-value functions.**

It is very important for us to make another distinction:

### On-policy value functions Vs Optimal value functions

We might use some **value functions** in order to understand how good a given **policy** is, and we might sometime use other **value functions** in order to understand what is the ***optimal*** **policy**.

- **On-policy value functions**
    
    These are the **value functions** that allow us to **evaluate** a given policy, just like the one described above.
    
- **Optimal value functions**
    
    These, on the other hand, help us understand what the ***optimal* policy** is in a given **environment.**
    

These functions, work as a wonderful tool for **evaluating** policies, but, as they are, they cannot be computed! 

To make this fit in our brain let’s look at an example:

We want to train an algorithm on…, let’s say… **map creation**.

The algorithm will traverse the map, given by a **graph,** with **nodes** pointing to the **intersections**, and **edges** connecting the nodes representing the **streets.**

We want the algorithm to start from a **node** $$A$$, transition to a **node** $$B$$, and so on, so forth, until it reaches the end of the map at a **node** $$Z$$.

Let’s take, for example **node** $$B$$.

It opens up a fork, **node** $$C$$ goes straight to **node** $$Z$$, following a very long straight **edge**, while **node** $$D$$, goes zigzag all over the city.

We want to know the best way to get from **node** $$A$$ to **node** $$Z$$, but, in order to know what the best way is, we have to calculate the value of being at the **node after the one in which we are**, which is a recursive property that the algorithm should have in order to allow it, and that is exactly what these **value functions** are missing.  

We could go over one more example, this time, again, chess.

I’ll make it quick and cut right to the chase, how could we ever compute all of the possible **pieces-to-board** combinations?

Think about it…


## Bellman Equations

Well, this was the piece of genius, that moved us from conceptualizing these algorithms, over to making them feasible.

### What is solves

1. **Recursive Iteration**
The Bellman equation allows an algorithm to use the *estimated value* of the next state to *update* the estimated value of the current state. 
This is called **bootstrapping**– *using an estimate to update an estimate*. This process leverages the recursive structure to propagate value information back through the state space.
2. **One big, or a lot smalls?**
    
    Well, referring back to the chess example, and as we’ve already discussed in the **dynamic programming** section, we can, instead of tackling one **huge problem** altogether, break that problem down, not having to calculate all the **expected returns** for every one **piece-to-board** combination, but we can instead take into account as-many-as-we-want-and-can states, and calculate the **rewards** from there. 
    

Let’s now bridge everything together, and let’s give a look at how **dynamic programming** and the **Bellman equations** interact with the **value functions**


# State-value functions

This is going to be a lot, but I hope I can make this somewhat easier to understand.

I will tackle this, first by introducing the **value function**, and then by inserting the right **Bellman equation,** expanding it, so that it should make more sense, and then describing what every piece does, let’s go.

## On-policy state-value function

Let’s start building some intuition starting from the **state-value function**:

$$
V^\pi(s)= E[G_t\|S_t=s]
$$

Let’s break it down:

- $$V^\pi(s)$$: the expected return when starting in state $$s$$ and following policy $$\pi$$

- $$E[G_t\|S_t=s]$$: This is the **expected cumulative reward** given a starting state $$S_t = s$$.

### Bellman Expectation Equation for $$V^\pi(s)$$

$$
V^\pi(s)=E_\pi[R(s,a,S_{t+1})+\gamma V^\pi(S_{t+1})\|S_t=s]
$$

Expanded:

$$
V^\pi(s)=\sum_{a\in A}\pi(a\|s)\sum_{s^\prime\in S} P(s^\prime\|s,a)[R(s,a,s^\prime)+\gamma V^\pi(s^\prime)]
$$

Let’s break it down:

- $$\sum_{a\in A}\pi(a\|s)$$: This represents the action taken given the state in which we currently are. It is sampled by the policy, and the action taken is contained in the set of all possible actions $$A$$;

- $$\sum_{s^\prime\in S} P(s^\prime\|s,a)$$: This represents the next state in which we’ll transition after taking an action $$a$$ at the current state $$s$$. It is given by the transition model that we described in the **MDP** section. The state in which we’ll transition, is contained in the set of all possible state $$S$$;

- $$R(s,a,s^\prime)$$: This is the reward model, that we have discussed in the **MDP** section;

- $$\gamma V^\pi(s^\prime)$$: This is the recursive part of the algorithm, it allows us to calculate the **future expected value**, by using the same value function that we are using now, but starting from the next state $$s^\prime$$.

I hope a lot of the pieces that we have saw before are now starting to click together, and many of the things we have just seen will be used in the next **value function** as well, so I will avoid describing them at all states.

## Optimal state-value function $$V^{\pi^*}(s)$$

$$

V^*(s)=max_\pi V^\pi(s)=~E_{\pi^*}[G_t|S_t=s]
$$

There are somethings that we must discuss in order to make sense:

- $$V^*(s)=max_\pi V^\pi(s)$$: This states that in order to find the **optimal state-value function** $$V^*(s)$$, we must find the policy $$\pi$$ that maximizes the **value** calculated by the **state-value function** that we saw before;

- $$E_{\pi^*}[G_t\|S_t=s]$$: This is the usual expected return, but the: $$E_{\pi^*}$$ means that we are expecting to act according to the optimal policy $$\pi^*$$.

### Bellman Optimality Equation for $$V^{\pi^*}(s)$$

$$
V^{*}(s)=\max_{a \in A(s)}~E[R(s,a,S_{t+1})+\gamma V^{*}(S_{t+1})\|S_t=s, A_t =a]
$$

Expanded:

$$
V^*(s)=\max_{a \in A(s)}\sum_{s^\prime \in S}P(s^\prime\|s,a)[R(s,a,s^\prime)+\gamma V^{*}(s^\prime)]
$$

I hope you find a lot of similarities with the **Bellman expectation equation** described above, the main difference is that we are calculating the **expected return** based on the optimal policy, we can see it here:

- $$\max_{a \in A(s)}$$: This means, that the action that we’re going to take from the given state $$s$$ is going to be the action that transitions us to the next best possible state, maximizing the **expected return**.

- $$V^{*}(s^\prime)$$: Here we can see the recursive property of the algorithm again, by executing the same function but starting from the next state.

# Action-state-value functions

## On-policy action-state-value function

$$
Q^\pi(s,a)=E[G_t\|S_t=s,A_t=a]
$$

Just like the **on-policy state-value function** right? Yes, but this time we are taking into account an predetermined initial action $$A_t=a$$, and acting according to the policy forever after.

### Bellman Expectation Equation for $$Q^\pi(s,a)$$

$$
Q^\pi(s,a)=E_\pi[R(s,A_t,S_{t+1})+\gamma Q^\pi(S_{t+1},A_{t+1})\|S_t=s, A_t=a]
$$

Expanded:

$$
Q^π(s,a)=∑_{s^\prime \in S}P(s^\prime∣s,a)[R(s,a,s^\prime)+γ∑_{a^\prime \in A(s^\prime)}\pi(a^\prime∣s^\prime)Q^π(s^\prime,a^\prime)
$$

Also the bellman equation looks very similar to what we saw already, but:

- $$∑_{a^\prime \in A(s^\prime)}\pi(a^\prime∣s^\prime)$$: We are now taking into account the action we will take from the next state given by the policy, over the set of all possible actions that can be taken from the next state.

- $$Q^π(s^\prime,a^\prime)$$: This shows, again(just because I love it) the recursiveness of the algorithm.

## Optimal action-state-value function

$$
Q^{\pi^*}(s,a)=\max_\pi Q^\pi(s,a)=E[G_t\|S_t=s,A_t=a]
$$

We see the same that we’ve seen in the **optimal state-value function**, but this time we are taking into account the **policy** that maximizes the **value** given by the $$Q^\pi(s,a)$$ function: $$\max_\pi Q^\pi(s,a)$$

### Bellman Optimality Equation for $$Q^{\pi^*}(s,a)$$

$$
Q^∗(s,a)=∑_{s^\prime \in S}P(s^\prime∣s,a)[R(s,a,s^\prime)+γ\max_{a^\prime∈A(s^\prime)}Q^∗(s^\prime,a^\prime)]
$$

The last one!

END

* TOC
{:toc}