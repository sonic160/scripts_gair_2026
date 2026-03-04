# AGENTS.md - Agent Guide for LLM Reliability Benchmark System

This file provides guidelines for agentic coding assistants working in this repository.

## Build / Lint / Test Commands

### Running Benchmarks
```bash
# Main entry point - run full benchmark with config
cd scripts
python batch_test_models.py --config configs/my_config.json

# Use default config (edit CONFIG_PATH in batch_test_models.py)
python batch_test_models.py
```

### Testing Individual Components
```bash
# Test evaluation script without running full benchmark
cd scripts
python my_eval.py  # Requires OPENROUTER_API_KEY environment variable

# Validate eval script format
python batch_test_models.py --config configs/test.json  # Validates before running
```

### Running Specific Tests
There is no formal test suite. Instead:
- Test evaluation scripts directly: `python my_eval.py` (has test mode in `__main__`)
- Run with small dataset: Set `testing_mode: true` in config JSON
- Use `num_repetitions: 1` for quick single-run tests

### Environment Setup
```bash
# Set API key (required)
export OPENROUTER_API_KEY=your_key_here

# Optional: Set rate limit (default: 1.0 seconds)
export OPENROUTER_RATE_LIMIT_SECONDS=1.0
```

## Code Style Guidelines

### Python Version
- Python 3.8+ (codebase tested with 3.14.2)

### Imports
- Standard library imports first, then third-party, then local
- Use `from typing import ...` for type hints
- Local imports: `from supporting_scripts import ...`
- Add parent directory for imports when needed: `sys.path.insert(0, str(Path(__file__).parent.parent))`

### Formatting
- Use 4 spaces for indentation
- Line length: Aim for 80-100 characters, flexible up to 120
- Use f-strings for string formatting: `f"Model: {model}, Cost: ${cost:.4f}"`
- Use raw strings for regex patterns: `r'\{[^{}]*\}'`

### Type Hints
- Always use type hints for function parameters and returns
- Use `-> dict`, `-> str`, `-> bool`, etc.
- For complex types: `from typing import List, Dict, Optional, Any, Tuple`
- Use `Dict[str, Any]` for flexible dictionaries, not bare `dict`

### Naming Conventions
- **Classes**: `CamelCase` (e.g., `OpenRouterClient`, `BenchmarkRunner`)
- **Functions/Variables**: `snake_case` (e.g., `chat_completion`, `run_benchmark`)
- **Constants**: `UPPER_SNAKE_CASE` (e.g., `CONFIG_PATH`, `MAX_TOKENS`)
- **Private methods**: Leading underscore (e.g., `_get_calculate_kaggle_score()`)

### Docstrings
- Use Google-style docstrings (triple quotes)
- Include Args, Returns, Raises, Examples sections
- Keep docstrings concise but informative
- Example:
```python
def evaluate_question(question: str, client, model: str, config: dict) -> dict:
    """
    Evaluate a single question with LLM.

    Args:
        question: Question text
        client: OpenRouterClient instance
        model: Model name
        config: Configuration dictionary

    Returns:
        Dictionary with raw_response, extracted_answer, usage keys
    """
```

### Error Handling
- Use specific exceptions: `ValueError`, `FileNotFoundError`, `KeyError`
- Include descriptive error messages
- Use try/except for API calls and file I/O
- Log errors appropriately: `print(f"[FAILED] {model}: {error}")`
- Return error information in result dictionaries for parallel execution

### File Paths
- Use `pathlib.Path` for all file operations, not `os.path`
- Resolve relative paths to absolute: `path.resolve()`
- Check file existence: `if not path.exists():`
- Use `Path(__file__).parent` for script-relative paths

### Threading and Parallel Execution
- Use `ThreadPoolExecutor` for parallel benchmarks
- Use `threading.Lock()` for thread-safe operations
- Global rate limiting via singleton pattern
- Always use `tqdm` progress bars for user feedback
- Track position for nested progress bars: `position=run_idx, leave=False`

### Configuration Management
- Use JSON files for configuration (see `scripts/configs/`)
- Validate config structure before execution
- Transform relative paths to absolute paths early
- Use `config.get('key', default_value)` for optional keys

### API Client Usage
- Always use `OpenRouterClient` for API calls
- Track usage with `get_usage_summary()` before/after calls
- Calculate per-question usage as difference
- Handle rate limiting automatically via `GlobalRateLimiter`

### Evaluation Scripts (my_eval.py)
- Must implement `evaluate_question()` function
- Return dict with: `raw_response`, `extracted_answer`, `usage`
- Optional: `metadata` field for additional info
- Include `if __name__ == '__main__':` test mode
- Use JSON parsing with regex fallback for answer extraction

### Output Files
- Save results as JSON: `json.dump(data, f, indent=2)`
- Use UTF-8 encoding: `open(path, 'w', encoding='utf-8')`
- Generate CSV with pandas: `df.to_csv(path, index=False)`
- Create markdown reports with tables

### Constants and Magic Numbers
- Define constants at module level or in config
- Avoid magic numbers in code
- Use descriptive names: `rate_limit_seconds` not `rl`

## Project Architecture

### Key Components
1. **OpenRouterClient** (`supporting_scripts/openrouter_client.py`)
   - API client with cumulative usage tracking
   - Global rate limiting via singleton pattern

2. **BenchmarkRunner** (`supporting_scripts/benchmark_runner.py`)
   - Orchestrates parallel benchmark execution
   - Loads user evaluation scripts dynamically

3. **BenchmarkLogger** (`supporting_scripts/benchmark_logger.py`)
   - Generates reports and tracks metrics
   - Saves results as JSON, CSV, Markdown

4. **batch_test_models.py** (`scripts/batch_test_models.py`)
   - Main entry point
   - Handles config loading and parallel model execution

5. **my_eval.py** (`scripts/my_eval.py`)
   - Template for user evaluation scripts
   - Answer extraction from LLM responses

### Data Flow
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

### Important Patterns
- **Rate Limiting**: Global singleton ensures all workers respect rate limits
- **Parallel Execution**: Two-level parallelism (models × repetitions)
- **Usage Tracking**: Per-question usage calculated as difference
- **Config Validation**: Validate scripts and datasets before execution
- **Error Recovery**: Continue on failure, report errors in results

## Common Patterns

### Dynamic Module Loading
```python
import importlib.util
spec = importlib.util.spec_from_file_location("module_name", script_path)
module = importlib.util.module_from_spec(spec)
spec.loader.exec_module(module)
```

### Rate-Limited API Calls
```python
# Client automatically uses GlobalRateLimiter
client = OpenRouterClient()  # No need to pass rate limiter
response = client.chat_completion(model=model, messages=messages)
```

### Sanitizing Model Names
```python
def sanitize_model_name(model: str) -> str:
    return model.replace('/', '-').replace('.', '.')
```

### Comparing Answers
```python
# Normalize answers for comparison (comma-separated, lowercase)
pred_norm = str(pred).strip().lower().replace(", ", ",")
correct_norm = str(correct).strip().lower().replace(", ", ",")
pred_set = set(p.strip() for p in pred_norm.split(',') if p.strip())
correct_set = set(c.strip() for c in correct_norm.split(',') if c.strip())
if pred_set == correct_set:
    correct_count += 1
```

## Key Constraints
- **Never exceed 10 concurrent API calls** (rate limit protection)
- Worker config: `workers` × `max_workers_per_model` ≤ 10
- Always validate evaluation scripts before running benchmarks
- Use testing_mode for submissions without ground truth
- Kaggle score formula: `max(0, min(1.0, accuracy - 0.15 × avg_cost_per_run))`
