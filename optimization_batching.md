# Performance Optimization: Batched Edge Predictions

## Problem
Self-augmentation was slow because we predicted edge labels **sequentially**:
- For each query graph, find 2-8 candidate edges
- For each candidate, call model separately: `predict_edge_label()`
- Each call: clone graph → create batch → model forward → softmax
- Result: 100-200+ model calls per test set

## Solution: Batch Predictions

### Implementation
Added `predict_edge_labels_batch()` function that:
1. Takes list of target nodes (all from same source)
2. Creates batch with all queries
3. Runs **single model forward pass** for all predictions
4. Returns list of predictions

### Code Changes
```python
# OLD (Sequential - SLOW)
for candidate in candidates:
    pred = predict_edge_label(model, data, head, candidate)  # One call per edge
    if pred is not None:
        new_edges.append((head, candidate, pred))

# NEW (Batched - FAST)  
pred_labels = predict_edge_labels_batch(model, data, head, candidates)  # One call total
for candidate, pred in zip(candidates, pred_labels):
    if pred is not None:
        new_edges.append((head, candidate, pred))
```

### Expected Speedup
- **8 candidates**: 8x faster (8 calls → 1 call)
- **Typical case (2-8 edges/query)**: 3-8x speedup
- **GPU utilization**: Much better with batched operations

### Files Modified
- `src/train.py`:
  - Added `predict_edge_labels_batch()` (lines ~755-829)
  - Updated `augment_graph_2hop_predicted()` to use batching
  - Updated `augment_graph_2hop_predicted_backward()` to use batching

## Further Optimization Ideas

### 1. Reduce Graph Cloning (Medium effort, 2x speedup)
Currently clone graph for each prediction. Could:
- Share graph structure across batch
- Only duplicate query-specific data

### 2. Cache Node Embeddings (High effort, 3-5x speedup)
Model computes same node embeddings for all predictions on same graph:
- Cache embeddings after first few layers
- Reuse for all candidate predictions
- Requires model architecture changes

### 3. Parallelize Test Batches (Low effort, 2-4x speedup)
Use multiprocessing to process multiple test graphs in parallel:
```python
from multiprocessing import Pool
with Pool(4) as p:
    results = p.map(augment_and_test, test_batches)
```
- Needs careful GPU memory management
- Best for CPU-heavy preprocessing

### 4. Early Stopping (Low effort, variable speedup)
Stop searching for candidates after finding N high-confidence edges:
```python
if len(high_conf_edges) >= MAX_EDGES_PER_QUERY:
    break
```

### 5. Hierarchical Filtering (Medium effort, 2x speedup)
Pre-filter candidates with cheap heuristics before expensive model:
- Distance-based: skip candidates too far from tail
- Degree-based: prefer high-degree intermediate nodes
- Relation-type filtering: some relation chains are more likely

## Benchmark Results
TODO: Run timing comparison after implementing batching

### Expected Performance
| Config | Before (est) | After (est) | Speedup |
|--------|-------------|-------------|---------|
| k=5, b=1-4 | ~180s | ~40s | 4.5x |
| k=6, b=1-4 | ~300s | ~60s | 5x |
| k=7, b=1-4 | ~450s | ~80s | 5.6x |
| **Total (k=5-7)** | **~930s** | **~180s** | **5x** |

Based on: avg 5-8 candidates per query × 3 test lengths × 4 background levels
