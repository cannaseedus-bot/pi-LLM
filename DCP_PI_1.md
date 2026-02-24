# π-LLM Distributed Cluster Protocol

**Identity:** `DCP-π-1.0.0`

This specification defines deterministic distributed execution for π-LLM across:

- Spectral weights
- Expert shards
- Attention heads
- FFT compute
- Optional ZK transcripts

## 1. Node Roles

Cluster nodes are deterministic and typed. Roles may overlap on the same machine.

| Role | Responsibility |
| --- | --- |
| Coordinator | Phase orchestration + transcript commit |
| Spectral Node | Fourier-domain operations |
| Attention Node | QKV + softmax execution |
| MoE Node | Expert execution |
| FFT Node | Deterministic FFT/IFFT |
| Proof Node | ZK generation / verification |

## 2. Sharding Strategy

π-LLM supports three shard modes.

### 2.1 Frequency Sharding

Split by disjoint spectral bins:

\[
[0\ldots K_1], [K_1\ldots K_2], \ldots
\]

Each node handles a disjoint frequency slice.

**Advantages:**
- Naturally aligned with periodic structure
- Reduced cross-node synchronization

### 2.2 Expert Sharding

Each node owns a subset of experts:

\[
E_1, E_2, \ldots, E_m
\]

Router dispatch sends tokens only to required expert nodes.

### 2.3 Head Sharding

Attention heads are partitioned across nodes.

## 3. Deterministic Phase Broadcast

Coordinator emits a phase envelope:

```json
{
  "step_id": 0,
  "layer": 0,
  "phase": "forward|backward|reduce",
  "input_hash": "",
  "precision_contract": "fp16|fp32|spectral-int4",
  "reduction_tree_id": ""
}
```

All nodes must:
1. Verify input hash
2. Execute phase deterministically
3. Emit output hash

## 4. Reduction Consensus

All-reduce constraints:

- Fixed binary-tree topology
- Ordered node IDs
- No dynamic reordering

Canonical merge:

```text
reduce(a, b) -> canonical_merge(a, b)
```

Merge behavior MUST be bitwise deterministic.

## 5. Fault Model

DCP-π requires:

- Byzantine detection via hash mismatch
- Deterministic replayability
- Snapshot hashing at per-layer boundaries

## 6. Communication Format

Binary frame structure:

```text
[ header ]
[ tensor_metadata ]
[ tensor_payload ]
[ hash ]
```

Required tensor metadata:

```json
{
  "basis": "",
  "bandlimit": 0,
  "resolution": 0,
  "precision": ""
}
```

Nodes MUST reject metadata-contract mismatches.

## 7. Scaling Behavior

Let:

- \(N\): sequence length
- \(L\): layer count
- \(E\): expert count
- \(K\): spectral bandlimit

Expected parallel scaling contributions:

- \(O(\text{K-shard count})\) for spectral partitioning
- \(O(\text{E-shard count})\) for MoE partitioning
- \(O(\text{H-shard count})\) for head partitioning
