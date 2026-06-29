# BittBridge — *Decentralized AI, Hands-On* (textbook)

A casual-but-rigorous, **do-it-yourself textbook** for the **OPIM 5509 BittBridge final project**: deploy an energy-demand–forecasting miner on **Bittensor Subnet 183**. Built from the BittBridge team's lecture transcripts woven with the open-source [`bittbridge/bittbridge`](https://github.com/bittbridge/bittbridge) guide and codebase, plus added concepts and math.

This is a [Jupyter Book](https://jupyterbook.org). It is the book edition of the 18 BittBridge videos (see the companion transcript repo, [`bittbridgesubnet183-transcripts`](https://github.com/drdave-teaching/bittbridgesubnet183-transcripts)).

## View it (local)
Open in your browser:
```
C:\Users\dww05002\kaltura\bittbridge-textbook\_build\html\index.html
```

## Rebuild after edits
```bash
jupyter-book build .
```
(or `python -c "import sys; from jupyter_book.cli.main import main; sys.argv=['jupyter-book','build','.']; main()"`)

## Chapters
1. **What Is Decentralized AI?** — Bittensor, subnets, miners & validators, the energy task, the incentive mechanism
2. **Setup: Wallet & Base Miner** — GCP VM → Python env → wallet & tokens → run the miner with `tmux`
3. **Deploying Models** — Linux basics, config file, linear regression & CART, redeploy, the leaderboard
4. **Advanced** — custom-model plugin architecture: train in Colab, deploy on Bittensor

## Source
- Narrative: the 18 polished transcripts (Dmitrii Tuzov & the BittBridge team).
- Code & commands: the [`bittbridge/bittbridge`](https://github.com/bittbridge/bittbridge) guide (`docs/guide/`) and the `miner_model_energy/` example notebook.
