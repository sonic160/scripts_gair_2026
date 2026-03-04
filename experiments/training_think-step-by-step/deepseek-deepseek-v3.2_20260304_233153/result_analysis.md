# Solution Analysis Report
Generated: 2026-03-05 00:20:14

## Configuration

### Model Parameters
- **Model**: deepseek/deepseek-v3.2
- **Temperature**: 1.0
- **Max Tokens**: None
- **Max Reasoning Tokens**: 0

### Dataset
- **Test Dataset**: C:\Users\zzg_s\OneDrive - CentraleSupelec\Code\Python\scripts_gair_2026\experiments\train.csv
- **Solution File**: C:\Users\zzg_s\OneDrive - CentraleSupelec\Code\Python\scripts_gair_2026\experiments\train_solution.csv

### Execution
- **Number of Runs**: 5
- **Start Time**: 2026-03-04 23:31:53
- **End Time**: 2026-03-05 00:20:14
- **Duration**: 0:48:20.704222

### Message
Training dataset. We force the model not the reason, but 'think lound' using a chain-of-thoughts.

## Results Summary

- **Mean Accuracy**: 0.7918 ± 0.0153
- **Mean Kaggle Score**: 0.7876
- **Total Cost**: $0.1409
- **Total Tokens**: 315,421
  - Prompt Tokens: 51,455
  - Completion Tokens: 263,966
- **Total API Requests**: 245

## Performance per Run

| Run | Accuracy | Kaggle Score | Correct / Total | Cost ($) | Tokens (Prompt + Completion) |
|-----|----------|--------------|-----------------|----------|-----------------------------|
| 1 | 0.7959 | 0.7910 | 39 / 49 | $0.0330 | 59,130 (10,291 + 48,839) |
| 2 | 0.8163 | 0.8127 | 40 / 49 | $0.0242 | 63,787 (10,291 + 53,496) |
| 3 | 0.7959 | 0.7915 | 39 / 49 | $0.0298 | 70,034 (10,291 + 59,743) |
| 4 | 0.7755 | 0.7711 | 38 / 49 | $0.0296 | 63,306 (10,291 + 53,015) |
| 5 | 0.7755 | 0.7718 | 38 / 49 | $0.0245 | 59,164 (10,291 + 48,873) |

## Cost Efficiency Analysis

### Overall Metrics

- **Average cost per 1% Accuracy (dollar cents)**: $0.035599
- **Average tokens per 1% Accuracy**: 796.7
- **Average Cost per Question**: $0.000575
- **Average Tokens per Question**: 1287.4

### Average Per Run

- **Average Cost per Run**: $0.028188
- **Average Tokens per Run**: 63084.2
  - Prompt Tokens: 10291.0
  - Completion Tokens: 52793.2
