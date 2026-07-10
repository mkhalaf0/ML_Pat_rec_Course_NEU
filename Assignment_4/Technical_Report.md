# Technical Report — Predicting Titanic Survival with a Decision Tree

**EECE 6544, Assignment #04**

## 1. What are we predicting, and from which parameters?

The target is `Survived` (1 = survived, 0 = died). Of the 891 passengers, 342
survived — a base survival rate of **38.4%**. We predict survival from seven
parameters: passenger class (`Pclass`), sex (`Sex`), age (`Age`), siblings/spouses
aboard (`SibSp`), parents/children aboard (`Parch`), fare paid (`Fare`), and port of
embarkation (`Embarked`). Identifier and free-text columns (`PassengerId`, `Name`,
`Ticket`, `Cabin`) were dropped because they are not usable parameters.

## 2. How was the data cleaned?

Three columns had missing values: `Age` (177 missing), `Cabin` (687, dropped), and
`Embarked` (2 missing). We filled `Age` with the median and `Embarked` with the most
common port, then encoded the categorical parameters as integers (`Sex`: male = 0,
female = 1; `Embarked`: S = 0, C = 1, Q = 2). After cleaning, zero values were
missing. The data was split 80/20 into 712 training and 179 test passengers,
stratified on the target so both sets keep the ~38% survival rate.

## 3. What model was trained?

A `DecisionTreeClassifier` (Gini impurity) constrained to `max_depth=4` and
`min_samples_leaf=10`. These limits keep the tree readable and prevent it from
memorizing the training set. The fitted tree has depth 4 and 13 leaves.

## 4. How accurate is the model on unseen passengers?

On the 179 held-out passengers:

| Metric | Value |
|---|---|
| Accuracy | 0.777 |
| Precision (survived) | 0.809 |
| Recall (survived) | 0.551 |
| F1 (survived) | 0.655 |

Confusion matrix (rows = actual, columns = predicted):

| | pred died | pred survived |
|---|---|---|
| **died** | 101 | 9 |
| **survived** | 31 | 38 |

The model is conservative about predicting survival: when it does predict
"survived" it is right ~81% of the time (high precision), but it misses about
45% of actual survivors (recall 0.55), driven by the 31 survivors it labels as
deaths.

## 5. Is the tree meaningfully better than a trivial baseline?

Yes. A majority-class baseline that predicts "died" for everyone scores **0.615**
accuracy. The decision tree scores **0.777** — an improvement of **+0.162** (about
16 percentage points) using only a handful of readable rules.

## 6. Which parameters matter most?

Feature importances (share of impurity reduction):

| Parameter | Importance |
|---|---|
| Sex | 0.598 |
| Pclass | 0.203 |
| Age | 0.089 |
| Fare | 0.070 |
| Embarked | 0.039 |
| Parch | 0.001 |
| SibSp | 0.000 |

**Sex alone accounts for ~60%** of the model's decisions, with passenger class a
distant second at ~20%. Age and fare fine-tune the result; `SibSp` and `Parch`
(family size) contribute almost nothing.

## 7. What rules did the tree actually learn?

The very first split is on **sex**, which matches the historical "women and
children first" evacuation policy:

- **Males** (`Sex = 0`) are predicted to *die* — except the youngest boys
  (`Age ≤ 3.5`), who are predicted to survive. This is the "children first" effect.
- **Females** (`Sex = 1`) are predicted to *survive*, especially in first and
  second class (`Pclass ≤ 2`). Third-class women are split further on port of
  embarkation and fare, where their survival becomes less certain.

Passenger class and fare consistently separate wealthier upper-deck passengers,
who had better access to lifeboats, from third-class passengers below deck.

## 8. Are any results surprising?

Not really — and that is a good sign. The tree independently rediscovered the two
best-documented facts about the disaster: being female and traveling in a higher
class dramatically raised the odds of survival. The one nuance worth flagging is
the model's low recall for survivors: because most passengers died, the tree
defaults toward predicting death and misses roughly half of the true survivors,
even though the survivors it does flag are usually correct.
