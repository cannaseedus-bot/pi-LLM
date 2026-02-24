# π-LLM Formal Architecture Specification

**Identity:** `π-LLM-1.0.0`

A periodic-manifold transformer with spectral control, geodesic attention, and deterministic execution.

## I. High-Level Architecture

\[
\pi\text{-LLM} = \text{Embed}_\pi \rightarrow [\pi\text{-Block}]^L \rightarrow \text{Spectral Decode}
\]

Each π-Block contains:

1. π-Geodesic Attention
2. π-Manifold MoE
3. Curvature-Adaptive Spectral FFN
4. Deterministic Residual + Normalize

## II. Core Design Principles

1. **Periodic token geometry**: tokens are mapped to \(S^1\), not linear \(\mathbb{Z}\)-indexed positions.
2. **Spectral bandlimit control**: every layer declares and enforces maximum frequency \(K\).
3. **Geodesic attention**: attention includes circular distance.
4. **Deterministic reduction**: fixed reduction ordering and canonical arithmetic behavior.
5. **Optional ZK verifiability**: inference/training traces can be constrained for proof systems.

## III. Input Representation

### 1. Token + Periodic Positional Embedding

Let sequence length be \(N\). Define angular coordinate:

\[
\theta_i = \frac{2\pi i}{N}
\]

Token representation:

\[
E_i = W_{tok}(x_i) + PE_\pi(\theta_i)
\]

Fourier positional encoding:

\[
PE_\pi(\theta) = [\sin(k\theta), \cos(k\theta)]_{k=1}^{K_{pos}}
\]

## IV. π-Block Specification

### 1. π-Geodesic Multi-Head Attention

Query/key/value projections:

\[
Q = XW_Q,\quad K = XW_K,\quad V = XW_V
\]

Circular (geodesic) distance:

\[
d_{ij} = \min\left(|\theta_i-\theta_j|,\; 2\pi - |\theta_i-\theta_j|\right)
\]

Attention logits with geodesic penalty:

\[
S_{ij} = \frac{Q_i \cdot K_j}{\sqrt{d_h}} - \lambda d_{ij}
\]

Deterministic softmax:

\[
A_{ij} = \frac{\exp(S_{ij})}{\sum_k \exp(S_{ik})}
\]

Determinism contract requires fixed polynomial approximation behavior and fixed reduction tree.

### 2. π-Manifold MoE

With \(E\) experts, routing weights:

\[
G_k(\theta_i) =
\frac{\exp\left(g_k(\theta_i) - \lambda d(\theta_i, \mu_k)\right)}{\sum_j \exp\left(g_j(\theta_i) - \lambda d(\theta_i, \mu_j)\right)}
\]

MoE output:

\[
Y(\theta_i) = \sum_k G_k(\theta_i)\,E_k(X(\theta_i))
\]

Supported expert families include spectral, periodic convolutional, and local-attention experts.

### 3. Curvature-Adaptive Spectral FFN

Local discrete curvature estimate:

\[
R_i = X_{i+1} - 2X_i + X_{i-1}
\]

Adaptive per-position bandlimit:

\[
K_i = K_0 + \gamma\lVert R_i \rVert
\]

Global cap must hold:

\[
K_i \le K_{max}
\]

Spectral expansion is truncated at \(K_i\) (bounded by \(K_{max}\)).

### 4. Residual Update

\[
X_{\ell+1} = X_\ell + \text{MoE}(\text{Attention}(X_\ell))
\]

All operations are shape-safe and precision-contract-checked.

## V. Spectral Constraints

Every layer declares:

```yaml
bandlimit: K
max_resolution: N
precision: fp16 | fp32 | field
```

Compiler/runtime invariant:

\[
K \le \frac{N}{2}
\]

## VI. Formal Type Signature

```text
πLLM<
  Layers = L,
  Resolution = N,
  ModelDim = D,
  Heads = H,
  Experts = E,
  Bandlimit = K
>
```

Static constraints:

- \(N\) fixed per compiled model variant
- \(K \le N/2\)
- \(H\) divides \(D\)
- \(E \ge 1\)

## VII. Training Objective

Base autoregressive loss:

\[
\mathcal{L}_{AR} = -\sum_i \log P(x_i\mid x_{<i})
\]

Optional spectral entropy regularization:

\[
\mathcal{L}_{total} = \mathcal{L}_{AR} + \lambda H_\pi
\]

## VIII. Inference Mode

Deterministic inference profile:

- Fixed polynomial softmax implementation
- Fixed reduction tree
- Optional top-k truncation
- No stochastic sampling unless explicitly enabled

## IX. Complexity Summary

Let \(N\)=sequence length, \(D\)=hidden dimension, \(L\)=layers, \(H\)=heads, \(E\)=experts, \(K\)=bandlimit.

- Attention (dense): \(O(LN^2D)\)
- Attention (FFT-accelerated path): \(O(LN\log N\,D)\) under applicable kernels
- MoE compute: \(O(LEND)\)
- MoE routing: \(O(LNE)\)
- Curvature term: \(O(LND)\)

## X. Determinism Contract

A conforming π-LLM runtime must declare and enforce:

1. Fixed merge/reduction tree
2. Fixed FFT ordering
3. Canonical rounding behavior per precision mode
4. Declared precision contract
5. Declared bandlimit contract
6. Declared routing anchors/ordering

A hash of full execution-critical config is required.

## XI. Optional ZK Mode

For proof-oriented execution:

- Replace/define exp via approved polynomial form
- Use field-compatible arithmetic path
- Commit to weights and reduction order
- Emit transcript/proof artifacts

## XII. Reference Model Classes

### Class A — Spectral Micro

```text
N = 512
D = 256
L = 6
E = 2
K = 32
```

### Class B — π-Base

```text
N = 2048
D = 768
L = 12
E = 4
K = 128
```

### Class C — π-MoE Large

```text
N = 4096
D = 2048
L = 24
E = 16
K = 256
```

## XIII. Distinctive Properties

π-LLM formally combines:

1. Circular token geometry on \(S^1\)
2. Geodesic attention bias
3. Explicit spectral truncation
4. Curvature-adaptive frequency control
5. Deterministic execution contract
6. Optional ZK-verifiable execution mode

## XIV. Conditional Guarantees (Constraint-Dependent)

If curvature is bounded, bandlimit contracts are enforced, and spectral entropy is controlled, expected benefits include:

- improved smooth-signal generalization,
- reduced frequency aliasing,
- improved periodic invariance behavior,
- better stability envelopes in training/inference.

These are engineering hypotheses requiring empirical validation.

## XV. Minimal π-OS Configuration Example

```text
πOS 1.0.0

manifold circle {
  type: S1,
  resolution: 2048,
  domain: [0,2π]
}

tensor hidden {
  shape: [768, 2048],
  basis: fourier,
  precision: fp16
}

expert spectral0 {
  type: spectral,
  bandlimit: 128,
  anchor: 0.0
}

routing geo {
  mode: geodesic,
  lambda: 1.2,
  anchors: [0.0, 1.57, 3.14, 4.71]
}

execute {
  init -> attend -> route -> reduce -> commit
}
```
