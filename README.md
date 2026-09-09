# NovaBank — Predictive Campaign Targeting at Scale

**Quantic MSBA · Analytics Methods and Frameworks Project**

---

## Note on problem framing

The project brief describes NovaBank as a **churn/retention** case. The dataset it links to —
[UCI ML Repository #222, Bank Marketing](https://archive.ics.uci.edu/dataset/222/bank+marketing) —
contains **no churn, exit, or attrition label**. Its target `y` records whether a client
**subscribed to a term deposit** after a marketing call.

Rather than fabricate a label the data cannot support, this project uses the provided dataset for the
problem it genuinely answers: **which customers should the call centre contact?** The decision
structure is unchanged — false positives are wasted agent calls, false negatives are missed
subscriptions.

## Headline results

| Model | AUC-ROC | PR-AUC |
|---|---|---|
| Logistic Regression (baseline) | 0.800 | 0.446 |
| Random Forest | 0.811 | 0.484 |
| **Gradient Boosting (selected)** | **0.815** | **0.490** |

Base subscription rate: 11.3%

### Recommended policy — call the top 20% by score

| Capacity | Subscriptions won | Random dialling | Lift |
|---|---|---|---|
| Top 10% | 435 (46.9% of all) | 93 | 4.69x |
| **Top 20%** | **616 (66.4% of all)** | **186** | **3.32x** |
| Top 30% | 692 (74.6% of all) | 278 | 2.49x |

Two-thirds of all available subscriptions from one-fifth of the calls.

## Two findings worth reading the notebook for

**1. `duration` is leakage.** Call length is unknown until the call ends. Including it gives
AUC 0.951; excluding it gives 0.815. The dataset authors state it must be dropped for any deployable
model. Section 4 quantifies the gap explicitly.

**2. Random splitting overstates performance.** Chronological validation (train May 2008–early 2010,
test on the months that follow) drops AUC from **0.815 to 0.634**. The two strongest features are
`nr.employed` and `euribor3m` — macro series that trend over the campaign period. The model partly
learns *when in the economic cycle* a call was placed rather than *who is a good prospect*. Removing
the macro block costs little in-sample (0.815 → 0.783) but **improves** the forward result
(0.634 → 0.683).

## Files

| File | Description |
|---|---|
| `NovaBank_Campaign_Targeting.ipynb` | Full analysis, executed with outputs |
| `chart_eda.png` | Six-panel exploratory analysis |
| `chart_models.png` | ROC, precision-recall, permutation importance |
| `chart_decision.png` | Cumulative gains, capacity scenarios, robustness |

## Reproducing

1. Download `bank-additional.zip` from the [UCI page](https://archive.ics.uci.edu/dataset/222/bank+marketing)
2. Place `bank-additional-full.csv` beside the notebook
3. Run all cells — `random_state=42` throughout

Requires: `pandas`, `numpy`, `scikit-learn`, `matplotlib`

## Citation

Moro, S., Cortez, P. & Rita, P. (2014). *A Data-Driven Approach to Predict the Success of Bank
Telemarketing.* Decision Support Systems, 62:22-31.

## AI usage

Claude (Anthropic) used for code scaffolding, identifying the `duration` leakage from the dataset
documentation, and suggesting the temporal-split test. All model choices, interpretations, risk
assessments and recommendations are the author's own.
