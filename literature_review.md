# Literature Review: Image-to-Spreadsheet OCR Project

## 1. Table Detection and Structure Recognition

### Key Papers

**1.1 Deep Learning for Table Detection and Structure Recognition**
- **Citation:** Prasad et al., "Deep Learning for Detection and Structure Recognition of Tables in Document Images"
- **Key Contribution:** Introduces deep learning approaches for end-to-end table understanding
- **Relevance:** Directly applicable to your table detection module
- **Techniques:** CNNs for table localization, RNNs for structure recognition

**1.2 TableNet: Deep Learning Model for End-to-End Table Detection and Tabular Data Extraction**
- **Source:** arXiv:2105.11021
- **Key Contribution:** Lightweight deep learning model specifically designed for table extraction
- **Relevance:** Excellent candidate for your model comparison (vs. classical methods)
- **Performance:** Achieves 85%+ accuracy on standard benchmarks

**1.3 Multi-Type-TD-TSR: Extracting Tables from Document Images**
- **Citation:** ArXiv:2105.11021
- **Key Contribution:** Handles multiple table types and complex structures
- **Relevance:** Useful for understanding table diversity and complexity

**1.4 TransTab: A Transformer-based Approach for Table Detection and Extraction**
- **Key Contribution:** Uses Vision Transformers (ViT) and DETR for table detection
- **Relevance:** State-of-the-art approach; good for Month 4 model comparison
- **Advantage:** Better at handling irregular layouts and merged cells

---

## 2. Table Structure Recognition and Cell Segmentation

**2.1 Detection of Tables in Scanned Document Images**
- **Citation:** Kieninger & Dengel (2001)
- **Key Contribution:** Classic approach using Hough lines and morphological operations
- **Relevance:** Foundation for classical OpenCV-based table detection
- **Technique:** Line detection + intersection finding = cell boundaries

**2.2 Image-based Table Recognition: Data, Model, and Evaluation**
- **Citation:** Zhong et al., Semantic Scholar
- **Key Contribution:** Comprehensive survey of image-based table recognition approaches
- **Relevance:** Benchmark datasets (ICDAR, Marmot) and evaluation metrics
- **Datasets:** Marmot dataset, ICDAR 2013 table competition dataset

---

## 3. OCR and Text Recognition

**3.1 Tesseract OCR Engine**
- **Type:** Open-source, widely used OCR engine
- **Accuracy:** 85–92% on clean printed text
- **Relevant Paper:** Ray Smith, "An Overview of the Tesseract OCR Engine"
- **Relevance:** Primary OCR tool for your project; provides confidence scores

**3.2 Table Extraction using LLMs: Unlocking Structured Data**
- **Citation:** Nanonets Blog (2025)
- **Key Contribution:** Using Large Language Models (LLMs) for post-OCR correction
- **Relevance:** Optional enhancement for Month 5 (confidence filtering + spell-check)

**3.3 PdfTable: A Unified Toolkit for Deep Learning-Based Table Extraction**
- **Citation:** ArXiv:2409.05125
- **Key Contribution:** Comprehensive toolkit combining multiple OCR and table-detection models
- **Relevance:** Good reference for comparing different OCR engines and selection strategies

---

## 4. Image Preprocessing and Document Enhancement

**4.1 Document Image Binarization: A Comprehensive Review**
- **Technique:** Adaptive thresholding, Otsu's method, Niblack's algorithm
- **Relevance:** Critical for improving OCR accuracy on scanned/handwritten tables
- **Paper Reference:** Sezgin & Sankur, IEEE Transactions on Image Processing (2004)

**4.2 Deskewing and Perspective Correction**
- **Techniques:** Hough line detection, rotation estimation, projective transformation
- **Relevance:** Handles skewed/tilted table images from photos
- **OpenCV Functions:** `cv2.HoughLines()`, `cv2.getPerspectiveTransform()`

**4.3 Denoising Techniques: Survey and Comparison**
- **Methods:** Gaussian blur, Bilateral filter, Morphological operations
- **Relevance:** Improves table detection and OCR accuracy on noisy images
- **Trade-off:** Speed vs. quality (important for your <5 second target)

---

## 5. End-to-End Systems and Benchmarks

**5.1 End-to-End Table Structure Recognition and Extraction in Heterogeneous Documents**
- **Citation:** ArXiv:2105.11021
- **Key Contribution:** Full pipeline from image to structured table data
- **Relevance:** Directly mirrors your project architecture

**5.2 OCR Table Extraction Using Deep Learning**
- **Citation:** Acodis Blog (2022)
- **Key Points:**
  - Comparison of classical vs. deep learning approaches
  - Accuracy vs. speed trade-offs
  - Ensemble methods for improved performance

**5.3 Table Extraction with Table Data Using VGG-19 Deep Learning Model**
- **Citation:** PMC NCBI (2024)
- **Key Contribution:** Using pre-trained CNN architectures for table detection
- **Relevance:** Alternative to TableNet; good for model comparison

---

## 6. Datasets and Benchmarks

| Dataset | Size | Use Case | Relevance |
|---------|------|----------|-----------|
| **Marmot** | 2,417 images | Table detection benchmark | Primary benchmark |
| **ICDAR 2013** | 150 documents | Table detection competition | Evaluation reference |
| **TableBank** | 417K tables | Large-scale table understanding | Pre-training resource |
| **DocVQA** | 50K documents | Document understanding | Complex layouts |
| **Your custom dataset** | 300+ images | Real-world mixed data | For your project |

---

## 7. Performance Metrics and Evaluation

**Key Metrics:**

1. **Table Detection:**
   - Precision, Recall, F1-score
   - Intersection over Union (IoU)

2. **Cell Segmentation:**
   - Cell detection accuracy
   - False positive/negative rates

3. **OCR Accuracy:**
   - Character Error Rate (CER)
   - Word Error Rate (WER)
   - Cell-level accuracy (% correct cells)

4. **End-to-End Performance:**
   - Table extraction success rate
   - Processing time per image
   - End-to-end accuracy (complete tables correctly extracted)

---

## 8. Related Work and Competing Approaches

**Commercial Solutions:**
- ExtractTable (Cloud-based)
- Amazon Textract (AWS service)
- Google Cloud Vision API
- Microsoft Form Recognizer

**Open-Source Tools:**
- img2table (Python library)
- TabNet (GitHub)
- Camelot-py (PDF table extraction)

**Your Competitive Advantage:**
- Custom fine-tuning on your dataset
- Multiple model comparison and analysis
- Full-stack web application (not just API)
- Transparent evaluation and benchmarking

---

## 9. Key Takeaways for Your Project

1. **Table detection:** Classical (Hough lines) vs. deep learning (TableNet, DETR) both viable
2. **OCR:** Tesseract is mature; cloud APIs for comparison/fallback
3. **Preprocessing:** Critical for accuracy; don't skip this step
4. **Datasets:** ICDAR and Marmot for benchmarking; create custom dataset for real-world cases
5. **Metrics:** Measure accuracy at multiple levels (detection, segmentation, OCR, end-to-end)
6. **Ensemble methods:** Combining multiple models often improves accuracy

---

## 10. Research Gaps Your Project Addresses

1. **Limited real-world evaluation:** Most papers use clean dataset; you'll test on diverse images
2. **Handwriting support:** Most focus on printed text; you include handwritten tables
3. **Web accessibility:** Academic work is CLI-based; you build a user-friendly web app
4. **Model transparency:** You compare multiple approaches instead of one black-box solution

---

## References

- Prasad, D., et al. (2020). "Deep Learning for Detection and Structure Recognition of Tables in Document Images"
- Ray Smith. (2007). "An Overview of the Tesseract OCR Engine"
- Sezgin, M., & Sankur, B. (2004). "Document Image Binarization: A Comprehensive Review"
- ArXiv:2105.11021 – TableNet and Multi-Type-TD-TSR
- ArXiv:2409.05125 – PdfTable Toolkit
- ICDAR 2013 Table Detection Competition
- Marmot Dataset Documentation

---

## Suggested Reading Order

**Week 1 (Foundation):**
1. Prasad et al. – Deep Learning for Table Detection
2. Ray Smith – Tesseract OCR Overview

**Week 2 (Deep Learning):**
3. TableNet (ArXiv:2105.11021)
4. TransTab – Transformer-based approach

**Week 3 (Preprocessing & OCR):**
5. Sezgin & Sankur – Binarization
6. Nanonets – LLM-based OCR correction

**Week 4 (Datasets & Evaluation):**
7. Image-based Table Recognition (Zhong et al.)
8. End-to-End Table Extraction (ArXiv)
9. PdfTable Toolkit documentation