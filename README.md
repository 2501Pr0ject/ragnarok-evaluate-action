# RAGnarok Evaluate Action

Evaluate your RAG pipeline quality in CI/CD with [RAGnarok-AI](https://github.com/2501Pr0ject/RAGnarok-AI).

**Advisory by default** — This action provides feedback, not blocking verdicts. LLM-as-Judge scores are suggestions, not absolute truth.

## Usage

```yaml
name: RAG Quality Check
on: [pull_request]

jobs:
  evaluate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: 2501Pr0ject/ragnarok-evaluate-action@v1
        with:
          config: ragnarok.yaml
```

## Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `config` | Path to `ragnarok.yaml` config file | - |
| `testset` | Path to testset JSON file (alternative to config) | - |
| `threshold` | Quality threshold (0.0-1.0) | `0.7` |
| `fail-on-threshold` | Fail workflow if below threshold | `false` |
| `comment-on-pr` | Post results as PR comment | `true` |
| `demo` | Run demo evaluation (for testing) | `false` |
| `python-version` | Python version | `3.10` |
| `ragnarok-version` | RAGnarok-AI version (`latest` or specific) | `latest` |
| `dataset-baseline` | Path to baseline testset for diff comparison | - |
| `dataset-current` | Path to current testset (defaults to `testset`) | - |
| `fail-on-dataset-change` | Fail if dataset differs from baseline | `true` |

## Outputs

| Output | Description |
|--------|-------------|
| `score` | Overall evaluation score (0.0-1.0) |
| `status` | `pass` or `review` |
| `report` | Full evaluation report (JSON) |
| `dataset-changed` | Whether dataset changed from baseline (`true`/`false`) |
| `dataset-diff-summary` | Summary of changes (`+2 -1 ~3`) |

## PR Comment

The action posts a comment distinguishing deterministic vs advisory metrics:

```markdown
## RAGnarok Evaluation Report

**Retrieval Metrics** *(deterministic)*
| Metric | Score | Status |
|--------|-------|--------|
| Precision | 0.82 | OK |
| Recall | 0.75 | OK |

**Generation Quality** *(LLM-as-Judge, advisory)*
| Metric | Score | Status |
|--------|-------|--------|
| Faithfulness | 0.68 | Review |
| Relevance | 0.72 | Review |

RAGnarok suggests reviewing the scores below threshold (0.7).

---
*Note: LLM-as-Judge scores are advisory, not definitive. Review recommended.*
```

## Examples

### Basic (advisory feedback)

```yaml
- uses: 2501Pr0ject/ragnarok-evaluate-action@v1
  with:
    config: ragnarok.yaml
```

### Strict mode (fail on threshold)

```yaml
- uses: 2501Pr0ject/ragnarok-evaluate-action@v1
  with:
    config: ragnarok.yaml
    threshold: 0.8
    fail-on-threshold: true
```

### With testset directly

```yaml
- uses: 2501Pr0ject/ragnarok-evaluate-action@v1
  with:
    testset: tests/golden.json
    threshold: 0.75
```

### Dataset versioning (v1.1.0+)

Detect when your testset has changed from a baseline:

```yaml
- uses: 2501Pr0ject/ragnarok-evaluate-action@v1
  with:
    testset: tests/golden.json
    dataset-baseline: tests/golden.baseline.json
    fail-on-dataset-change: true  # Fail if dataset changed
```

This is useful for:
- Detecting unintended testset modifications
- Enforcing review when golden sets change
- Tracking dataset evolution over time

### Demo (test the action)

```yaml
- uses: 2501Pr0ject/ragnarok-evaluate-action@v1
  with:
    demo: true
```

### Pin to specific RAGnarok version

```yaml
- uses: 2501Pr0ject/ragnarok-evaluate-action@v1
  with:
    config: ragnarok.yaml
    ragnarok-version: '1.4.1'
```

## Config File

Create `ragnarok.yaml` in your repo:

```yaml
testset: tests/golden.json
output: results.json
fail_under: 0.7
metrics:
  - precision
  - recall
  - mrr
criteria:
  - faithfulness
  - relevance
```

## Philosophy

This action follows RAGnarok-AI's **humble evaluation** approach:

- **Retrieval metrics** (Precision, Recall, MRR, NDCG) are deterministic and reliable
- **LLM-as-Judge metrics** (Faithfulness, Relevance, Hallucination) are advisory
- Default behavior is **warning, not blocking** — you decide when to enforce
- PR comments clearly distinguish what's definitive vs what needs human review

## Compatibility

| Action Version | RAGnarok-AI Version | Features |
|----------------|---------------------|----------|
| v1.1.0+ | 1.4.1+ | Dataset diff, all v1.0 features |
| v1.0.0 | 1.4.0+ | Evaluate, PR comments, threshold |

## License

AGPL-3.0 — See [RAGnarok-AI](https://github.com/2501Pr0ject/RAGnarok-AI)
