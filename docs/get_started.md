
# Getting Started

## ⚙️ Installation

To install the **TrustEval-toolkit**, follow these steps:

### **1. Clone the Repository**

```bash
git clone https://github.com/nauyisu022/TrustEval-toolkit.git
cd TrustEval-toolkit
```

### **2. Set Up a Conda Environment**

Create and activate a new environment with Python 3.10:

```bash
conda create -n trusteval_env python=3.10
conda activate trusteval_env
```

### **3. Install Dependencies**

Install the package and its dependencies:

```bash
pip install .
```



## 🤖 Usage

### **Configure API Keys**

Run the configuration script to set up your API keys:

```bash
python trusteval/src/configuration.py
```
![image](images/api_config.png)

### **Quick Start**

> *The following example demonstrates an **Advanced AI Risk Evaluation** workflow.*

### **Step 0: Set Your Project Base Directory**
```python
import os
base_dir = os.getcwd() + '/advanced_ai_risk'
```

#### **Step 1: Download Metadata**
```python
from trusteval import download_metadata

download_metadata(
    section='advanced_ai_risk',
    output_path=base_dir
)
```

#### **Step 2: Generate Datasets Dynamically**
```python
from trusteval.dimension.ai_risk import dynamic_dataset_generator

dynamic_dataset_generator(
    base_dir=base_dir,
)
```

#### **Step 3: Apply Contextual Variations**
```python
from trusteval import contextual_variator_cli

contextual_variator_cli(
    dataset_folder=base_dir
)
```

#### **Step 4: Generate Model Responses**
```python
from trusteval import generate_responses

request_type = ['llm']  # Options: 'llm', 'vlm', 't2i'
async_list = ['your_async_model']
sync_list = ['your_sync_model']

await generate_responses(
    data_folder=base_dir,
    request_type=request_type,
    async_list=async_list,
    sync_list=sync_list,
)
```

#### **Step 5: Evaluate and Generate Reports**

1. **Judge the Responses**
    ```python
    from trusteval import judge_responses

    target_models = ['your_target_model1', 'your_target_model2']
    judge_type = 'llm'  # Options: 'llm', 'vlm', 't2i'
    judge_key = 'your_judge_key'
    async_judge_model = ['your_async_model']

    await judge_responses(
        data_folder=base_dir,
        async_judge_model=async_judge_model,
        target_models=target_models,
        judge_type=judge_type,
    )
    ```

2. **Generate Evaluation Metrics**
    ```python
    from trusteval import lm_metric

    lm_metric(
        base_dir=base_dir,
        aspect='ai_risk',
        model_list=target_models,
    )
    ```

3. **Generate Final Report**
    ```python
    from trusteval import report_generator

    report_generator(
        base_dir=base_dir,
        aspect='ai_risk',
        model_list=target_models,
    )
    ```

Your `report.html` will be saved in the `base_dir` folder. For additional examples, check the `examples` folder.



# Trustworthiness Report

A detailed trustworthiness evaluation report is generated for each dimension. The reports are presented as interactive web pages, which can be opened in a browser to explore the results. The report includes the following sections:

> *The data shown in the images below is simulated and does not reflect actual results.*


## Test Model Results
Displays the evaluation scores for each model, with a breakdown of average scores across evaluation dimensions.
![Test Model Results](images/test_model_results.png)

## Model Performance Summary
Summarizes the model's performance in the evaluated dimension using LLM-generated summaries, highlighting comparisons with other models.
![Model Performance Summary](images/model_performance_summary.png)

## Error Case Study
Presents error cases for the evaluated dimension, including input/output examples and detailed judgments.
![Error Case Study](images/error_case_study.png)

## Leaderboard
Shows the evaluation results for all models, along with visualized comparisons to previous versions (e.g., our v1.0 results).
![Leaderboard](images/leaderboard.png)
