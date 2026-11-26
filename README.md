# emosentry

A Jupyter notebook that classifies tweets as hateful or normal. It compares four bag-of-words classifiers with a
bidirectional LSTM on GloVe embeddings, and turns emoji into words so that the LSTM can use them.

## Results

| Model | Accuracy | Hateful precision | Hateful recall | Hateful F1 |
|---|---|---|---|---|
| Multinomial naive Bayes | 0.848 | 0.68 | 0.81 | 0.74 |
| SVC | 0.900 | 0.90 | 0.71 | 0.79 |
| Decision tree | 0.893 | 0.81 | 0.78 | 0.80 |
| Logistic regression | 0.906 | 0.88 | 0.76 | 0.81 |
| BiLSTM with GloVe | 0.92 | 0.90 | 0.80 | 0.85 |

The test set has 12,679 of the 42,261 cleaned tweets, and about 73% of them are normal. These figures come from outputs
saved in the notebook after a Colab run in December 2023. They have not been reproduced, because the dataset is not in
the repository.

The LSTM's lead over logistic regression is small and comes from a single run. It also trains on lemmatised text with
emoji converted to words, while the other models do not, so the table does not isolate the effect of the architecture.

## How it works

1. Remove mentions, hashtags, links and stop words.
2. Train naive Bayes, SVC, decision tree and logistic regression on word counts.
3. Convert emoji to their names and lemmatise the text.
4. Train a bidirectional LSTM (128 units, 5 epochs) on frozen 100-dimensional GloVe embeddings.
5. Optionally, score messages from a YouTube live chat.

## Running it

You need Python 3.10 and the dataset saved as `MPD.xlsx` next to the notebook, with the text in a `tweets` column and
the label in a `labels` column (`0` hateful, `1` normal). The dataset is not included and its source is not recorded.

```bash
python3.10 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook emoSentry.ipynb
```

Run the cells in order. The notebook downloads the GloVe vectors, which is a large download, and the two "try a
sentence" cells wait for keyboard input.

The last section needs a YouTube Data API key in `YOUTUBE_API_KEY` (set it before starting Jupyter) and the ID of a
stream that is currently live in `VIDEO_ID`.

## Limitations

- Some tweets store emoji as HTML entities such as `&#128591;`. The hashtag filter removes them before conversion.
- The best classical model is chosen on the test set.
- The typed-sentence and live chat cells preprocess text differently from training.
- Correcting a prediction applies one training step to that sentence, without checking the updated model.

## Content warning

The dataset contains slurs and abusive language. Saved outputs that showed raw tweets or chat account names have been
cleared, although earlier versions remain in the Git history.
