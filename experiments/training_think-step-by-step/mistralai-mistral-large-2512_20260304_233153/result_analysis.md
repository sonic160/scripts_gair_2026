# Solution Analysis Report
Generated: 2026-03-04 23:44:14

## Configuration

### Model Parameters
- **Model**: mistralai/mistral-large-2512
- **Temperature**: 1.0
- **Max Tokens**: None
- **Max Reasoning Tokens**: 0

### Dataset
- **Test Dataset**: C:\Users\zzg_s\OneDrive - CentraleSupelec\Code\Python\scripts_gair_2026\experiments\train.csv
- **Solution File**: C:\Users\zzg_s\OneDrive - CentraleSupelec\Code\Python\scripts_gair_2026\experiments\train_solution.csv

### Execution
- **Number of Runs**: 5
- **Start Time**: 2026-03-04 23:31:53
- **End Time**: 2026-03-04 23:44:14
- **Duration**: 0:12:21.524818

### Message
Training dataset. We force the model not the reason, but 'think lound' using a chain-of-thoughts.

## Results Summary

- **Mean Accuracy**: 0.7429 ± 0.0208
- **Mean Kaggle Score**: 0.7332
- **Total Cost**: $0.3214
- **Total Tokens**: 250,626
  - Prompt Tokens: 54,510
  - Completion Tokens: 196,116
- **Total API Requests**: 245

## Performance per Run

| Run | Accuracy | Kaggle Score | Correct / Total | Cost ($) | Tokens (Prompt + Completion) |
|-----|----------|--------------|-----------------|----------|-----------------------------|
| 1 | 0.7347 | 0.7256 | 36 / 49 | $0.0609 | 47,900 (10,902 + 36,998) |
| 2 | 0.7143 | 0.7041 | 35 / 49 | $0.0679 | 52,544 (10,902 + 41,642) |
| 3 | 0.7755 | 0.7667 | 38 / 49 | $0.0584 | 46,223 (10,902 + 35,321) |
| 4 | 0.7551 | 0.7448 | 37 / 49 | $0.0689 | 53,209 (10,902 + 42,307) |
| 5 | 0.7347 | 0.7249 | 36 / 49 | $0.0652 | 50,750 (10,902 + 39,848) |

## Cost Efficiency Analysis

### Overall Metrics

- **Average cost per 1% Accuracy (dollar cents)**: $0.086539
- **Average tokens per 1% Accuracy**: 674.8
- **Average Cost per Question**: $0.001312
- **Average Tokens per Question**: 1023.0

### Average Per Run

- **Average Cost per Run**: $0.064286
- **Average Tokens per Run**: 50125.2
  - Prompt Tokens: 10902.0
  - Completion Tokens: 39223.2
