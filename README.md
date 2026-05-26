# BotHire

I built this during my job search after coming across too many sketchy postings — vague descriptions, no company info, suspiciously high pay for "no experience required" roles. Figured there had to be a pattern to them.

Turns out there is. BotHire flags fraudulent job postings using NLP and a handful of structural signals that scam posts almost always get wrong.

**[Try the live demo](https://niharikaaryasomayajula.github.io/Bot-Hire)**

---

## Results

Tested on the [Kaggle fake job postings dataset](https://www.kaggle.com/datasets/shivamb/real-or-fake-fake-jobposting-prediction) — ~18k real postings, about 5% fraudulent.

- Accuracy: **80%**
- F1-score (fraud class): **0.75**
- ROC-AUC: ~0.91

The fraud class F1 is what matters here. Getting 80% accuracy on a 95/5 dataset is easy — just predict "legitimate" every time. The 0.75 F1 on the minority class is what tells you it's actually catching scams.

![Confusion Matrix](outputs/plots/confusion_matrix.png)
![Top Fraud Tokens](outputs/plots/top_fraud_tokens.png)

---

## How it works

Two feature types combined:

**Text (TF-IDF, unigrams + bigrams)**
The vocabulary of scam posts is distinct — urgency words, earning promises, vague role descriptions. TF-IDF picks this up and keeps the model interpretable. Signal matching uses word-boundary logic so "earn" in "E-Learning" doesn't fire, but "earn $5000 weekly" does.

**Structural signals**
- No company logo
- No salary range
- Description under 100 characters
- No screening questions
- No company profile

Combined into a logistic regression with SMOTE oversampling to handle the 5% class imbalance. Chose LR over a neural network deliberately — the coefficients tell you exactly which words and features drove the score, which matters when you're explaining a flag to a job seeker.

---

## Run it yourself

```bash
git clone https://github.com/NiharikaAryasomayajula/Bot-Hire.git
cd Bot-Hire
pip install -r requirements.txt
```

Download the dataset from Kaggle and save as `data/fake_job_postings.csv`, then:

```bash
# Train
python src/train.py

# Check a single posting interactively
python src/predict.py --interactive

# Batch predictions on a CSV
python src/predict.py --input data/new_jobs.csv --output outputs/predictions.csv
```

---

## What's in this repo

```
src/
  preprocess.py   text cleaning and feature engineering
  model.py        BotHireModel class
  train.py        training, evaluation, and plots
  predict.py      inference — single posting or batch CSV
docs/
  index.html      live PWA (deployed via GitHub Pages)
  manifest.json
  sw.js
requirements.txt
```

---

## Limitations worth knowing

**What the model catches well:**
Postings with explicit scam vocabulary — earn, guaranteed, no experience required, bank details, work from home guaranteed. These appear consistently in the training data and the model is calibrated to them.

**What it misses:**
Structural scams — a well-written fake listing from a misrepresented company (wrong country, non-existent employer, credential-harvesting operation) will score lower than it deserves if the vocabulary is clean. The model can't verify whether a company actually exists or whether a Sydney posting is genuinely based in Sydney.

**Class imbalance caveat:**
~5% base rate in the training data means a decent model still produces false positives in practice. A score of 39/100 "Suspicious" is not the same as confirmed fraud.

**The live demo is rule-based:**
The deployed PWA uses a heuristic approximation of the trained model — sklearn can't run in the browser. The Python model in `src/` is the real thing. Results in the demo are indicative, not definitive.

**Dataset age:**
Training data is from 2014-2018. Scam language evolves and the model will miss newer tactics that weren't present in that period.

---

## Stack

Python, scikit-learn, Pandas, NumPy, imbalanced-learn, Matplotlib

---

*Built by [Niharika Aryasomayajula](https://www.linkedin.com/in/aryaniharika) — Sydney, NSW*
