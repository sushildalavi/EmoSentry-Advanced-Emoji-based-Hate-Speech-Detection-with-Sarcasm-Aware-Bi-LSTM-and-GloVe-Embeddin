# EmoSentry

A single Jupyter notebook, written in Google Colab, that classifies tweets as hateful or normal. It compares four
bag-of-words classifiers with a bidirectional LSTM on GloVe embeddings, converts emoji to their text names during
preprocessing, and ends with an experiment that reads comments from a YouTube live chat.

This is an exploratory project, not a service. The repository name mentions sarcasm awareness, but the notebook has no
sarcasm detection or sarcasm-specific features.

## What the notebook does

| Cells | Step |
|---|---|
| 0-3 | Load `MPD.xlsx`, strip mentions, hashtags, links and stop words, draw a word cloud |
| 4 | Convert emoji to text names (`emoji.demojize`) and lemmatise, producing the Bi-LSTM input |
| 5 | Vectorise with `CountVectorizer` and make a 70/30 train/test split (`random_state=42`) |
| 6-9 | Train multinomial naive Bayes, SVC, decision tree and logistic regression |
| 10-12 | Pick the most accurate of the four, plot the comparison, pickle it to `ML_model.pickle` |
| 13 | Classify a typed sentence with logistic regression |
| 14-16 | Download GloVe 6B and train the Bi-LSTM (frozen 100-dimensional embeddings, 128 LSTM units, 5 epochs) |
| 17 | Classify a typed sentence with the saved Bi-LSTM; if marked wrong, fit one step on it and save the model |
| 18 | Read a YouTube live chat and print messages the model scores as hateful |
| 19 | Classification report and confusion matrix for the Bi-LSTM |

Label `0` means hateful and `1` means normal.

## Recorded results

These numbers are copied from the outputs saved in the notebook. They were produced in Colab in December 2023 and have
not been reproduced since, because the dataset is not in the repository.

The cleaned dataset held 42,261 tweets (11,383 hateful, 30,878 normal), split into 29,582 for training and 12,679 for
testing.

| Model | Accuracy | Hateful precision | Hateful recall | Hateful F1 |
|---|---|---|---|---|
| Multinomial naive Bayes (alpha 0.2) | 0.848 | 0.68 | 0.81 | 0.74 |
| SVC | 0.900 | 0.90 | 0.71 | 0.79 |
| Decision tree | 0.893 | 0.81 | 0.78 | 0.80 |
| Logistic regression | 0.906 | 0.88 | 0.76 | 0.81 |
| Bi-LSTM with GloVe | 0.92 | 0.90 | 0.80 | 0.85 |

For scale, always predicting "normal" would score about 0.73 on this test set. The Bi-LSTM is ahead of logistic
regression by a small margin from a single run. The four classical models are trained on the cleaned tweets, while the
Bi-LSTM is trained on the demojised and lemmatised text, so the comparison does not isolate the model architecture.

## Setup

Requirements:

- Python 3.10.
- The dataset, saved as `MPD.xlsx` next to the notebook. It is not included, and its source is not recorded here. The
  notebook expects two columns: tweet text (`tweets`) and label (`labels`).
- Internet access for the NLTK resources and GloVe 6B (cell 14 downloads a zip of several hundred megabytes).

The notebook uses the Keras 2 `Tokenizer` and `Embedding` arguments, so it needs TensorFlow 2.15 or earlier. It was
written against pandas 1.5.

```bash
python3.10 -m venv .venv
source .venv/bin/activate
pip install "tensorflow==2.15.*" "pandas<2" "numpy<2" openpyxl scikit-learn matplotlib seaborn \
    nltk wordcloud emoji google-api-python-client google-auth jupyter
jupyter notebook emoSentry.ipynb
```

Run the cells in order. Cells 13 and 17 wait for keyboard input.

### YouTube live chat (cell 18)

Set a YouTube Data API v3 key in the environment before starting Jupyter. In Colab, set `os.environ["YOUTUBE_API_KEY"]`
from Colab's secret store rather than typing the key into a cell.

```bash
export YOUTUBE_API_KEY=your-key
```

Replace `VIDEO_ID` in the cell with the ID of a stream that is currently live. Run cell 17 first, since cell 18 relies
on its `pad_sequences` import.

## Known problems

- Cell 18 creates a new `Tokenizer` and never fits or loads one. Every message becomes an all-zero sequence, so its
  predictions carry no information (the saved output reports no message as hateful). It would need to load
  `tokenizer.pickle` and repeat the training preprocessing.
- Some tweets store emoji as HTML numeric entities such as `&#128591;`. The hashtag-removal pattern deletes them before
  demojisation, so those emoji never reach any model.
- Cell 17 preprocesses text differently from training (no stop-word removal or lemmatisation, and a different handling
  of emoji names).
- The "feedback" step is a single gradient update on one sentence, with no validation of the updated model.
- The classical models are compared and selected on the test set.
- There are no automated tests, and the trained model files are not included.

## Data and content warning

The dataset and the saved notebook outputs contain slurs and abusive language. The outputs also include account names
from public tweets and YouTube chat.
