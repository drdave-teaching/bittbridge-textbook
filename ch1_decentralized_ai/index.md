# Chapter 1 — What Is Decentralized AI?

:::{admonition} 🎥 The lectures behind this chapter
:class: seealso dropdown
This chapter is the book version of the four conceptual **deAI** lectures. No code yet — just the ideas you need before you touch a terminal.

- *deAI #1 — Why decentralized AI matters*
- *deAI #2 — Subnets, miners & validators (the school metaphor)*
- *deAI #3 — The BittBridge team & the subnet workflow*
- *deAI #4 — The energy-forecasting task & the leaderboard*

Optional deep dive in the guide: [`docs/guide/incentive-mechanism.md`](https://github.com/bittbridge/bittbridge/blob/main/docs/guide/incentive-mechanism.md).
:::

You almost certainly use AI as a consumer: you ask a chatbot a question and it gives you an answer. That's the tip of the iceberg. The interesting question for this project is not *how do I use a model* but *who gets to build and own the intelligence in the first place* — and what happens when that right is opened up to everyone.

## 1.1 Why "decentralized" is the whole story

Every major technological revolution has rhymed: a few players build the system, and then they **control the access, the resources, and the benefits**, while everyone else becomes a user. Large-scale AI has looked exactly like this — training a frontier model has meant multi-million-dollar budgets, enormous energy bills, and armies of engineers inside a handful of big companies.

Decentralized AI proposes a different shape. Instead of one company — instead of Google, Microsoft, OpenAI, or Anthropic — imagine **thousands of independent contributors** each producing models, computation, and intelligence, and **competing against each other** to do it better. A milestone that made this concrete: the first large language model trained entirely on **decentralized resources**, with no single owner footing the bill. Proof that the shape is possible.

```{admonition} Key idea — merit, not connections
:class: tip
The organizing principle of decentralized AI is brutally simple: **the strongest contributors get rewarded, on merit.** Not based on who you know, not based on your résumé — just whoever produced the best performance on the task. Hold onto this; the entire incentive mechanism in §1.5 exists to make this sentence literally true and automatic.
```

## 1.2 Bittensor and subnets

The platform we build on is **Bittensor** — a decentralized-AI network that runs on a **blockchain**. You can think of the blockchain as a *public dataset that no single person or entity controls*: it is verifiable, and it is where contributors compete and get paid.

Bittensor is not one monolithic AI. It is a large network broken into many small pieces called **subnets**.

```{admonition} Mental model — a subnet is a startup
:class: important
A **subnet** is like a *startup inside the bigger network*. Each one has its own mission, its own utility, and its own way of measuring success — a complete little ecosystem living inside the larger one. Bittensor today has 125+ subnets: compute marketplaces, deepfake detectors, 3-D asset generators, and many more, each with its own goals and architecture. You can browse them on [taostats](https://taostats.io). **BittBridge is one of those subnets — number 183 — and its mission is forecasting energy demand.**
```

A few real subnets, to make it concrete: **Targon** (subnet 4) is a GPU marketplace where miners rent out their machines — often cheaper than AWS or Azure; **BitMind** (subnet 34) detects AI-generated images and deepfakes using *generator* and *discriminator* miners; **404-GEN** turns prompts into downloadable 3-D assets. Different missions, same machinery underneath. That machinery is what you're about to learn.

## 1.3 Miners and validators: the school metaphor

Inside every subnet there are two roles, and the cleanest way to understand them — straight from the Bittensor team — is a school:

```{list-table}
:header-rows: 1

* - Role
  - In a school…
  - In the subnet…
* - **Subnet owner**
  - the **principal** — sets the mission, the rules, the grading criteria
  - defines the task and the scoring rules (for us: forecast New England energy demand)
* - **Validators**
  - the **teachers** — hand out tasks, grade the work, catch cheaters
  - send prediction requests, score the answers against reality, set on-chain weights
* - **Miners**
  - the **students** — solve the problem, compete for the best grade
  - **this is you** — run a model that answers the validator's request
```

The principal sets the test, the teachers grade it and make sure nobody cheats, and the students compete for the best score. In this project **you are a student/miner**: the validators are already deployed and maintained by the team, so your entire job is to build a miner that answers well.

## 1.4 The task: forecasting New England's energy demand

Here is what your miner is actually asked to do, every round:

1. A **validator** sends a request: *"What will electricity demand be at this future timestamp?"*
2. Your **miner** runs its model and returns a single number — a **point forecast** (e.g., "12,000 MW").
3. The validator **waits** until that timestamp arrives and the *actual* demand is published.
4. It compares your prediction to reality, computes a **reward**, and writes it to the blockchain.

The data that makes this possible:

- **Training data** — the past. Historical load plus weather, pulled automatically from **ISO New England**'s API and engineered into a tidy table (timestamp, total load, weather features).
- **Test data** — the future. The same columns, but the weather is a **forecast**, pulled from the **Iowa Environmental Mesonet**. In production your miner receives *future* rows and must predict their load.

```{admonition} Key idea — lag features look backward from the future
:class: tip
Your test rows live in the future, but if you engineer **lag features** (e.g., "demand two hours ago"), those features still reach back into the *past* training data. This is the single most important modeling idea in the project: the present is forecast, but the recent past is known.
```

## 1.5 The incentive mechanism: turning accuracy into rewards

This is where "rewarded on merit" becomes math. Validators score each miner with **absolute percentage error** between prediction and actual:

$$
\text{error}_i = \frac{\lvert \text{prediction}_i - \text{actual} \rvert}{\lvert \text{actual} \rvert}
$$

A small error should mean a big reward, and the relationship should be smooth, so error is converted to a score with **exponential decay** — every valid prediction earns *something* positive, but accuracy is rewarded steeply:

$$
\text{score}_i = e^{-k \,\cdot\, \text{error}_i}
$$

The validator normalizes everyone's scores into a **weight vector**, submits it to the blockchain, and **Yuma Consensus** aggregates the weights across all validators before **emissions** (TAO rewards) are distributed to miners. The same scores drive the public **leaderboard**.

```{admonition} The reward pipeline, end to end
:class: note
request → miners predict → actual arrives → percentage error → exponential-decay score → normalized weights → Yuma Consensus → emissions to miners → leaderboard updates

You don't have to implement any of this — the validators do — but understanding it tells you exactly what to optimize: **lower percentage error, consistently.** Full details in [`docs/guide/incentive-mechanism.md`](https://github.com/bittbridge/bittbridge/blob/main/docs/guide/incentive-mechanism.md).
```

## 1.6 What the validator looks like in practice

Behind the scenes, the validator is a running machine that constantly collects miners' predictions, waits for ground truth, and grades. For each miner UID it logs a prediction at a timestamp; when the actual load arrives it computes the reward (a prediction of 10,340 against an actual of 10,353 is excellent; 9,500 is not), and emits a matrix of **weights and scores per UID** to the blockchain. The closer your number, the bigger your slice.

## 1.7 So what are you actually doing here?

Look at this as **experiential learning** with real stakes. Over the next three chapters you will:

- stand up a **Google Cloud** virtual machine and live in the **Linux terminal**,
- create a **Bittensor wallet** and register on subnet 183,
- deploy a model into **production** and watch it on a **live leaderboard**,
- and iterate — because a good production model on the first try is rare. You won't just be the **data scientist**; you'll be the **MLOps engineer** and the **data engineer** too.

That full-stack experience — model, deployment, monitoring, iteration — is the real prize. Your golf-cart-simple linear regression will go live in the same arena as everyone else's, and the leaderboard will tell you, in public, every five minutes, how you're doing. Let's go build it.
