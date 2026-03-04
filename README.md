# LLM Reliability Benchmark System

Concurrent benchmark system for evaluating LLM performance on exam questions with Kaggle-style scoring.

## Quick Start

### 1. With Ground Truth (Evaluation Mode)

Create a config file (e.g., `my_config.json`):

```json
{
  "models": ["google/gemini-3-flash-preview", "openai/gpt-4.1-mini"],
  "results_path": "../experiments/my_experiment",
  "dataset_path": "../experiments/test.csv",
  "solution_path": "../experiments/solution.csv",
  "eval_script_path": "./my_eval.py",
  "temperature": 1.0,
  "max_reasoning_tokens": 600,
  "num_repetitions": 5,
  "testing_mode": false,
  "workers": 2,
  "enable_parallel": true,
  "max_workers_per_model": 5,
  "rate_limit_seconds": 1.0,
  "message": "Test run with ground truth"
}
```

Run the benchmark:

```bash
cd devel/scripts
python batch_test_models.py --config my_config.json
```

**Worker Configuration:**
- `workers`: Number of models to test in parallel (model-level)
- `max_workers_per_model`: Number of repetitions to run in parallel per model (repetition-level)
- **Total concurrent API calls** = `workers` × `max_workers_per_model`
- **⚠️ Do NOT exceed 10 concurrent instances** to avoid rate limits
  - Example: `workers=2` × `max_workers_per_model=5` = **10 concurrent requests** ✅
  - Example: `workers=3` × `max_workers_per_model=5` = **15 concurrent requests** ❌ (too many)

**Output Structure:**
```
my_experiment/
├── model-name_1_timestamp/           # Model 1 folder
│   ├── config.json
│   ├── my_eval.py
│   ├── run_1_results.json
│   ├── run_2_results.json
│   ├── ...
│   ├── solution.csv                  # Predictions for submission
│   ├── solution_kaggle.csv            # With cost column for Kaggle
│   ├── result_analysis.md             # Accuracy, Kaggle score, cost, tokens
│   └── wrong_answers.md               # Detailed error analysis
├── model-name_2_timestamp/           # Model 2 folder
│   └── ...
└── summary.md                        # ⭐ Model comparison table
```

**summary.md Structure:**
```markdown
# Model Comparison Report

## Configuration
- Models Tested: 2
- Repetitions: 5
- Temperature: 1.0
- Max Reasoning Tokens: 600

## Results Summary
| Rank | Model | Accuracy | Kaggle Score | Total Cost ($) | Total Tokens | Avg Cost/Run ($) |
|------|-------|----------|--------------|----------------|--------------|------------------|
| 1 | google-gemini-3-flash-preview | 0.8693 ± 0.0131 | 0.8402 | $1.1487 | 434,366 | $0.230 |
| 2 | openai-gpt-4.1-mini | 0.7867 ± 0.0223 | 0.7798 | $0.2279 | 189,855 | $0.046 |
```

**Kaggle Score Formula:** `kaggle = max(0, min(1.0, accuracy - 0.15 × avg_cost_per_run))`
- Balances accuracy vs. cost (lower cost = higher score)
- Encourages cost-efficient model selection

### 2. Without Ground Truth (Submission Mode)

Create a config for competition submission:

```json
{
  "models": ["google/gemini-3-flash-preview"],
  "results_path": "../experiments/kaggle_submission",
  "dataset_path": "../datasets/test.csv",
  "solution_path": "../datasets/dummy_solution.csv",
  "eval_script_path": "./my_eval.py",
  "temperature": 1.0,
  "max_reasoning_tokens": 600,
  "num_repetitions": 1,
  "testing_mode": true,
  "workers": 1,
  "enable_parallel": false,
  "message": "Kaggle submission"
}
```

**Key differences:**
- `testing_mode: true` - Skips accuracy calculation (no ground truth needed)
- `num_repetitions: 1` - Single run for submission
- `solution_path` - Can be a dummy file (not used when testing_mode=true)

Run:

```bash
python batch_test_models.py --config kaggle_config.json
```

**Output:**
```
kaggle_submission/
└── model-name_timestamp/
    ├── solution.csv              # ⭐ Submit this to Kaggle
    └── solution_kaggle.csv       # With average cost per run column
```

`solution_kaggle.csv` format:
```csv
question_id,prediction_1,Average cost per run
1,a,0.05
2,b,0.05
3,c,0.05
...
```

This file is ready for direct submission to Kaggle competition!

## Configuration Files

Pre-configured examples in `devel/scripts/configs/`:

| Config | Models | Purpose |
|--------|--------|---------|
| `training_data_zero_shot.json` | 11 models | Full training evaluation |
| `test_scenario_1.json` | 1 model | Quick single-model test |
| `test_scenario_3.json` | 3 models | Multi-model parallel test |

Usage:
```bash
python batch_test_models.py                           # Use default config
python batch_test_models.py --config configs/test_scenario_1.json
```

## Architecture

**Key Components:**
1. **OpenRouterClient** - LLM client with usage tracking and rate limiting
2. **BenchmarkRunner** - Orchestrates parallel benchmark execution
3. **BenchmarkLogger** - Generates reports and tracks metrics

**Data Flow:**
```
JSON Config → batch_test_models.py → BenchmarkRunner
  ↓
For each model (parallel via ThreadPoolExecutor):
  ↓
  For each repetition (parallel via ThreadPoolExecutor):
    ↓
    OpenRouterClient.chat_completion() → Rate-limited API calls
    ↓
  BenchmarkLogger → JSON files + CSV + Markdown reports
```

**Rate Limiting:**
- Global `RateLimiter` shared across all workers
- Respects `rate_limit_seconds` delay between API calls
- Prevents API overload even with high parallelism

## Output Files

**Per-Run Files:**
- `run_N_results.json` - Raw LLM responses and metadata

**Per-Model Files:**
- `solution.csv` - Predictions (prediction_1 to prediction_N columns)
- `solution_kaggle.csv` - With 'Average cost per run' column for Kaggle
- `result_analysis.md` - Detailed accuracy, Kaggle score, cost analysis
- `wrong_answers.md` - Error analysis grouped by run

**Aggregated Files:**
- `summary.md` - Model comparison table (ranked by Kaggle score)

## Requirements

- Python 3.8+
- pandas, tqdm
- OpenRouter API key: `export OPENROUTER_API_KEY=your_key`

## License

Part of LLM Reliability Engineering project.
