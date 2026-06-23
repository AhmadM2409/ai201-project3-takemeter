# TakeMeter Planning

## Community

I chose r/nba because it is an active basketball discussion community with a wide range of discourse quality. Posts and comments often include serious statistical analysis, emotional reactions to games, and unsupported hot takes, which makes it a strong fit for a text classification task.

## Labels

### analysis
A post is `analysis` if it makes a structured basketball argument using specific evidence, stats, film observations, historical comparison, or tactical reasoning.

Example 1: "The Celtics are hunting mismatches by forcing switches onto smaller guards, which is why Tatum keeps getting clean looks in the midrange."

Example 2: "SGA's free throw rate is not random. OKC spaces the floor with shooters, which gives him driving lanes and forces defenders to reach."

### hot_take
A post is `hot_take` if it makes a bold basketball claim with little or no supporting evidence.

Example 1: "Anthony Edwards is already better than every guard in the league."

Example 2: "The Suns are cooked. That team is never winning anything."

### reaction
A post is `reaction` if it is mainly an immediate emotional response to a game, play, player, or news event without a real argument.

Example 1: "WHAT WAS THAT SHOT???"

Example 2: "I cannot believe they blew that lead again."

## Hard Edge Cases

The hardest edge case will be posts that make a bold claim but include one small piece of evidence. For example: "LeBron is overrated because his Finals record is 4-6."

This could look like `analysis` because it includes a statistic, but I would label it `hot_take` because the post does not build a full argument or explain the context behind the stat.

Decision rule: if the evidence is specific and used to support a structured basketball argument, label it `analysis`. If the evidence is brief, cherry-picked, or mostly used to make a bold unsupported claim sound stronger, label it `hot_take`.

Another hard edge case will be emotional posts that also imply a basketball argument. For example: "This team has no late-game offense, they just spam isolations and pray."

I would label this `analysis` if the post explains the offensive issue with basketball reasoning. I would label it `reaction` if it is mostly frustration with no developed explanation.

## Data Collection Plan

I will collect at least 200 public comments from r/nba. I will focus on posts and comment threads where people are discussing games, players, trades, rankings, and team performance because those topics usually contain analysis, hot takes, and reactions.

My goal is to collect a balanced dataset with roughly:

* 70 examples labeled `analysis`
* 65 examples labeled `hot_take`
* 65 examples labeled `reaction`

If one label is underrepresented after collecting 200 examples, I will collect more examples from threads where that type of comment is more common. For example, post-game threads may contain more `reaction` comments, while serious discussion threads may contain more `analysis` comments.

I will save the dataset as one CSV file with these columns:

```csv
text,label,notes
```

The `notes` column will be used for difficult or borderline examples.

## Evaluation Metrics

I will evaluate the classifier using overall accuracy, per-class precision, recall, and F1 score.

Accuracy is useful because it shows the percentage of test examples the model labels correctly overall. However, accuracy alone is not enough because the dataset has three labels, and the model could perform well overall while still failing badly on one label.

Per-class F1 score is important because it shows whether the model can correctly identify each type of NBA discourse: `analysis`, `hot_take`, and `reaction`. I will also use a confusion matrix to see which labels the model mixes up most often, especially `analysis` vs. `hot_take`, since that is the hardest boundary in this project.

## Definition of Success

I would consider this classifier successful if the fine-tuned model performs better than the zero-shot Groq baseline on the same test set.

A useful target would be at least 70% overall accuracy and an F1 score of at least 0.65 for each label. This would show that the model is not only getting many examples correct overall, but also learning all three categories instead of only predicting the easiest or most common label.

For a real community tool, I would not expect this classifier to make final moderation decisions by itself. I would consider it good enough for deployment only as a support tool that gives a suggested label and confidence score for human review.

## AI Tool Plan

I will use AI tools in three specific ways during this project.

First, I will use an AI tool for label stress-testing. I will give the AI my label definitions and ask it to generate borderline NBA comments that could fit between `analysis`, `hot_take`, and `reaction`. If those examples are hard to classify, I will revise my label definitions or decision rules before labeling the full dataset.

Second, I may use an AI tool for annotation assistance by asking it to suggest labels for some collected comments. If I do this, I will manually review and correct every suggested label before adding it to the final dataset. I will also disclose this in my README.

Third, I will use an AI tool for failure analysis after training. I will give it examples that the fine-tuned model classified incorrectly and ask it to identify possible patterns, such as sarcasm, short comments, emotional language, or confusion between `analysis` and `hot_take`. I will verify any patterns myself before including them in the evaluation report.
