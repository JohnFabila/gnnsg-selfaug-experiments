# Plot Generation Guide

This document explains how to generate all the comparison plots and visualizations for the Self-Augmentation experiments.

## Overview

We compare multiple approaches for improving GNN reasoning on RCC8 spatial relations:
- **Classical**: 15 layers, 40 epochs baseline
- **Short**: 5 layers, 5 epochs baseline  
- **Head-only**: Self-augmentation from query head (confidence=0.7)
- **Head+Tail**: Bidirectional self-augmentation (confidence=0.8)
- **Optimized**: Batched predictions with k=8 support (10x faster)

## Data Sources

### Raw Results Files

All result logs are stored in `results/`:

1. **Classical baseline**: `results/Classical_EPIGNN_RCC8/test_rcc8_more_num_layers_results.json`
   - 15 layers, 40 epochs
   - k=2-7, b=1-4

2. **Short baseline**: `results/EPIGNN_short_experiment/EPIGNN_short_experiment_results.json`
   - 5 layers, 5 epochs
   - k=2-8, b=1-4

3. **Original self-aug** (failed): `results/SelfAug_Short_EPIGNN/SelfAug_Short_EPIGNN_results.json`
   - Added too many edges (-3.4% average)
   - Not used in final plots

4. **Head-only augmentation**: 
   - `results/selfaug_confidence_070.log` (k=2-6)
   - `results/selfaug_confidence_070_k7.log` (k=7)
   - Confidence threshold: 0.7
   - Average improvement: +10.15%

5. **Head+Tail augmentation**: `results/selfaug_head_tail_k567.log`
   - k=5-7, b=1-4
   - Confidence threshold: 0.8
   - Mixed results (worse than head-only)

6. **Optimized batching**: `results/selfaug_optimized_k5678.log`
   - k=5-8, b=1-4
   - Same algorithm as head+tail but 10x faster
   - Includes NEW k=8 results
   - Runtime: 20 minutes (1196 seconds)

### Model Checkpoint

All experiments use the same trained model:
- **Model**: `models/SelfAug_Short_EPIGNN_model_epoch_4.pth`
- **Architecture**: NBFdistR (5 layers)
- **Training**: 5 epochs on RCC8 dataset

## Generating Plots

### Main Comparison Plot

**Script**: `scripts/compare_selfaug_results.py`

**Generates**: 
- `results/SelfAug_Short_EPIGNN/SelfAug_All_Approaches_k2-8.png` (high-res)
- `results/SelfAug_Short_EPIGNN/SelfAug_All_Approaches_k2-8.pdf` (publication)

**Command**:
```bash
cd scripts
python compare_selfaug_results.py
```

**What it does**:
1. Loads all result files (JSON and log formats)
2. Parses accuracy values for each (k, b) configuration
3. Creates 2x2 subplot grid (one per b value)
4. Plots 5 lines per subplot:
   - Classical (green diamonds)
   - Short (blue circles)
   - Head-only (red triangles)
   - Head+Tail (orange squares)
   - Optimized (purple pentagons)
5. Prints comparison statistics table

**Output**:
- 4 subplots showing accuracy vs path length (k=2-8)
- Legend with all approaches
- Grid and axis labels
- Saved in both PNG (high-res) and PDF (vector) formats

### Data Parsing Details

The script uses regex to extract results from log files:

```python
# Pattern: (k, b): 0.accuracy
match = re.search(r'\((\d+), (\d+)\): (0\.\d+|1\.0)', line)
```

Example log line:
```
(5, 1): 0.98703125
```

Parses to:
- k=5 (path length)
- b=1 (number of blanks)  
- accuracy=0.987 (98.7%)

### Customizing Plots

To modify the plot appearance, edit `scripts/compare_selfaug_results.py`:

**Change k-range**:
```python
k_values = [2, 3, 4, 5, 6, 7, 8]  # Line ~66
```

**Change colors/markers**:
```python
ax.plot(k, accs, 'p-', linewidth=2.5, markersize=9,
        label='Optimized (10x faster)', color='purple', alpha=0.8)
```

**Marker styles**:
- `'D'` = diamond
- `'o'` = circle
- `'^'` = triangle up
- `'s'` = square
- `'p'` = pentagon

**Change y-axis range**:
```python
ax.set_ylim([0.75, 1.02])  # Line ~95
```

## Running Experiments

### Standard Test (no augmentation)

```bash
cd src
python train.py \
    experiments=fb_model_rcc8 \
    num_layers=5 \
    experiments.epochs=5 \
    experiments.exp_name=EPIGNN_short_experiment \
    experiments.test_only=true
```

### Self-Augmentation Test (optimized)

```bash
cd src
python train.py \
    experiments=fb_model_rcc8 \
    num_layers=5 \
    experiments.epochs=5 \
    experiments.exp_name=SelfAug_Short_EPIGNN \
    experiments.use_2hop_augmentation=true \
    experiments.test_only=true \
    2>&1 | tee ../results/my_experiment.log &
```

**Parameters**:
- `use_2hop_augmentation=true`: Enable self-augmentation
- `test_only=true`: Skip training, load checkpoint
- Path lengths: Configured in `src/train.py` line ~485
- Confidence thresholds: In `augment_graph_2hop_predicted()` calls

### Configuration

Edit `src/train.py` to change test configurations:

**Line ~485** - Path lengths to test:
```python
for path_len in [5, 6, 7, 8]:  # k values
```

**Line ~524** - Confidence threshold (head augmentation):
```python
confidence_threshold=0.8,
```

**Line ~533** - Confidence threshold (tail augmentation):
```python
confidence_threshold=0.8,
```

## Results Summary

### K=8 Results (NEW)

| Config | Baseline | Optimized | Improvement |
|--------|----------|-----------|-------------|
| (8,1)  | 92.50%   | 97.89%    | +5.39%      |
| (8,2)  | 60.48%   | 91.80%    | +31.31%     |
| (8,3)  | 49.72%   | 89.80%    | +40.08%     |
| (8,4)  | 44.42%   | 86.44%    | +42.02%     |

### Performance Metrics

- **Runtime**: 20 minutes for 16 configs (k=5,6,7,8 × b=1,2,3,4)
- **Speedup**: ~10x faster than sequential predictions
- **Per-config time**: ~75 seconds average
- **Edges added**: 3.7-15.1 edges per query (varies by k,b)

### Head-only vs Short Baseline

Average improvement: **+10.15%** across k=2-7, b=1-4

Best improvements:
- (6,4): +42.67%
- (7,4): +37.94%
- (6,3): +41.87%

## File Structure

```
gnnsg/
├── models/
│   └── SelfAug_Short_EPIGNN_model_epoch_4.pth  # Trained model
├── results/
│   ├── Classical_EPIGNN_RCC8/
│   │   └── test_rcc8_more_num_layers_results.json
│   ├── EPIGNN_short_experiment/
│   │   └── EPIGNN_short_experiment_results.json
│   ├── SelfAug_Short_EPIGNN/
│   │   ├── SelfAug_All_Approaches_k2-8.png  # Main plot
│   │   ├── SelfAug_All_Approaches_k2-8.pdf  # Main plot
│   │   └── SelfAug_Short_EPIGNN_results.json
│   ├── selfaug_confidence_070.log  # Head-only k=2-6
│   ├── selfaug_confidence_070_k7.log  # Head-only k=7
│   ├── selfaug_head_tail_k567.log  # Head+tail k=5-7
│   └── selfaug_optimized_k5678.log  # Optimized k=5-8
├── scripts/
│   └── compare_selfaug_results.py  # Plot generation script
└── src/
    └── train.py  # Main training/testing script

```

## Troubleshooting

### Plot shows missing data

Check that all required log files exist:
```bash
ls -lh results/selfaug*.log
ls -lh results/*/test_*.json
```

### Different results on re-run

Results may vary slightly (~0.5%) due to:
- Floating-point precision
- Batch normalization statistics
- Random seed (set to 42 by default)

### Log file parsing fails

Ensure log lines match the pattern:
```
(k, b): 0.accuracy
```

Example valid lines:
```
(5, 1): 0.98703125
(7, 4): 0.90515625
```

## Citation

If you use these results or methods, please cite:

```
[Add your paper citation here]
```

## Contact

For questions about plot generation or experiments:
- Check `optimization_batching.md` for performance details
- Check `head_tail_experiment.tex` for LaTeX writeup
- See commit history for implementation details
