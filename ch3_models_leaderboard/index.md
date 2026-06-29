# Chapter 3 — Deploying Models: Linux, Config & the Leaderboard

:::{admonition} 🎥 The lectures & 📖 the guide behind this chapter
:class: seealso dropdown
Videos: *Linux Basics*, *Codebase Update*, *Config + Linear Regression + CART*, *How to Make Changes & Redeploy*, *Download actuals-vs-predicted*, *Leaderboard Use*.

Guide pages:
- [`05-update-and-restart.md`](https://github.com/bittbridge/bittbridge/blob/main/docs/guide/05-update-and-restart.md)
- [`06-advanced-miner-models.md`](https://github.com/bittbridge/bittbridge/blob/main/docs/guide/06-advanced-miner-models.md)

Example models notebook &nbsp; [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/bittbridge/bittbridge/blob/main/miner_model_energy/ExampleMinerModels_UPDATED_with_load_lags.ipynb) &nbsp; [GitHub](https://github.com/bittbridge/bittbridge/blob/main/miner_model_energy/ExampleMinerModels_UPDATED_with_load_lags.ipynb)
:::

Your base miner runs a moving average — fine for a first deploy, hopeless on the leaderboard. This chapter is where you become competitive: a little more Linux, a code update, and then the loop that actually wins — **edit the config, train a real model, redeploy, read the leaderboard, repeat.**

## 3.1 Just enough Linux to be dangerous

You navigate the VM with a handful of commands. Run them, don't memorize them — muscle memory comes fast:

```{list-table}
:header-rows: 1

* - Command
  - What it does
* - `pwd`
  - **print working directory** — where am I right now?
* - `ls`
  - **list** the files/folders in the current directory
* - `cd <folder>` / `cd ..`
  - **change directory** into a folder / up one level
* - `nano <file>`
  - open the basic text editor (save **Ctrl+O**, exit **Ctrl+X**)
* - `↑` / `Ctrl+R`
  - recall the previous command / **search** your command history
```

One more `tmux` trick you'll want: to **scroll** inside a running session (which normally won't scroll), press **Ctrl+B** then **`[`** to enter navigation mode (yellow numbers appear top-right), scroll with the mouse or arrows, and press **`q`** to exit. This is how you read your miner's logs without stopping it.

## 3.2 Pull the latest code

The team ships improvements to the `main` branch, so before serious work, update (`docs/guide/05-update-and-restart.md`). Stop the miner first, then pull:

```bash
tmux attach -t miner    # back into the session
# Ctrl+C  -> stops the miner
# Ctrl+B, D -> detach
git pull                # fetch the latest code
```

```{admonition} If git refuses to pull
:class: warning
If you see *"please commit your changes or stash them before you merge,"* your local edits are in the way. The guide's blunt fix is to throw them away and take the remote version:

    git reset --hard
    git pull

Then re-activate the venv (`source venv/bin/activate`) and `pip install -r requirements.txt` to pick up any new dependencies.
```

## 3.3 The configuration file is your control panel

Everything about *which* model you train and *which* features it sees lives in one file — the model-params config (`nano model_params.yaml`). It's heavily commented; read it. Two kinds of settings:

- **Feature toggles** — booleans (`true`/`false`) that turn features on for the training **and** test datasets: time features, temperature, relative humidity, and so on.
- **Model hyperparameters** — e.g., the **max depth** of decision-tree models, leaf size, or the parameters of LSTM/RNN models.

```{admonition} Key idea — change the config, not the engine
:class: tip
You compete by editing **features and hyperparameters** in this one file (and later, in Chapter 4, by shipping your own model). Leave the training/validation split and the internal plumbing at their defaults unless the guide tells you otherwise. Track every change like a scientist — "experiment 1: time features only" — so you can attribute improvements.
```

## 3.4 Train a real model: linear regression → CART

With the config set, run the miner and it will offer you choices. Decline the baseline moving-average model, update the training dataset when prompted (the first fetch takes a few minutes), and pick a model to train. Start with **linear regression**.

Scroll the logs (Ctrl+B, `[`) and you'll see the dataset **shape**, the training time, and **metrics on the training and validation sets**, plus an actual-vs-predicted plot for each. Linear regression on this problem is usually weak, so when it asks whether to deploy, say **no** and train the **CART** model (a regression decision tree) instead.

```{admonition} Reading the metrics
:class: note
The number to watch is **validation $R^2$** — the fraction of variance your model explains on *held-out* data. A value above 0 already beats a random guess; CART typically lands well above linear regression here, because energy demand is full of nonlinear, threshold-like behavior (heaters kick on below a temperature, etc.) that a straight line can't capture. Always trust the **validation** figure over the training figure — a model can ace the training set and still be useless live.
```

When CART's metrics look better, deploy it: quit navigation mode (`q`), answer **yes**, and the miner redeploys *on the fly*. Within ~5 minutes a success log confirms the CART model deployed; within ~10 it's making live predictions.

## 3.5 The iteration loop (without stopping the miner)

Here's the move that makes you fast: you can keep your miner running, **detach** (Ctrl+B, D), and experiment in a second shell. Edit the config — say, add **humidity** and bump CART's **depth** from 6 to 7 — and do a *dry* training run to see the metrics **without deploying**. Compare against your last experiment. A small, well-chosen change (temperature + humidity + a touch more depth) can move your validation $R^2$ noticeably. When a configuration wins, reattach to the tmux session and redeploy it.

```{admonition} The competitive loop
:class: important
**edit config → dry-run train → compare metrics → if better, redeploy → watch leaderboard → repeat.** This is real MLOps: you are the data scientist *and* the engineer, shipping models into a live system and reading production feedback. That feedback loop — not any single model — is the skill this project teaches.
```

## 3.6 Your artifacts: download what your model did

Every run writes an **artifacts** folder (`docs/guide/06-advanced-miner-models.md`). Inside each run you'll find five files worth knowing:

```{list-table}
:header-rows: 1

* - File
  - What it holds
* - `model_*.joblib`
  - the **dumped model weights** — download and run locally
* - `metrics.json`
  - the run's metrics
* - `actual_vs_predicted.csv`
  - predictions vs. ground truth (great for your own plots)
* - `config_snapshot.yaml`
  - the exact config you used
* - `manifest.json`
  - data shape + training-data timestamp
```

To pull one to your laptop, get its path with `realpath actual_vs_predicted.csv`, then use the Cloud Shell **download file** option with that path. (If a download stalls, refresh the SSH window and retry — a known quirk.)

## 3.7 Read the leaderboard like a pro

The public website has **three** leaderboards, and knowing which to trust is itself a skill:

1. **Latest prediction** — updates every cycle; shows who was closest to ground truth *right now*. Exciting, but noisy: a near-random model can nail one timestamp by luck.
2. **24-hour average (cumulative reward)** — the one that matters. It rewards models that are *consistently* close over a full day, which is what the incentive mechanism actually pays for.
3. **Time-series plot** — overlays each miner's predictions on the real demand curve. Filter to 24 hours and you can *see* model behavior: a moving-average miner lags and smooths over the spikes; a CART miner makes the characteristic stair-step of a decision tree; a good model tracks the real ups and downs, including the demand spike on a cold, rainy day when everyone turns on the heat.

```{admonition} Key idea — optimize for the 24-hour board
:class: tip
A single lucky prediction is meaningless; **cumulative accuracy over time** is what earns emissions. When you compare your miner to a classmate's (filter by UID), compare them on the 24-hour view. And give a freshly restarted miner up to ~30 minutes to appear on the board.
```

By the end of this chapter your CART miner is live, you can iterate on it in minutes, and you can read the leaderboard honestly. In Chapter 4 we blow the doors off the config file and let you ship **any model you can dream up**.
