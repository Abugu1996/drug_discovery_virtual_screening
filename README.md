# drug_discovery_virtual_screening
Virtual screening dataset of 2,000 compounds tested against 400 protein targets across 17 columns. It captures compound chemistry (molecular weight, LogP, hydrogen bonds, polar surface area), protein context (length, isoelectric point, binding site size), and outcomes (binding affinity, plus a binary active flag set at affinity 7.0 or higher).


# project objective
Make sense of a virtual screening run through five focused dashboards. The questions driving the build were practical: which compounds bind well, which protein targets pull in the most hits, and what separates an active compound from a dead one. A second goal was to find out whether a model can call activity from a compound's properties by itself, without peeking at the binding score it was graded against.

# dataset
A single file, `drug_discovery_virtual_screening.csv`: 2,000 compounds tested against 400 protein targets across 17 columns. That works out to about 5 compounds per target.
The columns fall into three buckets.
- Compound chemistry: molecular weight, LogP, hydrogen bond donors and acceptors, rotatable bonds, polar surface area, calculated cLogP.
- Protein context: protein length, isoelectric point, hydrophobicity, binding site size.
- Interaction and outcome: a molecular weight ratio, a LogP-by-pI interaction term, binding affinity, and a binary `active` flag.
Three columns came in with holes. LogP, polar surface area, and hydrophobicity each lost 60 values. One polar surface area reading was negative, which can't happen physically.
Cleaning followed the Missing Values Audit. I filled the gaps with each column's median and flipped the negative surface area to its absolute value. No rows were thrown out, so all 2,000 compounds carried through.

## Key Questions (KPIs)
The first dashboard puts seven numbers up front so anyone opening the file gets the headline before reading a chart.
 KPI  Value 
 Total compounds 2,000
 Total targets  400 
 Active compounds  608 
 Hit rate | 30.4% 
 Average binding affinity  6.53 
 Average molecular weight  456.8 
 Average LogP  3.48 

## Process

1. Read the raw CSV and checked every column for type, range, missing values, and duplicates. Zero duplicate compounds, zero duplicate rows.
2. Cleaned the data per the audit: median fill on the three gappy columns, sign fix on the one bad surface area value.
3. Computed the KPIs above.
4. Cut the work into five separate dashboards so each fits a laptop screen, rather than packing forty charts onto one canvas.
5. Drew every chart in matplotlib using one fixed brown and gray palette for a consistent look.
Stack: Python, pandas, NumPy, scikit-learn, matplotlib.

## Dashboard Overview

1. Executive Overview. The seven KPIs, the active versus inactive split, the spread of binding scores, and the ten targets carrying the most compounds.
2. Molecular Discovery. Distributions for molecular weight, LogP, and polar surface area. Two scatter plots set molecular weight and LogP against binding affinity. A table lists the eight strongest binders.
3. Protein Target Analysis.Targets ranked by mean affinity, a heatmap of the twenty busiest proteins against affinity bands, hit rate per target, and the spread of protein lengths.
4. Structure Activity Relationship. Box plots compare active and inactive compounds on molecular weight, LogP, polar surface area, and hydrophobicity. A correlation matrix ties the numeric columns together.
5. Predictive Modelling. Feature importance, a confusion matrix, the ROC curve, and the precision, recall, F1, and AUC scores for the random forest.

## Project Insights

One finding reshapes how the whole dataset should be read, so it goes first.
-The `active` label is a cutoff on binding affinity. Every active compound scores 7.0 or higher. Every inactive one scores below 7.0. There's no overlap at all. The label holds no information beyond the affinity column it was cut from. Drop affinity into a classifier and you score a fake 1.000 on every metric, which tells you nothing. That's why affinity stays out of the model.
-LogP does most of the work. Active compounds run greasier. Their median LogP is 4.81 against 2.96 for the inactive group. LogP correlates 0.55 with the activity flag and 0.60 with binding affinity. In the model, the LogP-by-pI interaction term and LogP itself ranked as the two strongest predictors by a clear margin.
-Size and polarity barely register. Molecular weight, polar surface area, and hydrophobicity show almost no relationship with activity. Their correlations with the label sit at -0.03, 0.04, and 0.04. Active and inactive box plots for these three nearly sit on top of each other.
-The model holds up on chemistry alone. With affinity removed, the random forest still reaches an AUC of 0.952 and an F1 of 0.808 (precision 0.859, recall 0.763). A compound's measured properties carry enough signal to flag likely binders before any docking score exists.
-The hit rate looks generous. A 30.4% pass rate runs far above what a real screen returns, where strong hits are scarce. The clean 7.0 threshold and the near-balanced classes point to a generated or teaching dataset rather than raw lab output.


## Conclusion
The screen splits along lipophilicity. Greasier compounds bind harder, and LogP by itself explains most of the gap between hits and misses. The activity flag is a renamed affinity threshold, so anyone reusing this data should either model affinity directly or drop it from the features, never feed both. Working from chemistry alone, the forest still calls hits well enough to triage candidates ahead of docking. The five dashboards read top to bottom, headline numbers first and model last, so a reviewer can follow the logic without opening the code.
