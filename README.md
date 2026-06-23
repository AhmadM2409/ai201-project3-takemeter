# TakeMeter: NBA Reddit Discourse Classifier

## Project Overview

TakeMeter is a text classifier that evaluates the type of discourse found in r/nba posts and comments. The goal is to classify NBA discussion into three categories: `analysis`, `hot_take`, and `reaction`.

This project uses a fine-tuned `distilbert-base-uncased` model and compares it against a zero-shot Groq baseline using `llama-3.3-70b-versatile`.

## Community Choice

I chose r/nba because it is an active basketball discussion community with a wide range of discourse quality. Posts and comments often include serious statistical analysis, emotional reactions to games, and unsupported hot takes, which makes it a strong fit for a text classification task.

## Label Taxonomy

### analysis

A post is `analysis` if it makes a structured basketball argument using specific evidence, stats, film observations, historical comparison, or tactical reasoning.

Examples:

* "SGA gets to the line so much because OKC spaces the floor and forces defenders to guard him one-on-one on drives."
* "The Celtics are tough to beat because every starter can shoot, switch, or attack a closeout. That makes their offense harder to scheme against."

### hot_take

A post is `hot_take` if it makes a bold basketball claim with little or no supporting evidence.

Examples:

* "The Suns are cooked. That roster is never winning a championship."
* "Anthony Edwards is better than every guard in the league right now."

### reaction

A post is `reaction` if it is mainly an immediate emotional response to a game, play, player, or news event without a real argument.

Examples:

* "I cannot believe they blew another fourth quarter lead."
* "That dunk was insane. I jumped out of my chair."

## Data Collection

I collected public r/nba posts and comments related to NBA trades, player rankings, historical discussion, team performance, and game reactions.

Current dataset file:

```text
nba_takemeter_dataset.csv
```

Current label distribution:

| Label     |  Count |
| --------- | -----: |
| analysis  |     20 |
| hot_take  |      8 |
| reaction  |      5 |
| **Total** | **33** |

Note: this current version is a starter dataset used to test the workflow. The final assignment requires at least 200 labeled examples, so this dataset needs to be expanded before final submission.

## Difficult Labeling Examples

### Example 1

Text: "LeBron is overrated because his Finals record is 4-6."

Possible labels: `analysis`, `hot_take`

Decision: I would label this as `hot_take` because it uses one statistic to support a broad claim without explaining context.

### Example 2

Text: "This team has no late-game offense, they just spam isolations and pray."

Possible labels: `analysis`, `reaction`

Decision: I would label this as `reaction` if it is mostly frustration, but `analysis` if the post explains why the isolation offense is failing.

### Example 3

Text: "Mo will be a player that every ball knower will be like 'Damn, the Knicks really got this guy for how much?' give it some time and you'll see."

Possible labels: `hot_take`, `analysis`

Decision: I labeled this as `hot_take` because it predicts player value confidently without giving detailed evidence.

## Fine-Tuning Approach

The fine-tuned model started from:

```text
distilbert-base-uncased
```

The notebook split the dataset into train, validation, and test sets using a 70/15/15 split.

Training setup:

| Setting       | Value                   |
| ------------- | ----------------------- |
| Base model    | distilbert-base-uncased |
| Epochs        | 3                       |
| Learning rate | 2e-5                    |
| Batch size    | 16                      |

I kept the default hyperparameters because the first goal was to verify that the full pipeline worked before experimenting with training settings.

## Baseline Approach

For the baseline, I prompted Groq's `llama-3.3-70b-versatile` to classify each test example without fine-tuning. The prompt defined all three labels and instructed the model to output only one label name: `analysis`, `hot_take`, or `reaction`.

## Evaluation Results

The test set had 5 examples.

| Model                   | Accuracy |
| ----------------------- | -------: |
| Groq zero-shot baseline |     1.00 |
| Fine-tuned DistilBERT   |     0.60 |

The zero-shot Groq baseline outperformed the fine-tuned DistilBERT model on this small test set. This result is not reliable yet because the test set only had 5 examples.

## Fine-Tuned Confusion Matrix

Rows are true labels. Columns are predicted labels.

| True Label | Predicted analysis | Predicted hot_take | Predicted reaction |
| ---------- | -----------------: | -----------------: | -----------------: |
| analysis   |                  2 |                  0 |                  1 |
| hot_take   |                  1 |                  0 |                  0 |
| reaction   |                  0 |                  0 |                  1 |

## Fine-Tuned Model Per-Class Metrics

| Label    | Precision | Recall |   F1 | Support |
| -------- | --------: | -----: | ---: | ------: |
| analysis |      0.67 |   0.67 | 0.67 |       3 |
| hot_take |      0.00 |   0.00 | 0.00 |       1 |
| reaction |      0.50 |   1.00 | 0.67 |       1 |

## Wrong Prediction Analysis

### Wrong Prediction 1

Text: "another fun fact: in 1956, Bill Russell qualified for the Olympic High jump. He was ranked #2 in the US and #7 in the world..."

True label: `analysis`
Predicted label: `reaction`
Confidence: 0.36

Analysis: The model likely misclassified this because the example reads like an interesting fact or reaction-style comment rather than a structured argument. Even though it includes historical evidence, it does not use that evidence in a very explicit argument structure.

### Wrong Prediction 2

Text: "Mo will be a player that every ball knower will be like 'Damn, the Knicks really got this guy for how much?' give it some time and you'll see."

True label: `hot_take`
Predicted label: `analysis`
Confidence: 0.36

Analysis: The model likely treated the player evaluation as analysis because it discusses future player value. However, the comment does not provide specific evidence, stats, or reasoning, so it fits better as a hot take.

## Sample Classifications

| Text                                                                             | Predicted Label | Confidence |
| -------------------------------------------------------------------------------- | --------------- | ---------: |
| "another fun fact: in 1956, Bill Russell qualified for the Olympic High jump..." | reaction        |       0.36 |
| "Mo will be a player that every ball knower will be like..."                     | analysis        |       0.36 |
| "I cannot believe they blew another fourth quarter lead."                        | reaction        |        TBD |
| "SGA gets to the line so much because OKC spaces the floor..."                   | analysis        |        TBD |
| "The Suns are cooked. That roster is never winning a championship."              | hot_take        |        TBD |

One reasonable correct prediction is a comment like "I cannot believe they blew another fourth quarter lead" being classified as `reaction`, because it is mainly an emotional response without a developed basketball argument.

## Reflection: What the Model Learned vs. What I Intended

I intended the model to learn the difference between structured basketball reasoning, unsupported strong opinions, and emotional reactions. The current fine-tuned model appears to partially learn this distinction, but the dataset is too small for reliable results.

The model struggled most with the boundary between `analysis` and `hot_take`. It sometimes treated any player evaluation as analysis, even when the comment did not include evidence. It also misread factual historical information as reaction, probably because the example did not follow a clear argument format.

To improve the model, I would add more labeled examples, especially borderline cases where a comment includes some evidence but still functions as a hot take.

## Spec Reflection

One way the spec helped was by forcing me to define the labels before collecting and training on the data. This made it easier to decide how to label difficult examples.

One way the implementation diverged from the spec is that I initially tested the pipeline with a much smaller dataset than planned. I did this to make sure the Colab notebook, CSV format, baseline, fine-tuning, and evaluation exports worked before expanding the dataset to the required 200 examples.

## AI Usage

I used AI assistance to help write and refine the project planning document, including the community choice, label definitions, edge cases, data collection plan, and evaluation plan.

I also used AI assistance to convert raw collected NBA text into a starter CSV with `text`, `label`, and `notes` columns. I reviewed the labels and understand that the dataset needs to be expanded and checked before final submission.

I used AI assistance to interpret the model's wrong predictions and identify likely failure patterns, especially confusion between `analysis` and `hot_take`.

## Files

* `planning.md`
* `nba_takemeter_dataset.csv`
* `evaluation_results.json`
* `confusion_matrix.png`
