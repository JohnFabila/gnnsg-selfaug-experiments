# Classical EPIGNN - RCC8 Experiment Results

## Experiment Details

**Model**: Classical EPIGNN (NBFdistR with Forward/Backward Propagation)  
**Dataset**: RCC8 Spatial Relations  
**Training**: 40 epochs on `train_rcc8.csv`  
**Best Epoch**: 35 (99.91% validation accuracy)  
**Training Time**: 449.8 seconds (~7.5 minutes)  
**Inference Time**: 291.8 seconds (~4.9 minutes)

## Architecture Parameters

- **Model Type**: NBFdistR (Forward-Backward Message Passing)
- **Hidden Dimension**: 32
- **Facets**: 8
- **Number of Layers**: 15 message passing rounds
- **Aggregation Type**: `min` (minimum aggregation)
- **Loss Function**: Margin-based contrastive loss with cross-entropy
- **Learning Rate**: 0.01
- **Batch Size**: 128

## Results Summary

### Performance by Branch Count (b)

| Branches (b) | Mean Accuracy | Min Accuracy | Max Accuracy |
|--------------|---------------|--------------|--------------|
| b=1          | 99.65%        | 98.97%       | 100.00%      |
| b=2          | 95.52%        | 90.28%       | 100.00%      |
| b=3          | 92.31%        | 83.97%       | 99.95%       |
| b=4          | 89.90%        | 80.73%       | 99.88%       |
| b=5          | 88.03%        | 77.89%       | 99.77%       |
| b=6          | 86.88%        | 76.16%       | 99.69%       |
| b=7          | 86.02%        | 75.97%       | 99.59%       |
| b=8          | 85.30%        | 73.16%       | 99.48%       |

### Key Observations

1. **Near-perfect performance on simple graphs** (b=1): >99% accuracy across all path lengths
2. **Graceful degradation with complexity**: Performance decreases as branch count increases
3. **Strong short-path performance**: k=2-4 maintains >99% accuracy even with 8 branches
4. **Systematic generalization challenge**: Accuracy drops to ~73-90% for k=15, b=8 (hardest case)

## Files Generated

- `Classical_EPIGNN_RCC8_results_plot.pdf` - High-quality vector plot
- `Classical_EPIGNN_RCC8_results_plot.png` - Raster image (668KB)
- `test_rcc8_more_num_layers_results.json` - Raw accuracy data
- `config.yaml` - Experiment configuration
- `all_epoch_ys.pkl` - Learned relation embeddings per epoch

## Visualization

The generated plot shows 8 subplots (4x2 grid), one for each branch count:
- **X-axis**: Path length (k) from 2 to 15
- **Y-axis**: Accuracy percentage (70-102%)
- Each subplot shows how accuracy varies with path length for a fixed branch count

## Comparison Baseline

This serves as the **Classical EPIGNN baseline** for the RCC8 spatial reasoning task. Future experiments can compare against these results to evaluate improvements in:
- Systematic generalization to longer paths
- Robustness to graph complexity (higher branch counts)
- Sample efficiency during training
- Inference speed

---

**Experiment Date**: December 12, 2025  
**Location**: `/scratch/john/projects/AllEPIGNN/gnnsg/results/Classical_EPIGNN_RCC8/`
