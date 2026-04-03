# Dota 2 Prematch Outcome Prediction

This project explores how much can be predicted in Dota 2 **before a match really begins**. Instead of relying on post-game statistics such as KDA, damage, or economy, the project uses **prematch and draft-context information** to predict two outcomes:

1. **Win/Loss** — whether the match ends in a win or a loss  
2. **Match Duration** — whether the match is **short**, **medium**, or **long**

The goal is to move from “What happened by the end of the game?” to “What can already be inferred from the starting context?”

---

## Motivation

In an earlier version of this project, I used post-match performance statistics to predict whether I would win or lose a game. While those features were highly predictive, they also leaked information from later in the match.

This version asks a harder and more realistic question: **how much can be predicted from the draft and prematch setup alone?**

Because Dota 2 is heavily influenced by hero matchups, team composition, role balance, queue context, and patch environment, prematch features may already contain useful signal about both match outcome and match length.

---

## Data Source

The dataset was built using the **OpenDota API**, which provides structured access to public Dota 2 match data.

To construct the dataset, I:
- Queried my **500 most recent matches**
- Downloaded full match-level data for each game
- Cached raw match JSON locally to avoid repeated API calls
- Saved processed tables to CSV for faster reruns

---

## Prediction Tasks

This project studies two supervised classification tasks using the same prematch dataset:

### 1. Win/Loss Prediction
A **binary classification** task where:
- `1` = win
- `0` = loss

### 2. Match Duration Prediction
A **multiclass classification** task where matches are grouped into:
- **Short**: 36.21 minutes or less
- **Medium**: more than 36.21 and up to 44.62 minutes
- **Long**: more than 44.62 minutes

These duration classes were created using quantile-based splits so that the classes remain approximately balanced.

---

## Features

The feature matrix is built entirely from **prematch and draft-context variables**.

### Included features
- My selected hero
- Ally hero lineup
- Enemy hero lineup
- Team role counts
- Side (`Radiant` / `Dire`)
- Party size
- Average rank / skill context
- Game mode
- Lobby type
- Region
- Patch / version

### Excluded features
To keep the task realistic, I intentionally excluded post-match variables such as:
- Kills
- Deaths
- Assists
- GPM / XPM
- Hero damage
- Purchased items

These features are only known after the game has progressed, so using them would introduce data leakage.

---

## Data Processing

The project required several preprocessing steps before modeling:

- Removed rows missing essential fields such as outcome, duration, hero identity, and ally/enemy lineups
- Encoded win/loss as a numeric binary label
- Converted continuous match duration into three balanced classes
- Parsed JSON-like hero lineup fields
- One-hot encoded my hero selection
- Multi-hot encoded ally and enemy hero lineups
- Preserved numeric context variables such as rank and party size
- Included summarized role-count features for both teams

The final processed feature matrix has:

- **500 matches**
- **361 prematch/context features**

The train/test split uses:
- **400 training matches**
- **100 test matches**

A shared stratified split was used so that both win/loss and duration distributions stay balanced across train and test sets.

---

## Models

This project compares both **different classifiers** and **different feature representations**.

### Baseline classifiers
- **Logistic Regression**
- **Kernel SVM (RBF SVC)**

### Representation learning methods
- **PCA** for linear dimensionality reduction
- **Autoencoders** for nonlinear compressed representations

The main comparison is not only between linear and nonlinear classifiers, but also between:
- raw prematch features
- PCA-transformed features
- autoencoder latent representations

---

## Training Pipeline

The workflow used in this project is:

1. Collect match data from OpenDota
2. Build a prematch feature matrix
3. Define the two targets:
   - win/loss
   - short/medium/long duration
4. Split data into train and test sets using one shared stratified split
5. Train Logistic Regression and RBF-SVM on raw features
6. Apply PCA and retrain models on PCA features
7. Train autoencoders on the training data
8. Extract latent vectors from the encoder
9. Retrain Logistic Regression and RBF-SVM on latent representations
10. Evaluate all models on the held-out test set
11. Compare performance and visualize learned structure

---
