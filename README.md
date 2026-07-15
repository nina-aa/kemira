# Hyperspectral Imaging → Concentration Prediction (PLS Regression with White/Dark Reference Correction)

## What this does

This notebook builds a model that predicts a chemical concentration of phosphorus in liquid samples directly from **hyperspectral imaging (HSI) data**, instead of relying on a wet-lab assay for every sample.

Pipeline, step by step:

1. **Load ground-truth concentrations** from a CSV of lab measurements 
2. **Load hyperspectral cubes** (`.hdr`/`.bin` ENVI format) for 12 training samples, each a 3D array of `(height, width, spectral bands)`.
3. **Radiometric correction**: apply the standard white/dark reference correction —
   `corrected = (sample − dark) / (white − dark)`
   — so that raw sensor readings are turned into calibrated reflectance values, independent of lighting conditions and sensor dark current. Without this step, spectral readings shift with lighting/sensor drift and the model would not generalize across sessions.
4. **Signal preprocessing** on the mean spectrum per sample: min-max normalization → Savitzky-Golay smoothing → Gaussian denoising. This reduces high-frequency noise while preserving the shape of absorption/reflectance peaks that carry the chemical signal.
5. **Feature scaling** with `StandardScaler`.
6. **Model training**: Partial Least Squares (PLS) Regression, chosen because it handles many correlated spectral bands with few samples — a classic chemometrics approach for spectral data.
7. **Validation**: Leave-One-Out Cross-Validation (LOOCV), appropriate given the small sample size (12 training samples), reporting R².
8. **Held-out testing**: the trained model is applied to 3 completely unseen samples, comparing predicted vs. actual concentration and visualizing both LOOCV and test results on the same regression plot.
9. **Model persistence**: the fitted PLS model and scaler are saved with `joblib` for reuse without retraining.

There's also a visualization section (before the training section) that plots raw, normalized, smoothed, and denoised spectra per sample — useful for sanity-checking the data before committing to a model.

## Method Notes

**Why white/dark correction**: Applying white/dark reference correction *before* modeling is necessary as raw sensor data isn't directly usable, and skipping it silently produces a model that overfits to one lighting setup.
**Why PLS Regression**: spectral data has many highly collinear features (adjacent wavelength bands) and typically few samples relative to feature count — PLS is a standard chemometrics technique for exactly this setup.
**Why LOOCV**: with a small sample size, leave-one-out cross-validation makes efficient use of limited data while still giving an unbiased performance estimate.
**Reproducible evaluation.** The model is tested against 3 unseen samples, and both LOOCV and test predictions are plotted against the ideal 1:1 line so errors are visible, not hidden behind a single summary metric.

## Data

The notebook expects the following in the same directory:

- `pollution_data.csv` — ground-truth phosphorus concentration measurements
- `swiss/` — folder containing hyperspectral sample cubes (`.hdr` + `.bin` pairs) and reference files (`whiteReference.hdr/.bin`, `darkReference.hdr/.bin`)

These files aren't included in this repository.



