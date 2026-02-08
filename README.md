# Right Ventricle Segmentation in Cardiac MRI
## Project Overview
This project implements automated **right ventricle (RV) segmentation** in **cardiac MRI**. The RV plays a key role in circulation by pumping deoxygenated blood to the lungs, yet it has historically received less attention than the left ventricle in segmentation research due to its thin wall and irregular/complex geometry (often referred to as the “forgotten ventricle”). RV morphology and function can be prognostically relevant in several cardiac conditions (e.g., congenital arrhythmogenesis, right-ventricular cardiomyopathy), and more precise RV quantification could support earlier diagnosis and monitoring.

The segmentation approach is a **compact SegFormer3D-based** model inspired by [Perera et al. (2024)](https://ieeexplore.ieee.org/document/10678245), using a **4-stage hierarchical Transformer encoder** and a **lightweight MLP-style decoder** to produce RV masks. The model is trained and evaluated on the public **M&Ms-2** challenge dataset, with results reported separately for short-axis (SA) and long-axis (LA) views.


## Repository Structure
`RV Segmentation Code.ipynb`: Jupyter Notebook containing the full project code (data loading, model definition, training pipeline, and evaluation).

`RV Segmentation Report.pdf`: Comprehensive report detailing the project’s background, methodology, experiments, results, and improvement suggestions.

`RV Segmentation Presentation.pdf`: Project presentation slides. *(useful for a quick overview)*

`References.bib`: BibTeX file listing literature references cited in the report.


## Data Description
**Dataset:** [Multi-Disease, Multi-View & Multi-Center Right Ventricular Segmentation in Cardiac MRI (M&Ms-2)](https://www.ub.edu/mnms-2/) – a public challenge dataset for RV segmentation, released by the University of Barcelona (2021).

**Scope:** 360 cardiac MRI studies (patient scans) from 3 different health centers (acquired on 9 MRI scanners), covering patients with 8 different cardiac conditions, and healthy controls.

**Views:** Each study provides two complementary imaging orientations:
  - Short-Axis (SA): a stack of 2D slices in cross-section
  - Long-Axis (LA): a cine long-axis view

**Annotations:** All scans include expert-labeled RV segmentation masks, used as the ground truth for supervised learning.


## Project Workflow
### Data Preprocessing & Augmentation
- **Input format:** SA/LA cardiac MRI slices (single-channel), processed in a quasi-2D setup (D=1).
- **Preprocessing (applied to train/val/test):** 
  - Resample to a fixed voxel spacing (1.25, 1.25, 5.0 mm)
  - CropOrPad to (256, 256, 1)
  - Z-score normalization    
- **Augmentation (train only):**
  - Spatial transforms (image + mask):
    - *RandomFlip* (p=0.3): x/y axes flip
    - *RandomAffine* (p=0.3): light transformations including a ≤4% zoom-in, ≤3° in-plane rotation, and small in-plane translations
  - Intensity transforms (image only):
    - *RandomNoise* (p=0.2): Gaussian noise with σ ∈ [0, 0.01]
    - *RandomGamma* (p=0.15): contrast adjustment with log-gamma ∈ [-0.12, 0.12]
    - *RandomBiasField* (p=0.15): mild B1 inhomogeneity simulation

### Model Architecture
- **Backbone:** Compact 3D SegFormer-style segmentation Transformer (inspired by [Perera et al. (2024)](https://ieeexplore.ieee.org/document/10678245)).
- **Core idea:** no convoluted decoders, no UNet-style skip connections, just a straight-through Transformer with multi-scale outputs feeding into a slim decoder. Therefore, it remains efficient (≈ 3.9M params) while preserving strong multi-scale context.
- **Encoder:** 
  - 4-stage MiT pyramid: channels 32 → 64 → 160 → 256
  - Overlapping 3 × 3 × 3 patch embedding, stride (2,2,1) + LayerNorm
  - Blocks(×2/stage):
    - LayerNorm → MHSA (full) → +residual
    - LayerNorm → MLP (r=4) → +residual
  - Regularization: DropPath; no positional encodings
  - Shapes (D=1): ~1 × 128 × 256 → … → 1 × 16 × 256
  
  - *Diff vs [Perera et al. (2024)](https://ieeexplore.ieee.org/document/10678245):* no efficient attention / no Mix-FFN
- **Decoder:** 
  - Inputs: 4 encoder maps (32/64/160/256 ch)
  - Project: each via 1 × 1 × 1 → 128 ch
  - Upsample: trilinear to Stage-1 size
  - Fuse: concat → 512 ch
  - Head: 1 × 1 × 1, 512 → 128 → 1 (BN/ReLU/Dropout)
  - Output: logits → upsample to input size
  - *Design:* no UNet blocks; MLP-style fusion

### Training Setup
- **Data split:** 40%/40%/20% (144/144/72) for train/validation/test (patient-level split, fixed random seed) 
- **Losses, Metrics, and Logging:**
  - CombinedLoss: 2 × DiceLoss + Focal(BCEWithLogits) (α=0.5, γ=2.0)
  - Dice: sigmoid to logits → threshold 0.5 → overlap over (D,H,W), averaged across batch
  - Focal: built on BCEWithLogitsLoss, down-weights easy examples
  - TensorBoard: logs Loss/train and Loss/val per epoch
- **Training Loop & Optimization:**
  - Separate models trained for SA and LA views (distinct data loaders per view) with batch_size=1
  - Optimizer: AdamW (learning rate = 1e-4)
  - LR scheduler: ReduceLROnPlateau (patience 3 epochs, decay factor 0.5)
  - Gradient accumulation (4 steps) — simulates a larger batch under memory limits
  - Early stopping: stop after 3 epochs with no val loss improvement
  - Checkpointing: saves the best model (lowest val loss)
  - Epoch flow (50 epochs):
    - model.train() → train with backprop & accumulation
    - model.eval() → validation under no_grad

### Evaluation
- **Test data:** Held-out test set 
- **Prediction binarization:** Sigmoid output thresholded at 0.5 to yield a binary segmentation mask
- **Metrics:** Computed per sample, then averaged across test set
  - *Dice Coefficient:* overlap accuracy between predicted and ground-truth RV masks
  - *95th percentile Hausdorff Distance:* boundary discrepancy at the 95% cutoff (robust to outliers); computed only when both GT and prediction contain foreground
  - *Precision / Recall / F1:* computed voxel-wise from the binarized masks (flattened)
- **Tools:** MONAI (HD95) and scikit-learn (precision/recall/F1)


## Results
<table>
  <thead>
    <tr>
      <th rowspan="2" style="text-align:left;">Metrics</th>
      <th colspan="2" style="text-align:center;">Our SegFormer3D model</th>
      <th colspan="2" style="text-align:center;">
        nnU-Net ViT + ResBlocks + PReLU activation<br>
        (<a href="https://reference-global.com/article/10.2478/acss-2025-0002">Ayoob et&nbsp;al.,&nbsp;2025</a>)
      </th>
    </tr>
    <tr>
      <th style="text-align:center;">SA</th>
      <th style="text-align:center;">LA</th>
      <th style="text-align:center;">SA</th>
      <th style="text-align:center;">LA</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left;">Dice score</td>
      <td style="text-align:center;">0.5568</td>
      <td style="text-align:center;">0.8964</td>
      <td style="text-align:center;">0.9096</td>
      <td style="text-align:center;">0.8930</td>
    </tr>
    <tr>
      <td style="text-align:left;">Hausdorff distance</td>
      <td style="text-align:center;">76.8162</td>
      <td style="text-align:center;">62.4874</td>
      <td style="text-align:center;">66.4372</td>
      <td style="text-align:center;">15.6076</td>
    </tr>
  </tbody>
</table>

**Long-Axis (LA) View:**

- High Dice score (~0.90) → indicates strong volumetric overlap with the ground truth RV regions
- Precision is modest while Recall is high → model detects most RV pixels (sensitive) but includes some false positives
- HD95 distance remains elevated (∼62 mm) → suggests localized boundary misalignments still occur
- Competitive with nnU-Net ViT on overlap (Dice) metrics, but a higher Hausdorff implies less precise boundaries

**Strength:** Lightweight transformer achieves near-SOTA Dice with substantially lower computational cost.

**Short-Axis (SA) View:**

- Moderate Dice score (~0.56) → segmentation quality is less consistent compared to the LA view
- Low precision and recall → indicates that both false positives and false negatives are frequent
- High HD95 (~77 mm) → poor boundary alignment, likely due to the higher variability in slice orientations
- Underperforms relative to nnU-Net ViT baseline → model lacks robustness for the greater anatomical variability in this view

**Insight:** SA view exhibits more complex shape changes, suggesting the model might benefit from view-specific tuning or multi-view fusion strategies.

These findings highlight the model’s strengths in resource-constrained scenarios (lightweight architecture) in comparison to heavier models and identify some possibility to enhance the model, like integrating mechanisms or advanced activations.


## My Contributions
> Team project — the bullets below reflect my individual contributions.

- Implemented the segmentation model architecture (the `Model` section) in `RV Segmentation Code.ipynb`: SegFormer3D-inspired encoder–decoder (4-stage hierarchical Transformer encoder + lightweight MLP-style decoder).
- Built model sanity-check utilities: forward-pass validation, tensor shape checks, and output consistency checks.
- Collaborated with teammates to integrate the model into a PyTorch/MONAI pipeline, employing TorchIO augmentations and training separate models for LA/SA views.
- Authored the report section “SegFormer3D Model Architecture” in `RV Segmentation Report.pdf`.


## References
All external sources used in this project are cited in `RV Segmentation Report.pdf`. The complete BibTeX list is available in `References.bib`.
