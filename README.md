# DDH X‑ray preprocessing: remove labels and lines

This project provides a step‑by‑step notebook to clean DDH X‑ray images by removing overlaid measurement lines and printed labels using classic OpenCV inpainting. The current workflow centers on `notebooks/00_clean_start_preprocessing.ipynb` and supports multiple detection strategies, a mask registry, and a final “combine once, inpaint once” step. Final cleaned JPGs can be exported to a new sibling folder without touching your originals.

## Workspace layout

- `dataset/`
  - `<SIZE>/` (one of `224`, `227`, `256`, `299`, `331`)
    - `DDH/` and `Normal/` with `.jpg` images
    - Optionally: `DDH_with_labels/`, `Normal_with_labels/` (if you separated the labeled variants)
- `notebooks/`
  - `00_clean_start_preprocessing.ipynb` — main, modular pipeline (Sections 1–5)
- `results/` — manually (using https://cleanup.pictures/) or automatically produced results

## Environment (macOS, zsh)

Create a virtual environment and install the minimal dependencies:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install jupyter numpy opencv-python matplotlib
# Optional: register the venv as a Jupyter kernel
python -m ipykernel install --user --name thesis-venv --display-name "Python (thesis-venv)"
```

## Notebook workflow (Sections 1–5)

Open `notebooks/00_clean_start_preprocessing.ipynb` and run top‑to‑bottom.

1) Section 1 — Setup
- 1.1 Imports
- 1.2 Load images from `dataset/<SIZE>/...` into `images_color` and `entries_to_use`
- 1.3 Toggles (global knobs): e.g., `INPAINT_IN_GRAYSCALE`, `USE_GRAYSCALE`, line/text defaults, output roots

2) Section 2 — Working images and histograms
- 2.1 Optional grayscale conversion; previews on labeled images
- 2.2 Grayscale histograms by base class (labeled subset)

3) Section 3 — Remove lines (multiple detectors)
- 3.1 Percentile threshold on grayscale (bright line mask)
- 3.2 HoughLinesP (edge → Hough) with angle/length/gap controls
- 3.3 Derivatives (Sobel/DoG) with orientation gating
Each variant visualizes Original | Gray | Mask | Cleaned and keeps masks in memory.

4) Section 4 — Remove numbers/letters (annotations)
- Methods: `adaptive`, `mser`, or `hybrid` (union)
- Optional border‑only ROI and CC filtering (area, thinness)
- Outputs: `text_masks_all`, `text_cleaned_all`, coverage stats

Shared — Mask registry
- Small helper storing masks by `rel_path` and `method` (e.g., `thr31`, `hough32`, `deriv33`, `text4`, `text4b`)
- A registration cell collects masks from the sections you ran so Section 5 can combine them reliably

5) Section 5 — Combine masks and inpaint once
- Choose sources with `COMBINE_SOURCES` (e.g., `['thr31','hough32','text4b']`)
- Combine via logical OR, optionally erode/dilate and drop tiny CCs
- Inpaint once to produce `final_cleaned_all` and `final_masks_all`
- Visualize Original | Gray | Final mask | Final cleaned

Saving final JPGs
- Enable integrated saving in Section 5 (toggle `SAVE_FINAL_JPG = True`) or run the dedicated “Save final cleaned images (JPG)” cell at the bottom of the notebook.
- Output root defaults to `results/<SIZE>/cleaned_automatically/`, optionally preserving class subfolders.

## Data layout expectations

```
dataset/
  331/
    DDH/*.jpg
    Normal/*.jpg
    DDH_with_labels/*.jpg       
    Normal_with_labels/*.jpg    
```

Originals are never modified. Cleaned outputs are written to sibling folders.


