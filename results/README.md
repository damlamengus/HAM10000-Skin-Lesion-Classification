
# Experiment Results

This directory contains reproducible experiment outputs.

## Data Splits

`ham10000_splits.csv` stores the fixed train,
validation, and test assignments.

- Random seed: 42
- Splitting unit: lesion_id
- Strategy: lesion-level stratified splitting
- Train: 6981 images
- Validation: 1532 images
- Test: 1502 images

No lesion_id appears in more than one split.

Raw dermoscopic images are not stored here.
