# Right Ventricle Segmentation in Cardiac MRI
## Project Overview
This project implements automated **right ventricle (RV) segmentation** in **cardiac MRI**. The RV plays a key role in circulation by pumping deoxygenated blood to the lungs, yet it has historically received less attention than the left ventricle in segmentation research due to its thin wall and irregular/complex geometry (often referred to as the “forgotten ventricle”). RV morphology and function can be prognostically relevant in several cardiac conditions (e.g., congenital arrhythmogenesis, right-ventricular cardiomyopathy), and more precise RV quantification could support earlier diagnosis and monitoring.

The segmentation approach is a **compact SegFormer3D-based** model inspired by Perera et al. (2024), using a **4-stage hierarchical Transformer encoder** and a **lightweight MLP-style decoder** to produce RV masks. The model is trained and evaluated on the public **M&Ms-2** challenge dataset, with results reported separately for short-axis (SA) and long-axis (LA) views.


## Repository Structure
`RV Segmentation Code.ipynb`: Jupyter Notebook containing the full project code (data loading, model definition, training pipeline, and evaluation).

`RV Segmentation Report.pdf`: Comprehensive report detailing the project’s background, methodology, experiments, results, and improvement suggestions.

`RV Segmentation Presentation.pdf`: Project presentation slides. *(useful for a quick overview)*.

`References.bib`: BibTeX file listing literature references cited in the report.


## Data Description
**Dataset:** Multi-Disease, Multi-View & Multi-Center Right Ventricular Segmentation in Cardiac MRI (M&Ms-2) – a public challenge dataset for RV segmentation, released by the University of Barcelona (2021).

**Scope:** 360 cardiac MRI studies (patient scans) from 3 different health centers (acquired on 9 MRI scanners), covering patients with 8 different cardiac conditions and healthy controls.

**Views:** Each study provides two complementary imaging orientations:
  - Short-Axis (SA): a stack of 2D slices in cross-section.
  - Long-Axis (LA): a cine long-axis view.

**Annotations:** All scans include expert-labeled RV segmentation masks, used as ground truth for supervised learning.

**Data split used:** 144/144/72 studies for train/validation/test. Separate models were trained for SA and LA subsets to account for view-specific image characteristics.


## Project Workflow
- **Data Preprocessing & Augmentation**
- **Model Architecture**
- **Training Setup**
- **Evaluation**

## Results


## Contribution ???? 


## References
All external sources used in this project are cited in `RV Segmentation Report.pdf`. The complete BibTeX list is available in `References.bib`.
