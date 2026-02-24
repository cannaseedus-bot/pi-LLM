# π-LLM Benchmark Protocol v1.0.0 (PB-π-1)

## I. Purpose

This protocol defines a **neutral, reproducible framework** for comparing:

- **π-LLM** (periodic + spectral + geodesic)
- **Standard GPT-style Transformers** (Euclidean positional, dense attention)

It is designed to measure:

1. Language modeling quality
2. Spectral efficiency
3. Periodic reasoning performance
4. Generalization under smoothness
5. Stability and determinism
6. Compute efficiency
7. Quantization robustness

---

## II. Controlled Comparison Rules

To avoid benchmark bias, all comparisons must use:

1. Same parameter count (±2%)
2. Same tokenization
3. Same training data
4. Same optimizer
5. Same precision (fp16/fp32)
6. Same training steps
7. Same batch size

No architecture may receive extra hidden capacity.

---

## III. Core Benchmark Categories

### A. Standard Language Modeling

| Metric | Meaning |
| --- | --- |
| Perplexity | Cross-entropy |
| Bits-per-token | Information efficiency |
| MMLU / reasoning tasks | General reasoning |
| Code benchmarks | Structured logic |
| GSM8K | Math |

π-LLM must compete directly here.

### B. Spectral Efficiency Benchmarks

These tasks explicitly probe periodic smoothness.

#### 1) Fourier Completion Task

Given a partial periodic function, the model predicts the missing region.

- Metric: \(L_2\) error.
- Expected outcome: π-LLM may benefit from spectral inductive bias.

#### 2) Rotational Invariance Task

Evaluate output consistency under rotated input sequences.

- Metric: \(\Delta = | f(x) - f(\text{rotate}(x)) |\).
- Expected outcome: π-LLM should show lower \(\Delta\).

#### 3) Periodic Pattern Continuation

Examples:

- Sine-like sequences
- Cyclic code
- Rotating symbol patterns

Measure continuation accuracy.

### C. Long Context Scaling

Track:

| Metric | Meaning |
| --- | --- |
| Loss vs context length | Accuracy scaling |
| Throughput vs N | Runtime scaling |
| Memory footprint | Space cost |
| Attention FLOPs | Compute profile |

FFT-based π-attention is expected to show distinct scaling behavior.

### D. Stability Benchmarks

#### 1) Gradient Stability

Track:

- Max gradient norm
- Spectral energy drift
- Training divergence frequency

#### 2) Spectral Energy Concentration

Measure entropy:

\[
H_\pi = -\sum p_k \log p_k
\]

Lower entropy indicates smoother representation.

### E. Quantization Robustness

Compare INT4 spectral quantization against dense INT4.

| Metric | Compare |
| --- | --- |
| Perplexity drop | Relative degradation |
| Spectral error | Frequency-domain distortion |
| Memory savings | Compression benefit |
| Inference speed | Latency/throughput |

Hypothesis: π-LLM spectral quantization degrades more gracefully.

### F. Determinism & Reproducibility

Test:

1. Re-run inference across machines
2. Compare bitwise equality
3. Compare reduction-order sensitivity

Claimed deterministic reductions must be empirically verified.

### G. Distributed Scaling

Measure:

| Nodes | Speedup | Communication overhead |
| --- | --- | --- |

Test modes:

- Spectral shard scaling
- Expert shard scaling
- Head shard scaling

---

## IV. Efficiency Metrics

Track:

1. FLOPs per token
2. Spectral FLOPs
3. FFT overhead
4. MoE routing cost
5. Energy per token (when hardware counters are available)

---

## V. Statistical Evaluation

For every task:

- Run 3 independent seeds
- Report mean ± std
- Use paired t-test for deltas
- Report effect size

No cherry-picking.

---

## VI. Reporting Format

Every benchmark record should include:

```json
{
  "model_type": "π-LLM | GPT",
  "params": 125000000,
  "layers": 12,
  "heads": 12,
  "bandlimit": 128,
  "experts": 4,
  "training_tokens": 20000000000,
  "perplexity": 0.0,
  "task_scores": {},
  "spectral_entropy": 0.0,
  "quantization_drop": 0.0,
  "context_scaling_curve": [],
  "determinism_hash": ""
}
```

---

## VII. Expected Hypotheses

These are testable hypotheses, not guarantees:

| Area | Hypothesis |
| --- | --- |
| Periodic tasks | π-LLM better |
| Smooth signals | π-LLM better |
| Natural language | Comparable |
| Large-scale pretraining | TBD |
| INT4 robustness | π-LLM more stable |
| Determinism | π-LLM stronger |
| Pure reasoning | Unknown |

---

## VIII. Fairness Safeguards

- No one-sided regularizers
- Same dropout policy
- Same tokenizer
- Same vocabulary size
- Same training steps
- No hyperparameter cheating

---

## IX. Minimal Experimental Setup

| Setting | Value |
| --- | --- |
| Params | 125M |
| Layers | 12 |
| Dim | 768 |
| Heads | 12 |
| Experts (π) | 4 |
| Bandlimit | 128 |
| Context | 2048 |
| Tokens | 20B |

---

## X. What This Protocol Tests

This protocol directly tests whether:

1. Spectral bias improves generalization
2. Geodesic routing improves locality
3. Curvature adaptation improves stability
4. Spectral quantization is superior
5. Deterministic execution is practical

It does **not** assume π-LLM superiority.

---

## XI. Extension Layer (Advanced)

Future optional extensions:

- Adversarial periodic attacks
- Spectral compression curves
- π-PAC bound empirical validation
- Scaling law comparison
- Training efficiency curves (loss vs compute)

---

## Final State

PB-π-1 gives π-LLM a measurable, neutral benchmark standard with:

- Reproducible comparisons
- Determinism validation
- Quantization evaluation
- Distributed scaling evaluation

This turns claims into empirical tests.
