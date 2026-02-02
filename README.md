# Right Ventricle Segmentation in Cardiac MRI

## Project Overview
**Objective:** Automate the segmentation of the right ventricle (RV) in cardiac MRI scans (both short-axis and long-axis views) using advanced deep learning techniques.

**Significance:** Accurate RV segmentation is crucial for evaluating heart function and detecting diseases, yet it is challenging due to the RV’s thin walls and complex shape (often called the “forgotten ventricle” in cardiac imaging).

**Approach:** Developed a lightweight 3D Transformer-based model (SegFormer3D architecture) tailored for RV segmentation. The model was trained and evaluated on a multi-center MRI dataset, demonstrating robust performance across varied scanners and cardiac conditions.

## Project Overview
- **Task:** Right ventricle (RV) segmentation in cardiac MRI (binary mask), trained/evaluated separately for **short-axis (SA)** and **long-axis (LA)** views.
- **Clinical motivation:** RV quantification is clinically relevant, yet RV segmentation is challenging due to **thin walls** and **complex geometry** (“forgotten ventricle”).
- **Method:** Compact **SegFormer3D-inspired** model (Perera et al., 2024):
  - 4-stage Transformer encoder with **overlapping 3×3×3** patch embedding (stride **(2,2,1)**), **no positional embeddings**
  - lightweight **all-MLP decoder** with multi-scale fusion + upsampling to input resolution
- **Data:** **M&Ms-2** public challenge dataset — **360** annotated studies, multi-center (**3** centers, **9** scanners), multi-disease (**8** pathologies), **SA/LA** views.
- **Outcome:** Strong performance on **LA** and moderate performance on **SA** under the same evaluation protocol (see Results).


## Repository Structure
`RV Segmentation Code.ipynb`: Jupyter Notebook containing the full project code (data loading, model definition, training pipeline, and evaluation).

`RV Segmentation Presentation.pdf`: Slide deck summarizing the project’s motivation, approach, and key findings *(useful for a quick overview)*.

`RV Segmentation Report.pdf`: Comprehensive report detailing the project’s background, methodology, experiments, and results.

`References.bib`: BibTeX file listing literature references cited in the report.

## References
Full references for all cited works are provided in the `References.bib` file.
