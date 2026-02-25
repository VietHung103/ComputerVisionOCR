# [cite_start]Hybrid Method For Receipt OCR [cite: 1]

[cite_start]**Authors:** Hung Le Viet (23bi14182), Thanh Doan Duy, Nguyen Gia Nam, Dang Dinh Hoa [cite: 2]  
[cite_start]**Institution:** University of Science and Technology of Hanoi (USTH) [cite: 2]  
[cite_start]**Lecturer:** Nguyen Duc Dung [cite: 3]  
[cite_start]**Date:** February 2026 [cite: 3]  

##  Introduction
[cite_start]This project implements a robust two-stage Optical Character Recognition (OCR) pipeline designed for receipt understanding[cite: 4, 11]. [cite_start]Automatic receipt OCR systems face significant challenges such as complex layouts, diverse fonts, low image quality, and skewed text orientations[cite: 8]. 

To address these, our pipeline is divided into two main tasks:
1.  [cite_start]**Text Detection:** Utilizing **DBNet** (Differentiable Binarization Network) to accurately localize arbitrarily shaped and closely spaced text regions[cite: 12, 13, 14].
2.  [cite_start]**Text Recognition:** Utilizing **SVTR** (Scene Text Visual Transformer) combined with **CTC** (Connectionist Temporal Classification) loss for alignment-free, line-level sequence prediction[cite: 4, 15, 17].

[cite_start]The system is evaluated on the ICDAR 2019 SROIE dataset and benchmarked against existing implementations to analyze performance improvements[cite: 4, 18, 19].

---

##  Dataset
[cite_start]The dataset is derived from the **ICDAR 2019 SROIE (Scanned Receipt OCR and Information Extraction)** challenge[cite: 23]. 
* [cite_start]Consists of real-world scanned receipts from various retail stores[cite: 24].
* [cite_start]Exhibits diverse layouts, font styles, resolutions, and printing qualities[cite: 25].
* [cite_start]Contains challenges like skewed text, low contrast, background noise, and faded printing[cite: 26].

---

##  Methodology

### Task 1: Text Detection (DBNet)
[cite_start]DBNet is a segmentation-based model utilizing a differentiable binarization module to provide precise boundary predictions[cite: 13, 14].

* [cite_start]**Data Augmentation:** * *Geometric Transformations:* Random rotation ($-10^\circ$ to $+10^\circ$) and Random Crop[cite: 33, 35].
    * [cite_start]*Photometric Transformations:* Color Jitter, Motion Blur, and Noise[cite: 37].
* **Pre-processing:**
    * [cite_start]Images are downscaled proportionally to preserve the Aspect Ratio and padded into a standardized $640 \times 640$ black canvas[cite: 39, 40, 48].
    * [cite_start]Pixel values are normalized to the [0, 1] space[cite: 49].
* [cite_start]**Label Generation (Shrink Map):** Original text polygons are actively shrunk into "core text regions" using the Vatti clipping algorithm to create artificial boundary gaps, separating dense text clusters[cite: 51, 54, 55].
* [cite_start]**Architecture:** Comprises a ResNet-18 convolutional backbone, a Feature Pyramid Network (FPN) for multi-scale fusion, and a lightweight prediction head[cite: 130, 132, 136, 144]. [cite_start]Optimized using Binary Cross-Entropy (BCE) loss[cite: 157].
* [cite_start]**Post-processing:** The predicted probability map undergoes binarization (threshold $T=0.3$), contour extraction, polygon expansion (Vatti clipping with ratio $1.5$), and coordinate un-scaling back to the original image dimensions[cite: 66, 68, 69, 70, 80].

### Task 2: Text Recognition (SVTR + CTC)
[cite_start]The recognition stage consumes cropped line images and predicts the character sequence[cite: 126, 175].

* **Pre-processing:**
    * [cite_start]Crops are converted to single-channel grayscale[cite: 119].
    * [cite_start]Aspect-ratio preserving resize is applied to fit within a target size of $H=32, W=256$[cite: 93, 94, 95].
    * [cite_start]Images are right-padded to a fixed $256 \times 32$ canvas and pixel values are normalized to approximately [-1, 1][cite: 103, 104, 106, 109].
* [cite_start]**Architecture:** * *Patch Embedding:* Compresses height and splits width into patches, creating a token sequence[cite: 178, 179].
    * [cite_start]*Positional Embedding:* Added to encode left-to-right reading order[cite: 181, 182].
    * [cite_start]*Transformer Encoder:* Six identical layers with multi-head self-attention to model long-range dependencies[cite: 16, 186].
    * [cite_start]*Linear Classifier:* Maps tokens to character logits including a CTC blank class[cite: 207].
* [cite_start]**Decoding:** Uses CTC loss for training and greedy decoding at inference to yield the predicted text string[cite: 197, 223].

---

##  Results & Evaluation

### Task 1: Text Detection Performance
[cite_start]After 50 epochs of training, the DBNet model showed stable convergence (Training Loss: 0.0281)[cite: 211, 212]. The performance metrics are:
* [cite_start]**Precision:** 79.1% [cite: 216, 217]
* [cite_start]**Recall:** 91.9% [cite: 218, 219]
* [cite_start]**H-Mean (F1-Score):** 85.0% [cite: 220, 221]

[cite_start]*Highlight:* The model significantly elevated the Recall score to 91.9%, preventing the omission of blurred or closely printed text compared to baseline approaches[cite: 229, 233].

### Task 2: Text Recognition Performance
[cite_start]Evaluated using the ICDAR-style exact line matching protocol (case-insensitive)[cite: 258].
* [cite_start]**Train Hmean:** 80.16% [cite: 230]
* [cite_start]**Validation Hmean:** 77.62% [cite: 230]
* [cite_start]**Test Hmean:** 75.76% [cite: 230]

[cite_start]The SVTR+CTC configuration reliably recognizes short to medium alphanumeric strings and typical receipt phrases, providing a robust foundation for subsequent information extraction[cite: 271, 273, 294].

---

##  References
* [cite_start][1] A. Graves et al., "Connectionist Temporal Classification...", ICML 2006. [cite: 277]
* [cite_start][2] B. Shi et al., "An End-to-End Trainable Neural Network for Image-based Sequence Recognition...", TPAMI 2017. [cite: 278]
* [cite_start][3] M. Liao et al., "Real-time Scene Text Detection with Differentiable Binarization," AAAI 2020. [cite: 280]
* [cite_start][4] S. Huang et al., "ICDAR 2019 Competition on Scanned Receipt OCR and Information Extraction," ICDAR 2019. [cite: 282]
* [cite_start][5] R. Zhang et al., "SVTR: Scene Text Recognition with a Single Visual Model," arXiv 2022. [cite: 283, 284]
* [cite_start][6] zzzDavid, "ICDAR-2019-SROIE", GitHub repository, 2019. [cite: 285]