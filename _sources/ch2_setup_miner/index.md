# Chapter 2 — Setup: Wallet & Base Miner

:::{admonition} 🎥 The lectures & 📖 the guide behind this chapter
:class: seealso dropdown
Videos: *Miner #1 Intro*, *#2 Before You Start*, *#3.1–3.2 GCP VM Setup*, *#4.1–4.2 Wallets & Tokens*, *#5 Miner Deployment*.

Follow along in the official guide — this chapter is the narrated version of these pages:
- [`01-before-you-start.md`](https://github.com/bittbridge/bittbridge/blob/main/docs/guide/01-before-you-start.md)
- [`02-gcp-vm-setup.md`](https://github.com/bittbridge/bittbridge/blob/main/docs/guide/02-gcp-vm-setup.md)
- [`03-wallets-and-tokens.md`](https://github.com/bittbridge/bittbridge/blob/main/docs/guide/03-wallets-and-tokens.md)
- [`04-run-miner.md`](https://github.com/bittbridge/bittbridge/blob/main/docs/guide/04-run-miner.md)
:::

Everything starts at one place: the public GitHub repository **[`bittbridge/bittbridge`](https://github.com/bittbridge/bittbridge)** — the code *and* the documentation. Open it now. The README has a high-level workflow and a **checklist**; use the checklist religiously, so that when something breaks you know exactly which step to inspect. You'll work almost entirely on the **miner** side, because the validators are already deployed and maintained by the team.

```{admonition} Before you start — two prerequisites
:class: important
1. A **GitHub** account and a basic idea of what a repository is.
2. A **Google Cloud Platform** account with the **$300 free-trial** activated.

Two vocabulary words you'll need (`docs/guide/01-before-you-start.md`):
- **Virtual machine (VM):** a *remote computer* hosted on Google Cloud that runs 24/7 whether or not your laptop is on. Think "a PC in the cloud."
- **Virtual environment (venv):** an isolated folder inside your project that holds *just this project's* Python libraries, separate from your laptop's global Python.
```

## 2.1 The 30-second picture

A validator asks your miner, *"what will demand be in the next few minutes?"* Your miner runs a model, returns a number, and the validator later grades it against the real value and writes the reward to the Bittensor blockchain. Your job in this chapter is to get a **base miner** — a simple moving-average model — running in the cloud and registered on the subnet. Four moves: **VM → environment → wallet → run.**

## 2.2 Stand up the cloud VM

In the Google Cloud console (`docs/guide/02-gcp-vm-setup.md`):

1. **Firewall rule first.** Decentralization means miners and validators are separate machines around the world that must talk to each other, so you open a port. Create a rule — name it `allow-tcp-8091`, direction **ingress**, action **allow**, targets **all instances**, protocol **TCP**, port **8091**.
2. **Create the VM instance.** Change the boot disk to **Ubuntu 22.04 LTS (x86/64)**, bump the disk to **25 GB**, leave the rest default, and create it. Linux is the default here because it's light, fast, and what essentially every project runs on.
3. **Connect.** Click **SSH** and a terminal opens — you now have a real computer on Google's servers.

```{admonition} 💸 Don't forget to switch it off
:class: warning
The VM costs roughly **$25–26/month**. Your $300 trial will evaporate faster than you expect if you leave it running idle. Stop the instance when you're not using it.
```

## 2.3 Build the Python environment

In the SSH terminal:

```bash
# update the OS packages (like a macOS/Windows update)
sudo apt update && sudo apt upgrade -y

mkdir bittbridge_project && cd bittbridge_project
git clone <repo-url>        # clone bittbridge/bittbridge
cd bittbridge
ls                          # see the project structure: bittbridge/, docs/, neurons/, scripts/, ...
```

Now create and activate the **virtual environment** — when your prompt shows `(venv)`, it's active — then install the dependencies:

```bash
python -m venv venv
source venv/bin/activate    # prompt now shows (venv)
pip install -r requirements.txt
```

Verify Bittensor installed (you're looking for a `bittensor 4.x`). Two tiny but vital Linux habits introduced here: `ls` lists a folder's contents, and `cd <folder>` moves into it (`cd ..` moves up one level). You'll lean on these constantly in Chapter 3.

## 2.4 Create a wallet and get tokens

Now you touch the blockchain (`docs/guide/03-wallets-and-tokens.md`). Create a wallet for your miner with the `btcli` tool; name the wallet `miner`, accept the default hotkey, and set a password you will remember.

```{admonition} 🔑 Save your mnemonic — there is no "reset password"
:class: danger
Creating a wallet prints a **mnemonic phrase** (the seed words) and your **coldkey/hotkey addresses**. Store them somewhere safe *immediately*. This phrase is the **only** way to restore the wallet — lose it and the wallet is gone for good.
```

List your wallet and copy the **coldkey address**:

```bash
btcli wallet list
```

You need **testnet TAO** to participate. The faucet here is the instructor: send your coldkey address (following the in-class instructions) and tokens get transferred to you. Confirm with:

```bash
btcli wallet balance --wallet.name miner
```

Then **register** the wallet on the subnet — this announces to subnet 183 that a new miner wants to participate:

```bash
btcli subnet register     # confirm 'y', enter your password
```

When it finishes, the logs show your wallet registered on **netuid 183** with a unique **UID** (e.g., UID 12). **Save that UID** next to your mnemonic — it's how the leaderboard and validators identify you.

## 2.5 Wire up the data credentials

Your miner pulls real energy data from **ISO New England**, so create an account there and store the credentials in the project (`docs/guide/04-run-miner.md`). The repo ships a template, `.env.example`:

```bash
nano .env.example     # paste your ISO-NE email + password
# Ctrl+O, Enter to save; Ctrl+X to exit
mv .env.example .env  # rename so the app actually reads it
```

## 2.6 Run the miner (and keep it alive with tmux)

The final move is to run the miner — but a script started normally dies the moment you close the SSH window. The fix is **`tmux`**, a tool that runs your script in a background session that survives disconnects.

```{admonition} Mental model — tmux is a container for your script
:class: tip
Start your miner inside a `tmux` session and it keeps running on the VM even after you close your laptop — as long as the VM is on. You **detach** (leave it running) with **Ctrl+B**, then **D**, and **reattach** later with `tmux attach`. List sessions with `tmux ls`.
```

Inside a tmux session, activate the venv and launch the miner. The base model is a simple **moving average**; you can tune how far back it looks (e.g., editing the lookback in `neurons/miner.py` from 12 to 24 five-minute steps = the last two hours). Run it from the **`bittbridge` directory**:

```bash
tmux new -s miner
source venv/bin/activate
python neurons/miner.py        # (use the exact command from the guide)
```

You'll see an **Axon** (the miner's server) come up on an IP and port, and `miner running` — deployed onto **subnet 183**. Detach with **Ctrl+B, D** and walk away. Within five to ten minutes you'll see **success logs**: a validator asked for a prediction, and your miner, using the moving average of the most recent energy data, answered with a demand forecast for the next five minutes.

```{admonition} ✅ You did a lot
:class: note
In this one assignment you used the **Google Cloud Platform**, the **Linux terminal**, a **Bittensor wallet** and the **blockchain**, a **predictive model**, and **tmux** to keep it alive in production. You can now honestly say you deployed a prediction model onto a decentralized-AI network. Next chapter: make that model *good*.
```
