<div class="topnav">
<a href="#problem">Problem</a>
<a href="#data">Data</a>
<a href="#method">Method</a>
<a href="#results">Results</a>
<a href="#discussion">Discussion</a>
<a href="#resources">Resources</a>
<a href="#references">References</a>
</div>


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

Our methodology consists of three major components.

---

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
- sex
- bowel movement (stool quality)

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