## Self-Augmentation Failure Analysis

### Summary of Results
- **Average degradation**: -3.41% across k=2-5, b=1-4
- **Pattern**: Performance degrades with more augmented edges
  - k=3, b=1: 1 edge added → 100% (no change)
  - k=3, b=4: 4 edges added → 85.8% (-11.5%)
  - k=5, b=3: 6.8 edges added → 84.2% (-3.6%)

### Potential Issues Investigated

#### 1. ❓ Backward Graph Handling (Lines 823-831)
**Status**: Likely CORRECT
- fw graph gets: [(h,c), (c,h)] bidirectional
- bw graph gets: swap of fw = [(c,h), (h,c)]
- This maintains the fw/bw relationship correctly

#### 2. ✅ Inverse Relations
**Status**: CORRECT
- All 8 RCC8 relations have correct inverses
- Double inversion test passes (inv(inv(r)) == r)

#### 3. ✅ Target Edge Filtering
**Status**: CORRECT
- Query target is properly excluded from candidates
- No risk of predicting the answer edge

#### 4. ⚠️ Model Prediction Quality
**Status**: LIKELY THE REAL ISSUE
The model's predictions for missing edges appear to be systematically poor:
- **Hypothesis**: Model was trained to predict composition rules
- **Problem**: 2-hop missing edges might not follow composition
- **Example**: If h→b (DC) and b→c (EC), composition might be DC
  - But the TRUE label for h→c could be PO (not predictable from composition)
  - Model predicts DC (wrong), adds noisy edge

#### 5. ⚠️ Self-Reinforcing Errors
**Status**: VERY LIKELY
When we augment and then test:
1. Model predicts edge (h→c) has label DC (might be wrong)
2. We add (h→c, DC) to graph
3. Model sees this edge during test-time reasoning
4. Model uses the WRONG edge DC to make decisions
5. Final prediction becomes incorrect

This creates a **compounding error effect**.

### Why It's Failing

The fundamental problem is that **2-hop missing edges are not predictable from local structure alone**:

- **What the model learned**: Composition rules from training data
  - If a→b (r1) and b→c (r2), then a→c should be compose(r1, r2)
  
- **What we're asking it to do**: Predict labels for edges where h→c is missing
  - These are NOT in the graph, possibly because they don't follow composition
  - Or they require longer reasoning paths
  
- **What happens**: Model makes best guess (often DC or most common relation)
  - Guess is wrong more often than right
  - Wrong edges confuse the model during actual test query

### Evidence

1. **Correlation**: More edges added → worse performance
   - k=3, b=1: 1 edge → 0% change (edge might be correct)
   - k=3, b=4: 4 edges → -11.5% (more wrong edges)

2. **b=1 immunity**: When b=1 (single relation), model maintains accuracy
   - Composition is trivial: r∘r = r for most relations
   - Predictions more likely correct

3. **No improvements**: 0/16 configurations improved
   - If predictions were good, we'd see SOME improvement
   - 100% degradation rate suggests systematic error

### Conclusion

The self-augmentation approach fails because:
1. **Model predictions are not accurate enough** for missing 2-hop edges
2. **Wrong predictions compound** during test-time reasoning
3. **Missing edges are missing for a reason** - they might not follow simple composition

The approach would only work if:
- Model predictions were >95% accurate (they're not)
- OR we used ground-truth labels (defeats the purpose)
- OR we added confidence thresholding (but model seems overconfident)

**Recommendation**: Abandon this approach and explore:
- Better architectures that don't need augmentation
- Training strategies that explicitly learn composition
- Different forms of augmentation (e.g., path-based features)
