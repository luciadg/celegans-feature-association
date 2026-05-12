# Automated C. elegans Segmentation and Feature Association: Mapping Genes and Pharynxes
> [!IMPORTANT]
> **Notebook rendering issues?**
> **[Click here to view the Notebook with full results and images on NBViewer](https://nbviewer.org/github/luciadg/celegans-feature-association/blob/main/celegans_feature_association.ipynb)**

Academic project developed for the Master's in Bioinformatics (UDC). A specialized biomedical imaging and computer vision pipeline designed to segment *Caenorhabditis elegans* individuals and perform precise **spatial association** of their internal biological structures (pharynxes and `clec-60` gene expression foci) using multichannel 16-bit TIFF images.

## 1. The Spatial Association Challenge
In multi-organism images, simply detecting features is insufficient; the core challenge is correctly attributing each detected gene or pharynx to the exact bodily mask of the corresponding nematode.

This project solves this structural problem through:
- **Independent Normalization:** Min-Max rescaling (to uint8 [0-255]) applied per channel to prevent the brightfield channel from attenuating weaker fluorescent signals.
- **Hierarchical Data Architecture:** Implementation of nested dictionaries to store IDs, contours, masks, and quantitative feature counts.
- **Centroid-in-Mask Logic:** Validating the ownership of an internal structure by checking if its (X, Y) spatial coordinates fall within a region strictly `>0` in an individual's binary mask.

## 2. Dataset Specifications
The images utilized in this pipeline are part of the **Broad Bioimage Benchmark Collection**. These high-resolution files are not hosted in this repository due to their 16-bit depth and substantial size.

- **Source:** [Broad Bioimage Benchmark Collection (BBBC)](https://bbbc.broadinstitute.org/BBBC012)
- **Quantity:** 8 multichannel images.
- **Dimensions:** 696 x 520 pixels.
- **Bit Depth:** 16-bit (TIFF format).
- **Dynamic Range:** Intensity values between 100 and 2000.
- **Channels:** - **w1:** Brightfield (Anatomical reference).
    - **w2:** GFP (`clec-60` gene expression).
    - **w3:** mCherry (Pharynx localization).

## 3. Processing Methodology
The workflow is fully automated, applying specific computer vision strategies tailored to each channel's characteristics:

- **Anatomical Segmentation (w1 - Brightfield):** To mitigate uneven background illumination, a median filter is applied, followed by a **Black-Hat** morphological transformation (61x61 elliptical kernel) to extract dark structures. Individuals are isolated using Adaptive Gaussian Thresholding and cleaned via a Morphological Opening (3x3 kernel). Artifacts are removed using an area filter (>300 pixels).

- **Gene Expression (w2 - GFP):** Operating on a dark, uniform background, a global binary threshold (Threshold = 48) is rigorously tuned and applied to discriminate true expression from the amplified basal autofluorescence of the nematode's body.

- **Pharynx Localization (w3 - mCherry):** A global threshold (Threshold = 40) is combined with a targeted morphological dilation (4x4 circular kernel, 2 iterations). This is specifically designed to successfully fuse the two pharyngeal lobes into a single quantifiable mass.

## 4. Results and Model Limitations
The algorithm generates a detailed 1:N mapping with zero manual intervention, identifying an expression range of **0 to 6 genes per individual** (with the majority clustered between 0 and 3).

**Deviation Analysis:**
- Association accuracy is highly dependent on successful individual separation in the w1 channel. Nematodes in extreme proximity are occasionally segmented as a single continuous object.
- The dilation operation in the w3 channel presents a biological trade-off: while it successfully fuses individual lobes, in high-density areas it may cause the accidental fusion of pharynxes from adjacent nematodes. This shifts the resulting centroid outside the boundaries of either bodily mask, preventing proper association.

## 5. Environment Requirements
Install the required dependencies to replicate the environment and ensure proper reading of 16-bit `.tif` files:
```bash
pip install -r requirements.txt
