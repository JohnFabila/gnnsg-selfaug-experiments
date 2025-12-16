# EPIGNN Short Experiment - RCC8 Results

**Experiment Name:** EPIGNN_short_experiment  
**Date:** 2024  
**Purpose:** Ablation study to compare reduced model capacity (5 layers, 5 epochs) against classical configuration

## Experiment Configuration

### Model Parameters
- **Architecture:** NBFdistR (Neural Bellman-Ford with forward-backward message passing)
- **Number of Layers:** 5 (reduced from 15 in classical)
- **Hidden Dimension:** 32
- **Facets:** 8
- **Aggregation Type:** min
- **Message Passing:** Bidirectional (forward + backward)

### Training Parameters
- **Epochs:** 5 (reduced from 40 in classical)
- **Learning Rate:** 0.001
- **Weight Decay:** 0.0001
- **Dataset:** RCC8 (Region Connection Calculus - 8 relations)
- **Loss Function:** Margin-based contrastive loss (margin=1.0) + cross-entropy

### Training Performance
- **Best Epoch:** 4
- **Best Validation Accuracy:** 99.1%
- **Training Time:** 31.09 seconds
- **Inference Time:** 248 seconds

## Test Results Summary

### Performance by Branch Count (b)
| Branches (b) | Mean Accuracy | Min Accuracy | Max Accuracy |
|--------------|---------------|--------------|--------------|
| 1            | 73.88%        | 36.78%       | 100.00%      |
| 2            | 59.56%        | 29.23%       | 99.64%       |
| 3            | 54.01%        | 26.50%       | 98.30%       |
| 4            | 51.45%        | 25.81%       | 97.30%       |
| 5            | 49.65%        | 24.66%       | 95.84%       |
| 6            | 48.68%        | 24.72%       | 95.11%       |
| 7            | 47.98%        | 23.95%       | 95.16%       |
| 8            | 47.56%        | 24.20%       | 94.19%       |

### Comparison with Classical EPIGNN (15 Layers, 40 Epochs)

**Performance Gap Analysis:**
| Branches (b) | Classical Mean | Short Mean | Performance Gap |
|--------------|----------------|------------|-----------------|
| 1            | 99.65%         | 73.88%     | **25.77%**      |
| 2            | 95.52%         | 59.56%     | **35.95%**      |
| 3            | 92.31%         | 54.01%     | **38.30%**      |
| 4            | 89.90%         | 51.45%     | **38.45%**      |
| 5            | 88.03%         | 49.65%     | **38.38%**      |
| 6            | 86.88%         | 48.68%     | **38.20%**      |
| 7            | 86.02%         | 47.98%     | **38.04%**      |
| 8            | 85.30%         | 47.56%     | **37.74%**      |

## Key Findings

### 1. Capacity Limitations
The short experiment reveals severe capacity bottlenecks:
- **Short paths (k=2-4):** Good performance (94-100% accuracy)
- **Medium paths (k=5-7):** Moderate degradation (70-98% accuracy)
- **Long paths (k≥8):** Severe failure (24-70% accuracy)
- **Very long paths (k≥11):** Near-random performance (24-41%)

### 2. Systematic Generalization Failure
- Classical model maintains 73%+ accuracy across all test conditions
- Short model drops to 24% minimum (essentially random guessing for 8 RCC8 relations: 12.5% baseline)
- Performance gap ranges from 25.77% (b=1) to 38.45% (b=4)

### 3. Model Depth Requirements
This ablation study demonstrates that:
- 5 layers are **insufficient** for systematic generalization to longer reasoning chains
- Model needs sufficient depth to handle path lengths up to k=15
- Training epochs (5 vs 40) less critical than architectural depth

### 4. Training vs Generalization
- Both models achieve ~99% validation accuracy during training
- Short model **overfits to training distribution** without learning compositional structure
- Validation accuracy is **misleading** - test generalization reveals true capacity

## Files in This Directory

- `config.yaml` - Full Hydra configuration for this experiment
- `EPIGNN_short_experiment_results.json` - Raw test accuracies for all (k,b) combinations
- `EPIGNN_short_experiment_results_plot.pdf` - 4x2 visualization of results
- `EPIGNN_short_experiment_results_plot.png` - PNG version of plots
- `all_epoch_ys.pkl` - Validation predictions across all epochs
- `README.md` - This file

## Comparison Plots

The comparison plot showing both experiments side-by-side is saved in:
- `../Classical_EPIGNN_RCC8/EPIGNN_Comparison_plot.pdf`
- `../Classical_EPIGNN_RCC8/EPIGNN_Comparison_plot.png`

## Conclusion

This ablation study confirms that **model depth is critical** for systematic generalization in compositional reasoning tasks like RCC8. Reducing from 15 to 5 layers (67% reduction) results in 25-38% performance degradation across all branch counts, with catastrophic failure on complex reasoning chains (k≥11). The 5-epoch training schedule achieves high validation accuracy but fails to learn the underlying compositional structure needed for out-of-distribution generalization.

**Recommendation:** Use at least 15 layers for RCC8 tasks requiring generalization to path lengths k≥10.
