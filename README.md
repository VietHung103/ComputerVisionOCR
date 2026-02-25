Hybrid Method For Receipt OCR 

**Authors:** Hung Le Viet, Thanh Doan Duy, Nguyen Gia Nam, Dang Dinh Hoa   
**Institution:** University of Science and Technology of Hanoi (USTH)  
**Lecturer:** Nguyen Duc Dung  
**Date:** February 2026  

##  Introduction
This project implements a robust two-stage Optical Character Recognition (OCR) pipeline designed for receipt understanding.  Automatic receipt OCR systems face significant challenges such as complex layouts, diverse fonts, low image quality, and skewed text orientations. 

To address these, our pipeline is divided into two main tasks:
1.  **Text Detection:** Utilizing **DBNet** (Differentiable Binarization Network) to accurately localize arbitrarily shaped and closely spaced text regions.
2.  **Text Recognition:** Utilizing **SVTR** (Scene Text Visual Transformer) combined with **CTC** (Connectionist Temporal Classification) loss for alignment-free, line-level sequence prediction.

The system is evaluated on the ICDAR 2019 SROIE dataset and benchmarked against existing implementations to analyze performance improvements.

---

##  Dataset
The dataset is derived from the **ICDAR 2019 SROIE (Scanned Receipt OCR and Information Extraction)** challenge. 
* Consists of real-world scanned receipts from various retail stores
* Exhibits diverse layouts, font styles, resolutions, and printing qualities.
* Contains challenges like skewed text, low contrast, background noise, and faded printing.

---

##  Methodology

### Task 1: Text Detection (DBNet)

DBNet is a segmentation-based model utilizing a differentiable binarization module to provide precise boundary predictions.

* **Data Augmentation:** * *Geometric Transformations:* Random rotation ($-10^\circ$ to $+10^\circ$) and Random Crop.
    * *Photometric Transformations:* Color Jitter, Motion Blur, and Noise.
* **Pre-processing:**
    * Images are downscaled proportionally to preserve the Aspect Ratio and padded into a standardized $640 \times 640$ black canvas.
    * Pixel values are normalized to the [0, 1] space.
* **Label Generation (Shrink Map):** Original text polygons are actively shrunk into "core text regions" using the Vatti clipping algorithm to create artificial boundary gaps, separating dense text clusters.
* **Architecture:** Comprises a ResNet-18 convolutional backbone, a Feature Pyramid Network (FPN) for multi-scale fusion, and a lightweight prediction head. Optimized using Binary Cross-Entropy (BCE) loss.
* **Post-processing:** The predicted probability map undergoes binarization (threshold $T=0.3$), contour extraction, polygon expansion (Vatti clipping with ratio $1.5$), and coordinate un-scaling back to the original image dimensions.

### Task 2: Text Recognition (SVTR + CTC)

The recognition stage consumes cropped line images and predicts the character sequence.

* **Pre-processing:**
    * Crops are converted to single-channel grayscale.
    * Aspect-ratio preserving resize is applied to fit within a target size of $H=32, W=256$.
    * Images are right-padded to a fixed $256 \times 32$ canvas and pixel values are normalized to approximately [-1, 1].
* **Architecture:** * *Patch Embedding:* Compresses height and splits width into patches, creating a token sequence.
    * *Positional Embedding:* Added to encode left-to-right reading order.
    * *Transformer Encoder:* Six identical layers with multi-head self-attention to model long-range dependencies.
    * *Linear Classifier:* Maps tokens to character logits including a CTC blank class.
* **Decoding:** Uses CTC loss for training and greedy decoding at inference to yield the predicted text string.

---

##  Results & Evaluation

### Task 1: Text Detection Performance
After 50 epochs of training, the DBNet model showed stable convergence (Training Loss: 0.0281). The performance metrics are:
* **Precision:** 79.1% 
* **Recall:** 91.9% 
* **H-Mean (F1-Score):** 85.0% 

*Highlight:* The model significantly elevated the Recall score to 91.9%, preventing the omission of blurred or closely printed text compared to baseline approaches.

### Task 2: Text Recognition Performance
Evaluated using the ICDAR-style exact line matching protocol (case-insensitive).
* **Train Hmean:** 80.16% 
* **Validation Hmean:** 77.62% 
* **Test Hmean:** 75.76% 

The SVTR+CTC configuration reliably recognizes short to medium alphanumeric strings and typical receipt phrases, providing a robust foundation for subsequent information extraction.

---

##  References
* [1] A. Graves et al., "Connectionist Temporal Classification...", ICML 2006. 
* [2] B. Shi et al., "An End-to-End Trainable Neural Network for Image-based Sequence Recognition...", TPAMI 2017. 
* [3] M. Liao et al., "Real-time Scene Text Detection with Differentiable Binarization," AAAI 2020. 
* [4] S. Huang et al., "ICDAR 2019 Competition on Scanned Receipt OCR and Information Extraction," ICDAR 2019. 
* [5] R. Zhang et al., "SVTR: Scene Text Recognition with a Single Visual Model," arXiv 2022. 
* [6] zzzDavid, "ICDAR-2019-SROIE", GitHub repository, 2019.
