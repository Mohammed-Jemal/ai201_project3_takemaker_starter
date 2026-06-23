# Takemeter r/soccer post classifier Evaluation Report:

** Theis project evaluates and compare two distinct paradigms for classifying text from the r/soccer reddit communities in to three parts as 'analysis', 'hot_take', and 'reaction'. We buit a custom fine_tuned transformer model using "DistilBert" and compared its performance directly against a zero-shot LLM baseline running **LLMA-3.3-70b-versatile** via the Groq API.

The custom model was trained on a meticulously curated dataset of 203 manually labeled community posts.

## Evaluation results:

### Overall performance summary:
Model                               Accuracy
Zero_shot baseline(Groq)            0.903
fine_tuned DistilBert               0.774


** Finetuning regression: -0.129


### Per-Class Metrics Matrix

| Model | Class | Precision | Recall | F1-Score | Support |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Zero-shot Baseline (Llama 3.3)** | analysis | 1.00 | 0.80 | 0.89 | 10 |
| | hot_take | 0.90 | 0.90 | 0.90 | 10 |
| | reaction | 0.85 | 1.00 | 0.92 | 11 |
| **Fine-tuned DistilBERT** | analysis | 0.67 | 1.00 | 0.80 | 10 |
| | hot_take | 1.00 | 0.30 | 0.46 | 10 |
| | reaction | 0.85 | 1.00 | 0.92 | 11 |

### Fine-Tuned Model Confusion Matrix (Text Representation)

| Actual \ Predicted | Predicted: analysis | Predicted: hot_take | Predicted: reaction |
| :--- | :---: | :---: | :---: |
| **Actual: analysis** | **10** | 0 | 0 |
| **Actual: hot_take** | 5 | **3** | 2 |
| **Actual: reaction** | 0 | 0 | **11** |

---

## Failure Case Analysis

A pattern-recognition sweep via Claude revealed that our fine-tuned DistilBERT model suffered a significant blind spot: it struggled heavily with the boundary between `hot_take` and `analysis`, frequently misclassifying highly opinionated arguments as objective analysis if the sentence contained formal tactical jargon.

Here are 3 specific failure examples from the test set:

### 1. "Tactical fouls in the midfield should be an automatic red card; it ruins the flow of open, attacking football."
* **True Label**: `hot_take`
* **Predicted Label**: `analysis` (Confidence: 0.61)
* **Why it Failed**: The sentence utilizes tactical buzzwords ("tactical fouls", "midfield", "attacking football"). Because our training data for `analysis` is dense with these exact tokens, the model's self-attention layers over-indexed on the vocabulary tokens while completely missing the normative statement of opinion ("should be an automatic red card").

### 2. "Losing a match in extra time is entirely down to poor physical conditioning and lack of preparation by the coaching staff."
* **True Label**: `hot_take`
* **Predicted Label**: `analysis` (Confidence: 0.51)
* **Why it Failed**: This post structurally masquerades as a statement of analytical causality ("is entirely down to..."). DistilBERT lacked the deep semantic reasoning required to distinguish a sweeping, unproven overgeneralization (a hot take) from a real statistical evaluation.

### 3. "Belgium was really handed the easiest group with crappiest teams and the easiest route to the QF but decided to eat crayons instead"
* **True Label**: `hot_take`
* **Predicted Label**: `analysis` (Confidence: 0.43)
* **Why it Failed**: While highly informal, this post presents a structural argument regarding a team's tournament pathway ("easiest group... easiest route"). The model failed to realize that the vivid colloquial insult ("eat crayons instead") pulled this firmly into subjective territory, defaulting instead to `analysis` due to the tournament structural terminology.

---

## Sample Classifications

Below is a subset of test records passing through the custom pipeline:

* **Text**: *"Oh my god, I cannot believe they just missed that shot!"*
    * **Predicted**: `reaction` (Confidence: 0.96)
    * *Justification*: A classic, text-book emotional exclamation with zero analytical vocabulary or overarching debate potential.


* **Text**: *"The tactical shift to an inverted fullback allowed them to completely dominate the central midfield transition phases during the second half"*
    * **Predicted**: `analysis` (Confidence: 0.94)

    
* **Text**: *"Uruguay PR’s team is generational. I’m not sure why they are so highly regarded, this isn’t 2010 anymore"*
    * **Predicted**: `reaction` (Confidence: 0.56) [Actual: `hot_take`]

---

## Model Reflection: Captured vs. Intended
Our fine-tuning setup successfully captured surface-level vocabulary signals. It perfectly isolated `reaction` posts because emotional venting features unmistakable punctuation and token distributions. 

However, there is a clear gap between our intent and the model's actual decision boundary regarding opinions. We intended the model to capture *epistemic intent* (objective data vs. controversial beliefs). Instead, the model overfit to *topic signals*. If a post spoke about tactics, it classified it as `analysis`, completely bypassing whether the statement was actually an aggressive, subjective claim. 

To fix this boundary regression, we would need to enrich our dataset with explicitly labeled "controversial tactical opinions" to break the model's false association between soccer jargon and objective analysis.

---

## Specification Reflection
* **Guidance**: The clear separation of the test set from the training and validation workflows guarded us against data leakage, accurately revealing that our model was severely underperforming the baseline model on unseen text long before any deployment.
* **Divergence**: We consciously adjusted our hyperparameters away from the notebook defaults, raising our training run from 3 epochs to **5 epochs** and moving the learning rate to **$3 \times 10^{-5}$**. This change was made to ensure our training loss converged smoothly given the highly condensed vocabulary constraints of our small 141-example training set split.

---

## AI Usage Disclosure
1.  **Code Hyperparameter Calibration**: I directed ChatGPT to optimize our `TrainingArguments` block when early evaluation showed poor accuracy. It suggested moving the learning rate up to $3 \times 10^{-5}$ and extending training upto 5 epochs to maximize optimization steps over a micro-dataset.
2.  **Error Pattern Discovery**: We provided Claude with our list of 7 misclassified test data points. It surfaced the structural insight that our fine-tuned classifier completely loses precision when a controversial statement features formal football taxonomy.