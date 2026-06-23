#project 3 planning: r/soccer Text classification pipline

## 1. Taxonomy and lable definitions:
The goal is to classify the posts that came from tread/ from the soccer communities in to three mutually exclusive catagories based on lingustic structure, intent, and tone.
* **analysis**: posts containing objective tactical breakdowns, statistical metrics, player performance data,or team structural evaluations.
* *Example 1*: "Beyond the additional match for semifinalists, I feel that goalscoring/assist stats for WCs across expansions aren't really comparable due to the average quality of opponent decreasing over time."
* *Example 2*: "Statistically, teams that switch to a back-five in the final ten minutes concede 15% more chances due to dropping their defensive line too deep."

* **reaction**: purely emmosional expression, instant venting, commantery on the match "vibes",kits or immidiate match tread responses.
* *Example 1*: "Oh my god, I cannot believe they just missed that shot!"
* *Example 2*: "Who the fuck greenlit those Brazil shorts, it looks like they've got a massive piss stain."

* **hot_take**: Highly opinionated, provocative,or controversial statements of belief or theory that invite for hot depate.
* *Example 1*: "The tournament doesn't actually begin until the quarter-finals; everything before that is just a waste of time."
* *Example 2*: "Musiala is vastly overrated compared to Wirtz; he relies far too much on individual dribbling rather than elevating his teammates."

## 2. Hard annotation Descitions and egde cases:
During my data collection process, several structural bounderies proved difficult to manually classify. we resolved that using the following determinstic huristics:

* * sarcastic opionion Vs reactions*: If the post used highly hyperbolic(e.g., "decided to eat crayons instead"), it was classified as a `hot_take` if it posited an overall opinion on a team's status, or a `reaction` if it was purely a match-thread vent.
* * Opionionated tactics*:A tactical post that used opionionated words but remained anchored to structural formations((e.g., "dropping their defensive line too deep") was kept as `analysis` rather than `hot_take`.)


## 3. Data collection plann:

* **Target size** I will try to collect more than 200 posts fro r/soccer match threads and post match discusion. I also more focus on world cup post.
* **Distribution**: Stratified accross the three classes to make sure the baalance train split distiribution

## 4. Evaluation Metrics Reasoning
Given the balanced nature of the test set (11 reaction, 10 hot_take, 10 analysis), overall **Accuracy** serves as our primary comparative benchmark. However, because `hot_take` relies on structural nuance that models frequently misinterpret as technical analysis, **per-class F1-scores** will be monitored closely to expose boundary blindness.

## 5. AI Tooling Strategy
* **LLM Choice**: `llama-3.3-70b-versatile` via Groq API will be used as our zero-shot baseline.
* **Pattern Finding Assistance**: Claude was utilized to batch-analyze the text of wrong predictions to extract shared semantic features (e.g., text lengths, grammatical structures) that tripped up the transformer head.
