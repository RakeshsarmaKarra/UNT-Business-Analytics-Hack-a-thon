![Image](https://github.com/RakeshsarmaKarra/UNT-Business-Analytics-Hack-a-thon/blob/main/UNT_Business_Analytics_Hack_a_thon.png)

# Project overview
This project was completed as part of the UNT Business Analytics Hack‑a‑thon, focusing on predicting warranty cost segments for heavy‑duty trucks based on their build configurations and historical claim patterns.

Using truck style and option codes combined with claim and labor cost scales, the goal is to help the manufacturer:
Identify high‑risk configurations early in the design process.
Understand which options and option bundles are most associated with elevated warranty costs.
Build a multiclass classification model to predict warranty‑cost levels for new vehicle builds.

# Business questions
- Which individual attributes or option codes are significantly associated with increased warranty costs?
- Are there specific pairs of attributes that interact in a way that exacerbates warranty claims?
- What modeling approaches can effectively quantify these relationships and offer predictive insights for future vehicle configurations?

# Data description
Two main datasets were provided by the hackathon organizers:

## Configuration data

Truck – unique truck identifier                                                                                                                                
Style – truck style / configuration family                                                                                                                                
Attribute 1–8 – option codes (e.g., “Option 83”, “Option 181”), representing installed components or packages                                                                                                

## Claims data
Truck Number, Claim Number                                                                                                
Scale Claim Cost – ordinal: Very Low, Low, Medium, High, Very High                                                                                                
Scale Labor Cost – same ordinal scale as above                                                                                                

After cleaning and joining on truck identifiers, a final modeling table was created with:                                                                                                                                
Configuration: Truck, Style, Attribute 1–8                                                                                                                                
Encoded claim features: Scale Claim Cost Encoded, Scale Labor Cost Encoded                                                                                                                                
Target: Warranty Cost (ordinal class derived from combined cost information)                                                                                                                                
Ordinal scales for claim and labor cost were encoded numerically (1–5) to preserve their ordering.                                                                                                

# Exploratory analysis & key drivers
## Cost distributions
Warranty costs are highly skewed toward lower‑cost classes, with “Low” and “Very Low” dominating and only a small share of Medium/High/Very High cases. This creates a class‑imbalance challenge for modeling.
​
## Correlation insights
A correlation matrix on the numeric features shows:                                                                                                
Scale Claim Cost Encoded and Scale Labor Cost Encoded have the strongest correlation with Warranty Cost.                                                                
Among configuration fields, Attribute 1–4 show the most meaningful relationships with cost features and with each other, while Attribute 7–8 and Truck ID are much weaker.                                                                
Style has a moderate correlation with warranty cost, but its impact is smaller than the detailed option codes in early attribute positions.                                                                
![Heatmap](https://github.com/RakeshsarmaKarra/UNT-Business-Analytics-Hack-a-thon/blob/main/Heatmap.jpg)

## Interpretation:
High warranty cost is primarily driven by higher claim/labor severities, and specific option codes in Attribute 1–4 are more strongly associated with elevated cost than later attributes.

# Modeling approach
The prediction task is a multiclass classification problem (warranty‑cost segment). Three main models were evaluated:
​
- Multinomial Logistic Regression (LogisticRegression(multi_class='multinomial'))
- One‑Vs‑Rest Logistic Regression (OneVsRestClassifier(LogisticRegression))
- XGBoost Multiclass Classifier (XGBClassifier with label‑encoded targets)

All models were trained on the same x_train, y_train split and evaluated on a held‑out x_test, y_test set.                                                                 
Metrics: Accuracy, classification report, confusion matrix, and macro ROC–AUC (OvR).
​

## Model comparison
| Model                               | Accuracy | ROC–AUC (macro, OvR) | Notes                                                                 |
|-------------------------------------|----------|----------------------|-----------------------------------------------------------------------|
| OneVsRestClassifier (LogReg, OvR)   | 0.4934   | 0.7407               | Only ROC–AUC computed; moderate overall discrimination.               |
| LogisticRegression (multinomial)    | 0.4946   | 0.5521               | Collapses to dominant class; very poor recall for minority classes.   |
| XGBClassifier (multiclass)          | 0.5091   | 0.8016               | Best ROC–AUC; non‑zero recall on multiple minority cost classes.      |


Captures non‑linearities and interactions among Style, Attribute 1–8, and cost features, unlike linear models.
Improves macro ROC–AUC from ≈0.55 (multinomial logistic regression) to ≈0.80, and improves recall on minority high‑cost classes, which are most important from a risk perspective.

![roc-auc](https://github.com/RakeshsarmaKarra/UNT-Business-Analytics-Hack-a-thon/blob/main/roc-auc%20XGBoost.png)
​
# Answers to the business questions
## 1. Individual attributes / option codes driving warranty cost
Higher values of Scale Claim Cost Encoded and Scale Labor Cost Encoded are strongly associated with higher Warranty Cost classes.
Among configuration attributes, Attribute 1–4 are the most influential; they show stronger correlations with warranty cost and appear higher in tree‑based feature importance than Attributes 7–8 or the truck ID.
Warranty cost is most strongly driven by encoded claim and labor severities and by specific option codes in the early attribute positions (Attribute 1–4).
​
## 2. Interactions that exacerbate warranty claims
Correlations among Attribute 1–4 and between these attributes and the cost encodings suggest that certain bundles of options tend to co‑occur in higher‑cost builds.
The significant performance gap between XGBoost and logistic regression confirms that interaction effects (e.g., Attribute 1 × Attribute 3, plus high claim/labor levels) matter: tree ensembles exploit these interactions, while linear models cannot.
No single option is solely responsible; specific combinations of early attributes, together with higher claim/labor severity, drive vehicles into high‑warranty‑cost segments.
​
## 3. Effective modeling approaches and predictive insights
Multinomial Logistic Regression gives a baseline but fails to capture interactions and performs poorly on rare high‑cost classes.
One‑Vs‑Rest Logistic Regression improves ROC–AUC but still uses linear boundaries and lacks interaction modeling.
XGBoost Multiclass provides the best trade‑off between accuracy and macro ROC–AUC, with materially better performance on critical, high‑cost segments.
​
Tree‑based ensemble methods like XGBoost are the most suitable choice for quantifying the relationships between configuration and warranty cost and for predicting cost segments for new vehicle builds.
​

# These predictions can be used for:
- Pre‑launch risk assessment of proposed builds.
- Scenario testing (e.g., “What happens to warranty risk if we change Attribute 2 and Attribute 3?”).
- Design and pricing decisions, prioritizing safer options or pricing high‑risk bundles appropriately.
