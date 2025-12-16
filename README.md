# Self-Augmentation Experiments for GNN Reasoning

This repository contains all experiments and results for improving Graph Neural Network reasoning on long-path queries using self-augmentation with predicted edges.

## 🎯 Key Results

- **+10.15% average improvement** with head-only self-augmentation (k=2-7)
- **+42.67% improvement** on hardest cases (k=6, b=4)
- **10x faster** with batched predictions (20 min vs 3 hours)
- **k=8 results**: +5.4% to +42.0% improvement over baseline

## 📊 Quick Start

### View Results
```bash
# Main comparison plot
open results/SelfAug_Short_EPIGNN/SelfAug_All_Approaches_k2-8.pdf
```

### Generate Plots
```bash
cd scripts
python compare_selfaug_results.py
```

## 📁 Repository Structure

```
├── README.md                           # This file
├── EXPERIMENTS_SUMMARY.md              # Complete timeline
├── PLOT_GENERATION_GUIDE.md            # Reproduction guide
├── optimization_batching.md            # Performance details
├── head_tail_experiment.tex            # LaTeX writeup
├── src/train.py                        # Main code
├── scripts/compare_selfaug_results.py  # Plot generation
├── results/                            # All result logs & plots
└── models/                             # Trained model
```

## 📖 Documentation

- **[EXPERIMENTS_SUMMARY.md](EXPERIMENTS_SUMMARY.md)** - Complete experimental timeline
- **[PLOT_GENERATION_GUIDE.md](PLOT_GENERATION_GUIDE.md)** - Reproduction instructions
- **[optimization_batching.md](optimization_batching.md)** - Performance optimization
- **[head_tail_experiment.tex](head_tail_experiment.tex)** - Paper writeup

## 📈 Results Summary

| Approach | Improvement | Runtime | Status |
|----------|------------|---------|---------|
| Head-only | **+10.15%** | 3h | ✓✓ |
| Optimized | Same | **20m** | ✓✓✓ |

**Last Updated**: December 16, 2025
