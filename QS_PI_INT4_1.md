# π-LLM INT4 Spectral Quantization Specification

**Identity:** `QS-π-INT4-1.0.0`

This specification defines Fourier-domain quantization for π-LLM weights and/or activations.

## 1. Motivation

Spectral coefficients are assumed to decay approximately as:

\[
|a_k| \sim 1/k^2
\]

Practical implication:
- Low-frequency bins preserve most energy
- High-frequency bins can be quantized more aggressively

## 2. Quantization Layout

For each Fourier bin pair \((a_k, b_k)\), store:

- Sign bit
- 4-bit magnitude
- Shared per-block scale

Default block size: **32 frequency bins**.

## 3. INT4 Packing

Per coefficient:
- 4-bit magnitude
- 1 sign bit (separately packed stream or packed sideband)
- fp16 shared scale per block

Reference block layout:

```text
uint16 scale
uint8 coeffs[block_size/2]   # two 4-bit magnitudes per byte
bitset signs[block_size]      # sign stream
```

## 4. Adaptive Precision Policy

Dual precision regioning:

- Low-frequency bins \((k \le K_{low})\): INT8 or FP16
- High-frequency bins \((k > K_{low})\): INT4

Policy parameter:

```text
low_precision_boundary = K_low
```

## 5. Quantization Rule

For value \(x\):

\[
q = \mathrm{round}(x / \mathrm{scale}), \quad |q| \le 7
\]

Per-block scale:

\[
\mathrm{scale} = \max(|x|) / 7
\]

## 6. Error Bound (Block-Level)

For block reconstruction:

\[
\|x - \hat{x}\|_2 \le \mathrm{scale} \cdot \sqrt{\mathrm{block\_size}}
\]

Given spectral decay, aggregate loss should remain bounded in low-k-dominant regimes.

## 7. Dequantization

Deterministic dequantization:

\[
\hat{x} = q \cdot \mathrm{scale}
\]

All dequant paths must be reduction-order deterministic under DCP-π.
