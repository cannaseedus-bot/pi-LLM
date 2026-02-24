# π-LLM Training Blueprint

**Identity:** `TB-π-1.0.0`

This blueprint defines deployment-grade training behavior for π-LLM.

## 1. Training Objective

Autoregressive objective:

\[
\mathcal{L}_{AR} = -\sum_i \log P(x_i \mid x_{<i})
\]

With spectral entropy regularization:

\[
\mathcal{L}_{total} = \mathcal{L}_{AR} + \lambda H_\pi
\]

## 2. Optimizer: π-AdamW

Moment updates:

\[
m_t = \beta_1 m_{t-1} + (1-\beta_1)g_t
\]
\[
v_t = \beta_2 v_{t-1} + (1-\beta_2)g_t^2
\]

Bias correction:

\[
\hat{m}_t = m_t/(1-\beta_1^t), \quad \hat{v}_t = v_t/(1-\beta_2^t)
\]

Update rule:

\[
W_{t+1} = W_t - \eta\frac{\hat{m}_t}{\sqrt{\hat{v}_t}+\epsilon} - \eta\lambda W_t
\]

All reductions must follow deterministic ordering constraints.

## 3. Spectral Regularization

Post-step constraints:

1. **Low-pass enforcement**
   \[
   a_k, b_k = 0 \quad \text{for } k > K_{max}
   \]

2. **Optional spectral shrinkage**
   \[
   a_k \leftarrow \frac{a_k}{1 + \alpha k^2}
   \]

## 4. Learning-Rate Schedule

Cosine decay over horizon \(T\):

\[
\eta_t = \eta_{max}\frac{1}{2}(1 + \cos(\pi t/T))
\]

Include linear warmup for first \(W\) steps.

## 5. MoE Load-Balancing Loss

Auxiliary balancing term:

\[
\mathcal{L}_{balance} = \sum_k\left(\frac{1}{N}\sum_i G_k(\theta_i)-\frac{1}{E}\right)^2
\]

Encourages uniform expert utilization.

## 6. Mixed-Precision Modes

| Mode | Forward | Gradient/Optimizer |
| --- | --- | --- |
| fp32 | fp32 | fp32 |
| fp16 | fp16 | fp32 master |
| spectral-int4 | dequant→fp16 | fp32 master |

Master weights remain fp32 in all modes.

## 7. Distributed Training Procedure

Run under DCP-π:

1. Local forward
2. Local backward
3. Deterministic-tree all-reduce gradients
4. Apply optimizer update
5. Commit step hash

Optional: attach recursive proof per step.

## 8. Checkpoint Contract

Checkpoint payload:

```json
{
  "weight_hash": "",
  "spectral_config": {},
  "optimizer_state_hash": "",
  "step": 0,
  "cluster_hash": ""
}
```
