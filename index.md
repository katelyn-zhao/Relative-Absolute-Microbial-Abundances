# Relative vs. Absolute Microbial Abundances

## Navigation
- [Problem](#problem)
- [Data](#data)
- [Method](#method)
- [Results](#results)
- [Discussion](#discussion)
- [Resources](#resources)
- [References](#references)

---

## Problem

The gut microbiome plays an important role in human health, and microbial composition has been linked to conditions such as inflammatory bowel disease, Parkinson’s disease, Alzheimer’s disease, and autism spectrum disorder.

However, most microbiome datasets contain only **relative abundances**, since absolute quantification is more expensive and harder to obtain. Relative abundance data is useful, but it is compositional, meaning each feature is measured as a proportion of the total rather than a direct quantity.

Our project asks whether we can:

1. recover meaningful biological signals from relative and absolute abundance data,
2. compare how well each representation supports downstream prediction tasks, and
3. use relative abundance data to estimate total microbial load and generate **synthetic absolute abundance tables**.

<p align="center">
  <img src="assets/rel_vs_abs_wetlab_workflow.png" width="700">
</p>

---

## Data

Our dataset contains paired microbiome feature tables and limited host metadata.

### Inputs
- **Metadata:** age, body mass index (BMI), sex, bowel movement type, and related host variables
- **Relative abundance table:** sample-by-feature table based on sequencing-derived counts
- **Absolute abundance table:** sample-by-feature table estimating true microbial quantity per unit sample

### Preprocessing
- Both tables were filtered using **Microbiome Coverage (micov)** at a 30% threshold
- Samples, metadata, and feature tables were aligned by sample ID
- Blanks and controls were removed
- Key metadata variables were cleaned before analysis

### Why this matters
Relative abundance data is much easier to collect, but absolute abundance better reflects the true quantity of microbes. This project evaluates whether relative abundance can be used to recover similar biological conclusions and approximate absolute measurements.

---

## Method

Our methodology consists of four major components: data preparation, diagnostic checks for technical bias, predictive modeling on host metadata, and modeling absolute microbial abundance from relative abundance.

---

### 1. Data Preparation

We were provided with four microbiome feature tables:

- metagenomic relative abundance
- metagenomic absolute abundance
- metatranscriptomic relative abundance
- metatranscriptomic absolute abundance

Each table is structured as **sample × feature**, where features represent microbial taxa or gene expression measurements. A metadata file contains host variables such as age, BMI, sex, and bowel movement quality.

In this project we primarily used the **metagenomic abundance tables together with host metadata**.

#### Metadata Cleaning

Although the feature tables had already undergone filtering for coverage, prevalence, and control samples, the metadata required additional cleaning.

Key steps included:

- standardizing missing data values (`N/A`, `not applicable`, etc.)
- removing implausible values (e.g., BMI = 500 or age = 150)
- correcting data type inconsistencies (e.g., `"TRUE"` vs `True`)
- removing duplicate records

After cleaning, the metadata was aligned with the microbiome tables using sample IDs.

---

### 2. Plate Bias Diagnosis (Batch Effect Check)

Before training prediction models, we examined whether **technical sequencing plate effects** influenced microbial profiles.

If microbiome features strongly encode plate identity, predictive models may capture technical variation rather than biological signal.

#### Approach

1. Combined training, validation, and test sets.
2. Treated `_diluted` samples as belonging to their original plates (confirmed by the wet-lab team).
3. Trained a **Random Forest classifier** to predict sequencing plate from microbial features.
4. Evaluated performance using **5-fold stratified cross-validation**.

#### Baseline

A majority-class baseline achieved: Accuracy ≈ 0.028


#### Model Results

| Data Representation | Accuracy |
|--------------------|---------|
| Absolute abundance | ~0.334 |
| Relative abundance | ~0.190 |
| Estimated absolute abundance | ~0.204 |

All models performed above baseline, indicating **some plate-related signal** in the data.

However, prediction accuracy remained moderate, and this analysis was used only as a **diagnostic check**, not as a core result.

---

### 3. Predictive Modeling of Host Metadata

Our second objective was to compare whether **absolute abundance or relative abundance better supports predictive tasks**.

We trained models to predict the following host variables:

- age  
- sex  
- body mass index (BMI)  
- bowel movement category  

#### Dataset Splits

To avoid data leakage, the dataset was split as: 60% training, 20% validation, and 20% test.


- Training set → model fitting  
- Validation set → hyperparameter selection  
- Test set → final evaluation  

All preprocessing decisions were based **only on the training set**.

---

### 4. Age Prediction

We evaluated both classification and regression tasks.

#### Age Classification

Age was grouped into six bins: under 20, 20–35, 35–50, 50–65, 65–80, and over 80.


These bins follow ranges used in the **American Gut Microbiome Project**.

Models trained:

- Random Forest classifier

Evaluation metrics:

- accuracy
- macro-averaged ROC-AUC

We also performed **bootstrap testing (1000 iterations)** to compute confidence intervals when comparing relative vs absolute abundance models.

#### Age Regression

To predict numerical age we trained:

- Random Forest Regressor  
- Gradient Boosting Regressor  
- RBF Support Vector Regressor

Evaluation metrics:

- R²
- Mean Absolute Error (MAE)

---

### 5. Sex Prediction

To predict biological sex, we trained three classifiers:

- Random Forest  
- Logistic Regression  
- RBF Support Vector Machine

Samples without male or female labels were removed.

Evaluation metrics:

- accuracy
- macro-averaged ROC-AUC

Bootstrap resampling (1000 iterations) was used to test differences between models trained on relative vs absolute abundance.

Feature importance was analyzed using **SHAP (SHapley Additive Explanations)** for tree-based models.

---

### 6. BMI Prediction

BMI was evaluated using both **regression and classification tasks**.

#### Feature Processing

Only microbial taxa columns beginning with `"G"` were used.

To remove extremely sparse taxa: Prevalence threshold = 40%

This threshold was selected after testing several candidates (0.1-0.5) and choosing the best validation performance.

All abundance features were transformed using: log(x + 1) 
to reduce skewness.

#### Regression Models

- Random Forest Regressor  
- HistGradientBoosting Regressor  
- Support Vector Regressor

Evaluation metrics:

- R²
- Mean Absolute Error

A **mean-BMI baseline model** was used for comparison.

#### Classification Models

BMI was converted into clinical categories: underweight (<18.5), normal (18.5–25), overweight (25–30), obese (30–40), and severe obese (>40).


Models trained:

- Random Forest
- HistGradientBoosting
- Support Vector Machine

Evaluation metrics:

- accuracy
- macro F1
- weighted F1

We also generated **confusion matrices and multiclass ROC curves**.

To test statistical significance between representations, we performed **bootstrap resampling (10,000 iterations)**.

Feature interpretation was performed using **SHAP analysis**.

---

### 7. Stool Quality Prediction

The metadata variable `bowel_movement` describes stool quality.

Responses were mapped to three categories: normal, diarrhea, and constipated.


We trained the same classifiers used in BMI classification:

- Random Forest
- HistGradientBoosting
- Support Vector Machine

Evaluation metrics:

- accuracy
- macro F1
- weighted F1

Bootstrap testing and SHAP analysis were again used to evaluate model differences and feature importance.

---

### 8. Differential Abundance Analysis

We performed differential abundance testing using two frameworks:

- **ANCOM-BC**
- **BIRDMAn**

These methods identify taxa whose abundance significantly differs across host phenotypes.

#### ANCOM-BC Pipeline

Using **QIIME2**, we:

1. removed blanks and control samples
2. aligned microbiome tables with metadata
3. retained key covariates (age, BMI, sex, bowel movement)
4. binned age and BMI into interpretable ranges
5. fitted multifactor ANCOM-BC models

Outputs included:

- log-fold change estimates
- FDR-adjusted significance values
- genus-level heatmaps

We also conducted **PERMANOVA using Bray-Curtis distance** to quantify the variance explained by host covariates.

#### BIRDMAn Analysis

BIRDMAn models microbial counts using Bayesian regression.

Unlike ANCOM-BC, which corrects compositional bias via sampling fractions, BIRDMAn models the **generative count process** directly.

Results from both methods were compared using heatmaps and credible intervals.

---

### 9. Modeling Absolute Abundance from Relative Abundance

Our final objective was to estimate **absolute microbial abundance using only relative abundance data**.

In the NPH study, absolute abundance is computed as: A_true = Relative_abundance × microbial_load


where microbial load is measured using spike-in DNA.

Because this lab procedure is expensive, we attempted to **predict microbial load directly from relative abundance features**.

#### Model Target

The model predicts microbial load in log space: y = log(1 + load). 
After prediction, we apply a bias-corrected back-transformation: load_hat = exp(y + σ²/2) − 1. 
Predicted absolute abundance is then computed as: A_pred = relative_abundance × predicted_load.

---

### 10. Feature Engineering for Load Prediction

To capture both compositional and sequencing-depth information, we constructed a feature matrix containing:

1. centered log-ratio transformed compositions
2. normalized read counts
3. log total read counts
4. sparsity indicators
5. log read counts per taxon

These representations provide complementary information about microbial structure and sequencing intensity.

---

### 11. Model Evaluation

We evaluated load prediction using:

- R²
- MAE
- RMSE
- Spearman correlation
- median fold error

To compare predicted and true abundance tables, we used:

- **Bray-Curtis dissimilarity**
- **Mantel correlation test**
- **Principal Coordinates Analysis (PCoA)**
- **Procrustes analysis**

These analyses measure how well predicted abundance preserves **sample-to-sample ecological structure**.

---

### 12. Downstream Validation

To test whether predicted abundance tables remain useful for biological analysis, we repeated the metadata prediction tasks (age, sex, BMI, stool quality) using the synthetic tables.

This evaluates whether predicted abundance preserves the **predictive signal present in the original absolute abundance data**.

---

### 13. Cross-Dataset Validation

To evaluate generalizability, we retrained the best-performing model on a separate **long-read sequencing dataset**.

Because this dataset had fewer samples and different features, the model was retrained rather than directly transferred.

We also subsampled the original dataset to match the smaller sample size to determine whether differences in performance were driven by:

- sequencing modality  
- or dataset size.


### 1. Differential Abundance Analysis

We use **ANCOM-BC** to identify taxa that are enriched or depleted across host phenotype groups. This allows us to compare biological signals across abundance representations.

Our workflow:
- clean metadata and feature tables
- match sample IDs
- retain important covariates such as age, BMI, sex, and bowel movement type
- remove implausible values
- discretize age and BMI into bins for clearer interpretation
- fit multifactor ANCOM-BC models with explicit reference levels
- summarize significant genus-level effects
- visualize results with heatmaps

To assess **signal preservation**, we apply the same pipeline to:
- true held-out absolute abundance tables
- predicted absolute abundance tables

This lets us evaluate whether synthetic absolute profiles recover similar effect directions and relative magnitudes.

<p align="center">
  <img src="assets/all_heatmaps_side_by_side.png" width="800">
</p>

### 2. Predictive Modeling

We build baseline supervised learning models to predict host metadata variables from microbiome features, including:
- age
- body mass index (BMI)
- Bristol Stool Scale
- related host phenotypes

This helps us compare how relative and absolute abundance representations perform in downstream prediction tasks.

### 3. Modeling Absolute Abundance from Relative Abundance

Our final goal is to estimate absolute abundance using only relative abundance data.

The idea is:

1. use relative abundance features to predict the **total absolute microbial load** for each sample,
2. multiply the predicted total by each feature’s relative proportion,
3. generate a **synthetic absolute abundance table**.

Because total abundance varies widely across samples, we apply a **log(1 + x)** transformation to the prediction target. We experiment with multiple regression approaches, including:
- linear models
- random forest
- gradient boosting

<p align="center">
  <img src="assets/abundance_model_training.png" width="700">
</p>

---

## Results

### Differential Abundance

Our differential abundance analysis compares whether important taxa and phenotype-associated signals are preserved across:
- true absolute abundance data
- predicted absolute abundance data

We evaluate whether predicted absolute tables recover similar:
- significant taxa
- effect directions
- relative effect magnitudes

### Predictive Modeling

We compare model performance across abundance representations for host phenotype prediction tasks. These results help determine whether absolute abundance provides stronger predictive signal than relative abundance.

### Relative-to-Absolute Modeling

We evaluate the quality of predicted absolute abundance tables by comparing their structure to true absolute abundance profiles. Ordination and downstream analyses help assess how well the synthetic data preserves the original biological relationships.

<p align="center">
  <img src="assets/rpca_ordination_abundance.png" width="700">
</p>

---

## Discussion

This project studies the tradeoff between data quality and data availability in microbiome research.

Absolute abundance measurements are more biologically interpretable, but they are harder and more expensive to collect. Relative abundance data is far more common, so an effective method for estimating absolute abundance from relative abundance could make more datasets useful for downstream biological analysis.

More broadly, our work asks two key questions:

- How much biological signal is lost when using only relative abundance?
- Can predicted absolute abundance recover enough signal to support meaningful downstream analysis?

If successful, this framework could help researchers extend absolute-abundance-style analysis to datasets where only relative abundances are available.

---

## Resources

- **GitHub Repository:** [Add repo link here]
- **Final Report:** [Add report link here]
- **Poster:** [Add poster link here]
- **Demo / Notebook:** [Add demo link here]

---

## References

Add your references here in a consistent citation format.