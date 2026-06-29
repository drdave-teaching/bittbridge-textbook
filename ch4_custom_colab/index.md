# Chapter 4 — Advanced: Train in Colab, Deploy on Bittensor

:::{admonition} 🎥 The lecture & 📖 the guide behind this chapter
:class: seealso dropdown
Video: *Advanced Models with Colab — Custom-model live demo*.

Guide page: [`07-custom-models.md`](https://github.com/bittbridge/bittbridge/blob/main/docs/guide/07-custom-models.md)

Custom-model notebook &nbsp; [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/bittbridge/bittbridge/blob/main/miner_model_energy/ExampleMinerModels_UPDATED_with_load_lags.ipynb) &nbsp; [GitHub](https://github.com/bittbridge/bittbridge/blob/main/miner_model_energy/ExampleMinerModels_UPDATED_with_load_lags.ipynb)
:::

In Chapter 3 you competed by toggling features and tuning a built-in CART. Now we remove the ceiling. The **custom-model plugin architecture** lets you train **any regressor you can write** — gradient boosting, an LSTM, a stacked ensemble, whatever — in **Google Colab**, then ship its trained weights to your miner in production. This is the part where your data-science skills, not the config file, decide your rank.

```{admonition} The big idea — separate training from serving
:class: important
Your miner doesn't train in production; it **serves** a saved model. So the workflow is: export a **training bundle** from the VM → **train anywhere** (Colab, with a GPU/TPU if you like) → save the **weights** → copy them back to the VM → tell the miner to serve them. The plugin architecture is just a clean contract between "where you train" and "where you serve."
```

## 4.1 Export a training bundle from the VM

Start as before — update and restart (`git reset --hard && git pull`, reinstall deps), pick your features in the config (say temperature, humidity, and a 2-hour lag), and run the miner. This time choose the option to **create a new custom-model folder**, naming it e.g. `custom_test`. Behind the scenes the miner creates `artifacts/custom_test/` containing **your training dataset**, **metadata** (JSON), and a **template Colab notebook**.

Zip that folder and download it to your laptop:

```bash
cd artifacts
zip -r custom_test.zip custom_test
realpath custom_test.zip      # copy this path, then use "download file"
```

(If the download won't start, refresh the SSH window and retry — same quirk as Chapter 3.)

## 4.2 Train your model in Colab

Open [Colab](https://colab.research.google.com) and upload the **template notebook** from the bundle (think of it as exactly that — a template). Set the runtime to **TPU/GPU** if you want, restart the session, and upload your **training dataset** into a `custom/` folder in the Colab file pane. Run the first cells to load the data — you'll see your chosen features (temperature, humidity, time, lag) all present.

```{admonition} Two hard rules
:class: warning
1. **Don't edit features inside Colab.** All feature engineering — adding, dropping, renaming — happens back on the VM via the config file. Colab is for *modeling only*, on the features you already chose.
2. **Your model must be a regressor.** The task is point forecasting of a continuous load value. The template's gradient-boosting and neural-net cells are just examples — replace them with anything (LSTM, RNN, XGBoost, a ridge regression), as long as it's a regressor.
```

Now write *your* model. The template ships example architectures, but you're meant to swap in your own. Critically, **add evaluation metrics and a couple of visuals** in the notebook — this is how you'll know whether the model is production-worthy *before* you deploy it. (In the live demo, a quick ridge regression scored an $R^2 \approx 0.3$ — better than a random guess, but a more carefully built custom model reached $R^2 \approx 0.8$–$0.9$. The gap between those two is your job.)

## 4.3 Save the weights the way the runtime expects

The serving side needs the trained **weights** in a known format:

```{list-table}
:header-rows: 1

* - Model family
  - Save with
* - scikit-learn (linear, ridge, trees, boosting…)
  - `joblib.dump(model, "model_custom.joblib")`
* - neural nets
  - TensorFlow/Keras `model.save(...)`
```

Saving produces a weights file (e.g. `model_custom.joblib`). Download it from Colab to your laptop's Downloads folder.

## 4.4 Ship it to the miner

Back on the VM, upload the weights file (Cloud Shell **Upload file**). It lands in your **home directory**, so move it into the model's artifacts folder:

```bash
cd ~                       # the upload lands here
mv model_custom.joblib bittbridge/artifacts/custom_test/
# verify:
cd bittbridge/artifacts/custom_test && ls   # model_custom.joblib should be here
```

```{admonition} tmux is a container that picks up your changes
:class: tip
You don't have to stop the miner to swap models. Make all your changes externally, then **Ctrl+C** the running miner inside its tmux session — when it restarts, the tmux "container" picks up the new files. Detach/scroll exactly as in Chapter 3.
```

## 4.5 Run your custom model in production

From the `bittbridge` directory, run the miner and choose the custom-model option. Decline to update data or create a new folder, and enter your folder name — `custom_test`. The runtime **identifies your model** in that folder and asks whether to deploy. (If the folder held several models, you'd choose which to serve here.) Confirm, and watch for the **success log**: your custom model is deployed.

A few minutes later you'll see it making **live predictions**. The logs even show the **model input row** — a JSON record of the exact weather-forecast features your model used for that prediction; copy it into a JSON viewer to inspect what it saw. Remember from Chapter 1: the leaderboard for this prediction only resolves **six hours later**, when the validator receives the real demand and grades you.

```{admonition} ✅ You closed the full loop
:class: note
You exported a training bundle, trained a model of your own design in Colab, saved its weights, shipped them to a cloud VM, and served them to a live decentralized-AI subnet that scores you in public against the world. That is the entire MLOps lifecycle — model, package, deploy, monitor, iterate — on a real system.
```

## 4.6 Where to go next

You now have everything you need to **compete**: pick smarter features, engineer better lags, try ensembles, tune relentlessly against the **24-hour leaderboard**, and redeploy in minutes. Send the team screenshots when you hit bugs, and share feedback. Be creative, experiment, and have fun — your miner is out there right now, predicting the lights of New England.

*Let the games begin.* 🏆
