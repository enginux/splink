# 🚀 Splink Workflow Guide

Welcome to the **Splink Workflow Guide**! 

This repository provides a structured approach to entity resolution using **Splink**, enabling efficient record linkage and deduplication at scale.

## 📌 Overview
Splink is a Python package for probabilistic record linkage (entity resolution) that allows you to deduplicate and link records from datasets without unique identifiers.

This workflow demonstrates best practices to:
- Clean and preprocess data for entity resolution.
- Configure Splink for optimal matching.
- Perform scalable and efficient record linkage.
- Evaluate and fine-tune results for accuracy.

## 🏗️ Installation
To get started, install the required dependencies with my chosen backend - spark:

```sh
!pip install 'splink[spark]'
```

## 🚦 Getting Started
Follow these steps to use this workflow:

1. **Prepare your data** – Ensure datasets are cleaned and formatted properly.
2. **Define linkage rules** – Configure comparison and scoring functions.
3. **Run Splink** – Execute entity resolution with your chosen backend (Spark, DuckDB, etc.).
4. **Evaluate results** – Analyze and refine matches for accuracy.

## 🚀 Quickstart 
To get a basic Splink model up and running, use the following code. It demonstrates how to:

Estimate the parameters of a deduplication model
Use the parameter estimates to identify duplicate records
Use clustering to generate an estimated unique person ID.
```python
import splink.comparison_library as cl
from splink import DuckDBAPI, Linker, SettingsCreator, block_on, splink_datasets

db_api = DuckDBAPI()

df = splink_datasets.fake_1000

settings = SettingsCreator(
    link_type="dedupe_only",
    comparisons=[
        cl.NameComparison("first_name"),
        cl.JaroAtThresholds("surname"),
        cl.DateOfBirthComparison(
            "dob",
            input_is_string=True,
        ),
        cl.ExactMatch("city").configure(term_frequency_adjustments=True),
        cl.EmailComparison("email"),
    ],
    blocking_rules_to_generate_predictions=[
        block_on("first_name", "dob"),
        block_on("surname"),
    ]
)

linker = Linker(df, settings, db_api)

linker.training.estimate_probability_two_random_records_match(
    [block_on("first_name", "surname")],
    recall=0.7,
)

linker.training.estimate_u_using_random_sampling(max_pairs=1e6)

linker.training.estimate_parameters_using_expectation_maximisation(
    block_on("first_name", "surname")
)

linker.training.estimate_parameters_using_expectation_maximisation(block_on("email"))

pairwise_predictions = linker.inference.predict(threshold_match_weight=-5)

clusters = linker.clustering.cluster_pairwise_predictions_at_threshold(
    pairwise_predictions, 0.95
)

df_clusters = clusters.as_pandas_dataframe(limit=10)
```

## 🔧 Configuration
You can tweak the following settings for better results:
- **Blocking rules**: Define rules to limit the number of comparisons.
- **Scoring functions**: Adjust parameters for similarity calculations.
- **Threshold tuning**: Optimize match acceptance criteria.

## 📂 Project Structure
```
📁 splink/
├── 📜 README.md
├── 📦 requirements.txt
├── 📁 data/         # Sample datasets
├── 📁 notebooks/    # Jupyter notebooks  
│   ├── 📁 tutorials/  # Step-by-step guides and explanations  
│   ├── 📁 examples/   # Practical use cases and sample workflows  
├── 📁 scripts/      # Python scripts for automation  
└── 📁 results/      # Output match results  

```

## 🎯 Use Cases
This workflow is useful for:
- **Customer data deduplication**
- **Fraud detection through identity matching**
- **Merging datasets across different sources**
- **etc. imagination is the limit**

## 🤝 Contributing
Contributions are welcome! Feel free to submit issues, suggestions, or pull requests.

---
Happy Linking! 🚀
