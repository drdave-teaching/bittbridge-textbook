# Preface

Welcome to the **book edition of the BittBridge final project** — the decentralized-AI capstone for **OPIM 5509** at the University of Connecticut.

Most of this course is about building models that live on *your* machine. This project is different: you are going to take a forecasting model, put it on a virtual machine in the cloud, register it on a **public blockchain**, and let it compete — live, against your classmates and against strangers around the world — to predict the **electricity demand of New England**. When your model is good, it earns rewards. When it's bad, the leaderboard says so, in public, every five minutes. That feedback loop is the whole point.

This book is not a dry reference manual. It's the project as the **BittBridge team actually teaches it** — casual in voice, serious in content, and **relentlessly hands-on**. It is built from two things woven together:

- the **lecture transcripts** recorded by Dmitrii Tuzov and the BittBridge team (Dmitrii, Niharika, Faeze, Ben, and Matt), which supply the narrative and the intuition, and
- the open-source **[`bittbridge/bittbridge`](https://github.com/bittbridge/bittbridge)** repository — the actual subnet codebase and step-by-step guide — which supplies the commands, the code, and the ground truth.

Wherever the lecture says "run this command" or "open this notebook," this book links you straight to the real file in the guide, so you can read the explanation and run the implementation in one place.

## Who this book is for

You're a graduate student (or a practicing analyst) who can already read a dataset into pandas, split it into train and test, fit a `RandomForestRegressor` or a `DecisionTreeRegressor`, and say something intelligent about the result. You do **not** need to know anything about blockchains, crypto, Linux, or the cloud — we build all of that from zero. If you've never opened a terminal in your life, you're exactly who we wrote this for.

## How the book is organized

```{tableofcontents}
```

- **Chapter 1 — What Is Decentralized AI?** The concepts, with no code: why decentralized AI matters, what **Bittensor** and **subnets** are, the **miner/validator** relationship (told through a school metaphor), the **BittBridge** subnet and its energy-forecasting task, and how the **incentive mechanism** turns forecast accuracy into on-chain rewards.
- **Chapter 2 — Setup: Wallet & Base Miner.** From an empty Google Cloud account to a *running miner*: the repo, the virtual machine, the firewall, the Python environment, your Bittensor wallet and testnet tokens, and `tmux` to keep it all alive.
- **Chapter 3 — Deploying Models: Linux, Config & the Leaderboard.** The Linux you actually need, pulling code updates, the configuration file, training **linear regression** and **CART** miners, redeploying on the fly, downloading your artifacts, and reading the live leaderboard.
- **Chapter 4 — Advanced: Train in Colab, Deploy on Bittensor.** The custom-model plugin architecture — build *any* regressor you like in Colab, save its weights, and ship it to your miner in production.

## How to read it

Read with a terminal open and the [`bittbridge/bittbridge`](https://github.com/bittbridge/bittbridge) guide in the next tab. When you hit a command, run it. When you hit a {bdg-primary}`Key idea` or a math callout, slow down — it's the minimum you need to reason about how the system scores you. And when something breaks (it will), remember the project's own advice: the README checklist exists precisely so you can find *where* it broke.

Let's get to work — and may your miner climb the leaderboard.

— *Dave & the BittBridge team*
