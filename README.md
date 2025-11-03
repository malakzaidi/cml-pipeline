# CML - Continuous Machine Learning

## Overview

CML (Continuous Machine Learning) is an open-source CI/CD tool designed specifically for machine learning projects. Created by Iterative.ai, CML brings DevOps practices to ML/AI workflows, enabling data scientists to automatically train and evaluate models in production-like environments.

## What is CML?

CML helps you automate ML workflows by:
- Running ML experiments in CI/CD pipelines (GitHub Actions, GitLab CI, Bitbucket Pipelines)
- Generating visual reports with metrics, plots, and tables
- Comparing model performance across commits and branches
- Provisioning cloud compute resources (AWS, Azure, GCP) on-demand
- Enabling reproducible ML experiments

## Architecture & Workflow

```
┌─────────────────────────────────────────────────────────────┐
│                     Developer Workflow                       │
└─────────────────────────────────────────────────────────────┘
                              ↓
         ┌────────────────────────────────────┐
         │   1. Push Code/Data Changes        │
         │      to Git Repository             │
         └────────────────────────────────────┘
                              ↓
         ┌────────────────────────────────────┐
         │   2. CI/CD Pipeline Triggered      │
         │      (GitHub Actions, GitLab CI)   │
         └────────────────────────────────────┘
                              ↓
         ┌────────────────────────────────────┐
         │   3. CML Provisions Resources      │
         │      (Optional: Cloud GPU/CPU)     │
         └────────────────────────────────────┘
                              ↓
         ┌────────────────────────────────────┐
         │   4. Run ML Pipeline               │
         │      - Data preprocessing          │
         │      - Model training              │
         │      - Model evaluation            │
         └────────────────────────────────────┘
                              ↓
         ┌────────────────────────────────────┐
         │   5. CML Generates Report          │
         │      - Metrics (accuracy, loss)    │
         │      - Plots (confusion matrix)    │
         │      - Model comparisons           │
         └────────────────────────────────────┘
                              ↓
         ┌────────────────────────────────────┐
         │   6. Report Posted as Comment      │
         │      on Pull Request/Commit        │
         └────────────────────────────────────┘
                              ↓
         ┌────────────────────────────────────┐
         │   7. Team Reviews & Merges         │
         │      Based on ML Metrics           │
         └────────────────────────────────────┘
```

## Installation

### Prerequisites
- Git repository (GitHub, GitLab, or Bitbucket)
- CI/CD platform account
- Node.js 16+ (for npm installation)

### Install CML

**Via npm:**
```bash
npm install -g @dvcorg/cml
```

**Via pip:**
```bash
pip install cml
```

**In CI/CD Pipeline (recommended):**
CML is typically installed directly in your CI/CD configuration file.

## Quick Start Guide

### Step 1: Set Up Your ML Project Structure

```
my-ml-project/
├── .github/
│   └── workflows/
│       └── cml.yaml          # CI/CD configuration
├── data/
│   └── dataset.csv
├── src/
│   ├── train.py              # Training script
│   └── evaluate.py           # Evaluation script
├── models/
│   └── model.pkl
├── plots/
│   └── metrics.png
└── requirements.txt
```

### Step 2: Create a Training Script

**train.py:**
```python
import pandas as pd
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score
import json
import matplotlib.pyplot as plt

# Load data
df = pd.read_csv('data/dataset.csv')
X_train, X_test, y_train, y_test = train_test_split(...)

# Train model
model = RandomForestClassifier()
model.fit(X_train, y_train)

# Evaluate
predictions = model.predict(X_test)
accuracy = accuracy_score(y_test, predictions)

# Save metrics
with open('metrics.json', 'w') as f:
    json.dump({'accuracy': accuracy}, f)

# Generate plot
plt.plot(...)
plt.savefig('plots/confusion_matrix.png')
```

### Step 3: Configure CI/CD Pipeline

**GitHub Actions (.github/workflows/cml.yaml):**
```yaml
name: CML Training Pipeline

on: [push, pull_request]

jobs:
  train-and-report:
    runs-on: ubuntu-latest
    steps:
      # Checkout repository
      - uses: actions/checkout@v3
      
      # Setup Python environment
      - uses: actions/setup-python@v4
        with:
          python-version: '3.9'
      
      # Install dependencies
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install cml
      
      # Run training pipeline
      - name: Train model
        run: python src/train.py
      
      # Generate CML report
      - name: Create CML Report
        env:
          REPO_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          # Write report header
          echo "# ML Training Report" >> report.md
          echo "## Metrics" >> report.md
          
          # Add metrics from JSON
          cat metrics.json | cml comment create --publish report.md
          
          # Add plots
          echo "## Confusion Matrix" >> report.md
          cml publish plots/confusion_matrix.png --md >> report.md
          
          # Post report as PR comment
          cml comment update report.md
```

### Step 4: Understanding the Pipeline Flow

```
┌──────────────────────────────────────────────────────────┐
│ Stage 1: Environment Setup                               │
│ • Checkout code                                          │
│ • Install Python/dependencies                            │
│ • Install CML                                            │
└──────────────────────────────────────────────────────────┘
                         ↓
┌──────────────────────────────────────────────────────────┐
│ Stage 2: ML Pipeline Execution                           │
│ • Load and preprocess data                               │
│ • Train model                                            │
│ • Generate metrics and visualizations                    │
└──────────────────────────────────────────────────────────┘
                         ↓
┌──────────────────────────────────────────────────────────┐
│ Stage 3: Report Generation                               │
│ • Collect metrics from JSON files                        │
│ • Publish plots/images                                   │
│ • Format markdown report                                 │
└──────────────────────────────────────────────────────────┘
                         ↓
┌──────────────────────────────────────────────────────────┐
│ Stage 4: Report Publishing                               │
│ • Post as PR comment or commit comment                   │
│ • Make results accessible to team                        │
└──────────────────────────────────────────────────────────┘
```

## Key CML Commands

### 1. `cml comment create`
Posts a comment with your report content.

```bash
cml comment create report.md
```

### 2. `cml publish`
Uploads and publishes assets (images, plots) and returns a markdown link.

```bash
cml publish plots/accuracy.png --md >> report.md
```

### 3. `cml runner`
Provisions cloud compute resources (GPU/CPU instances) for training.

```bash
cml runner \
  --cloud aws \
  --cloud-region us-west-2 \
  --cloud-type=m5.large \
  --labels=cml-runner
```

### 4. `cml pr`
Creates or updates a pull request.

```bash
cml pr --md >> report.md
```

## Advanced Example: Model Comparison

```yaml
name: Model Comparison Pipeline

on: [pull_request]

jobs:
  compare-models:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0  # Fetch all history for comparison
      
      - uses: actions/setup-python@v4
        with:
          python-version: '3.9'
      
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install cml
      
      # Train model on current branch
      - name: Train current model
        run: |
          python src/train.py
          mv metrics.json metrics_current.json
          mv plots/metrics.png plots/metrics_current.png
      
      # Checkout main branch and train
      - name: Train baseline model
        run: |
          git checkout main
          python src/train.py
          mv metrics.json metrics_baseline.json
          mv plots/metrics.png plots/metrics_baseline.png
          git checkout -
      
      # Generate comparison report
      - name: Compare models
        env:
          REPO_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          echo "# Model Comparison Report" >> report.md
          echo "" >> report.md
          
          echo "## Current Branch Metrics" >> report.md
          cat metrics_current.json >> report.md
          echo "" >> report.md
          cml publish plots/metrics_current.png --md >> report.md
          
          echo "## Baseline (main) Metrics" >> report.md
          cat metrics_baseline.json >> report.md
          echo "" >> report.md
          cml publish plots/metrics_baseline.png --md >> report.md
          
          cml comment create report.md
```

## CML with Cloud Resources

### Provisioning GPU Instances

```yaml
name: GPU Training Pipeline

on: [push]

jobs:
  deploy-runner:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: iterative/setup-cml@v1
      
      - name: Deploy runner on AWS
        env:
          REPO_TOKEN: ${{ secrets.PERSONAL_ACCESS_TOKEN }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        run: |
          cml runner \
            --cloud=aws \
            --cloud-region=us-west-2 \
            --cloud-type=g4dn.xlarge \
            --cloud-gpu=tesla \
            --labels=cml-gpu

  train-model:
    needs: deploy-runner
    runs-on: [self-hosted, cml-gpu]
    steps:
      - uses: actions/checkout@v3
      - name: Train on GPU
        run: |
          python src/train_gpu.py
          
      - name: Generate report
        env:
          REPO_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          cml comment create report.md
```

## Benefits of Using CML

1. **Reproducibility**: Every experiment is tracked and versioned with Git
2. **Collaboration**: Team members can review model changes like code reviews
3. **Automation**: No manual metric tracking or report generation
4. **Cost Efficiency**: Spin up cloud resources only when needed
5. **Visibility**: Metrics and plots visible directly in PRs
6. **Integration**: Works with existing ML tools (DVC, MLflow, TensorFlow, PyTorch)

## Best Practices

1. **Version your data**: Use DVC (Data Version Control) alongside CML
2. **Keep reports concise**: Focus on key metrics and visualizations
3. **Use descriptive commit messages**: Help team understand experiment goals
4. **Set up notifications**: Configure alerts for failed training runs
5. **Optimize cloud usage**: Terminate runners after job completion
6. **Cache dependencies**: Speed up pipeline execution

## Integration with Other Tools

```
CML Ecosystem:
┌──────────────────────────────────────────────────────┐
│                       Git/GitHub                      │
│              (Version Control & Hosting)              │
└──────────────────────────────────────────────────────┘
                         ↕
┌──────────────────────────────────────────────────────┐
│                        CML                            │
│          (CI/CD Orchestration & Reporting)            │
└──────────────────────────────────────────────────────┘
         ↕                ↕                ↕
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│     DVC      │  │   MLflow     │  │  TensorBoard │
│ (Data/Model  │  │  (Experiment │  │  (Training   │
│  Versioning) │  │   Tracking)  │  │   Metrics)   │
└──────────────┘  └──────────────┘  └──────────────┘
```

## Troubleshooting

### Issue: Report not posting
**Solution**: Ensure `GITHUB_TOKEN` or `REPO_TOKEN` has proper permissions.

### Issue: Cloud runner not starting
**Solution**: Verify cloud credentials and check instance availability in region.

### Issue: Metrics file not found
**Solution**: Ensure training script saves outputs before CML report generation.

## Resources

- **Official Documentation**: https://cml.dev
- **GitHub Repository**: https://github.com/iterative/cml
- **Community Forum**: https://discuss.dvc.org
- **Example Projects**: https://github.com/iterative/cml-examples

## License

CML is open-source software licensed under Apache License 2.0.

---

**Get Started**: Add CML to your ML project today and bring CI/CD best practices to your machine learning workflow!
