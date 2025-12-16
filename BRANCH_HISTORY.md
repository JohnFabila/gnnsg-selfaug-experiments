# Branch History and Experiments

This document explains all the experimental branches and their results.

## Repository Structure

This repository contains the **final consolidated results** from all experiments. The actual experimental branches with commit history are tracked locally due to size constraints.

## Branch Summary

### main (current)
- **Status**: Published on GitHub
- **Contents**: All final results, documentation, plots, and trained model
- **Commit**: 649c585

### feature/new-experiments (Failed Attempt)
- **Goal**: Original self-augmentation experiment
- **Result**: -3.4% degradation
- **Issue**: Added too many low-confidence edges (~20 per query)
- **Lesson**: Quality over quantity
- **Status**: Local only
- **Key Files**: 
  - Initial augmentation implementation
  - Failed experiment logs

### feature/next-experiment (SUCCESS ✓)
- **Goal**: Head-only self-augmentation with confidence threshold
- **Result**: **+10.15% average improvement** (k=2-7)
- **Configuration**: Confidence threshold = 0.7
- **Status**: Local only
- **Key Results**:
  - k=2-4: Minimal improvement (0-0.7%)
  - k=5: Strong improvement (0.6-8.9%)
  - k=6: Massive improvement (1.3-42.7%)
  - k=7: Large improvement (1.7-37.9%)
- **Files Included in Main**:
  - `results/selfaug_confidence_070.log` (k=2-6)
  - `results/selfaug_confidence_070_k7.log` (k=7)
  - Comparison plots

### feature/next-improvements (Mixed Results)
- **Goal**: Head+Tail bidirectional self-augmentation
- **Approach**: Augment from both query head AND query tail
- **Configuration**: Confidence threshold = 0.8
- **Result**: Mixed (+2.1% to +6.0% for k=7, but worse for k=5)
- **Status**: Local only
- **Technical Challenges**:
  1. Tail nodes are sink nodes (no outgoing edges)
  2. Solution: Find incoming paths TO tail (z→a→t), add edges (h,z)
  3. Threshold increased to 0.8 to reduce edge count
- **Conclusion**: Head-only approach superior
- **Files Included in Main**:
  - `results/selfaug_head_tail_k567.log`
  - `head_tail_experiment.tex`

### feature/optimization (10x Speedup ✓)
- **Goal**: Performance optimization via batched predictions
- **Result**: **10x faster** (20 minutes vs 3 hours)
- **Coverage**: Extended to k=8
- **Status**: Local only
- **Optimization**: 
  - Batched edge predictions: N candidates = 1 model call (was N calls)
  - Bonus: +0.58% accuracy improvement from batch stability
- **K=8 Results (NEW)**:
  - (8,1): 97.89% (+5.39% vs baseline)
  - (8,2): 91.80% (+31.31% vs baseline)
  - (8,3): 89.80% (+40.08% vs baseline)
  - (8,4): 86.44% (+42.02% vs baseline)
- **Files Included in Main**:
  - `results/selfaug_optimized_k5678.log`
  - `optimization_batching.md`
  - Updated `src/train.py` with batched predictions

## Why Only One Branch on GitHub?

The full experimental branches (feature/new-experiments, feature/next-experiment, etc.) contain:
- Large data files (~2.25 GB total)
- Multiple model checkpoints at different stages
- Intermediate experiment outputs
- Full git history of all iterations

Due to GitHub size limits, these are kept locally. The **main** branch contains:
- All final results and logs
- All comparison plots
- Complete documentation
- Final optimized code
- Trained model (best checkpoint)

## Complete Results Summary

All results from all branches are preserved in this repository:

### Baseline Results
- **Classical EPIGNN**: 15 layers, 40 epochs
  - Results: `results/Classical_EPIGNN_RCC8/`
- **Short EPIGNN**: 5 layers, 5 epochs
  - Results: `results/EPIGNN_short_experiment/`

### Self-Augmentation Results
1. **Original (Failed)**: 
   - Log: `results/selfaug_experiment.log`
   - Result: -3.4%

2. **Head-only (Best)**:
   - Logs: `results/selfaug_confidence_070.log`, `results/selfaug_confidence_070_k7.log`
   - Result: +10.15% average
   - Plots: `results/SelfAug_Short_EPIGNN/SelfAug_Confidence_vs_Baselines_k2-7.pdf`

3. **Head+Tail**:
   - Log: `results/selfaug_head_tail_k567.log`
   - Result: Mixed
   - Documentation: `head_tail_experiment.tex`

4. **Optimized (k=5-8)**:
   - Log: `results/selfaug_optimized_k5678.log`
   - Result: Same accuracy, 10x faster
   - Plots: `results/SelfAug_Short_EPIGNN/SelfAug_All_Approaches_k2-8.pdf`

## Accessing Experimental Branches Locally

If you cloned this repository and want to see the full experimental history:

```bash
# These branches exist in the original local repository at:
# /scratch/john/projects/AllEPIGNN/gnnsg

cd /path/to/original/gnnsg
git branch
# Shows:
#   feature/new-experiments     (Failed -3.4%)
#   feature/next-experiment     (Success +10.15%)
#   feature/next-improvements   (Mixed results)
#   feature/optimization        (10x speedup)
#   main                        (Original code)
```

## Documentation Files

All documentation is included in the main branch:

- **README.md** - Quick start guide
- **EXPERIMENTS_SUMMARY.md** - Complete experimental timeline (9,735 bytes)
- **PLOT_GENERATION_GUIDE.md** - How to reproduce all plots (7,340 bytes)
- **optimization_batching.md** - Performance optimization details
- **head_tail_experiment.tex** - LaTeX writeup for papers
- **BRANCH_HISTORY.md** - This file

## Reproducing Results

All experiments can be reproduced from the code and data in the main branch. See [PLOT_GENERATION_GUIDE.md](PLOT_GENERATION_GUIDE.md) for detailed instructions.

## Timeline

- **Dec 15, 2025**: Original self-augmentation (failed)
- **Dec 15, 2025**: Head-only with confidence (success +10.15%)
- **Dec 16, 2025**: Head+Tail experiments (mixed results)
- **Dec 16, 2025**: Performance optimization (10x speedup)
- **Dec 16, 2025**: Extended to k=8 (new results)
- **Dec 16, 2025**: Repository published to GitHub

## Contact

For questions about experimental branches or accessing full history:
- Check commit messages in main branch
- Review [EXPERIMENTS_SUMMARY.md](EXPERIMENTS_SUMMARY.md)
- All results and code are preserved in main branch
