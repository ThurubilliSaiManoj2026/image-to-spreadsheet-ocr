# System Architecture: Image-to-Spreadsheet Web Application Using OCR

## 1. High-Level Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                     USER INTERFACE (Frontend)                   │
│              HTML/CSS/JavaScript - Web Browser                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ • Image Upload / Drag-Drop                              │   │
│  │ • Preview of uploaded image                             │   │
│  │ • Editable data grid (Handsontable)                     │   │
│  │ • Export options (CSV, Excel, JSON)                     │   │
│  └─────────────────────────────────────────────────────────┘   │
└──────────────────────────┬──────────────────────────────────────┘
                           │ HTTP/REST API
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                    BACKEND (Flask/FastAPI)                       │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ API Endpoints:                                          │   │
│  │ • POST /upload → Process image → Return JSON table     │   │
│  │ • POST /export → Convert table → Download file         │   │
│  │ • GET /status → Check processing status                │   │
│  └─────────────────────────────────────────────────────────┘   │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│            IMAGE PROCESSING PIPELINE (Python)                   │
│  ┌──────────────────────────────────────────────────────┐      │
│  │ 1. IMAGE PREPROCESSING                               │      │
│  │    • Read image (OpenCV)                              │      │
│  │    • Convert to grayscale                             │      │
│  │    • Denoise (Gaussian blur, morphology)              │      │
│  │    • Adaptive thresholding / Binarization             │      │
│  │    • Deskew (Hough lines rotation)                    │      │
│  └──────────────────────────────────────────────────────┘      │
│                           │                                     │
│  ┌──────────────────────────────────────────────────────┐      │
│  │ 2. TABLE DETECTION & STRUCTURE RECOGNITION            │      │
│  │    Option A: Classical (Default - Month 2)            │      │
│  │    • Hough line detection (horizontal + vertical)     │      │
│  │    • Line intersection → cell boundaries              │      │
│  │    • Extract table region (bounding box)              │      │
│  │                                                       │      │
│  │    Option B: Deep Learning (Month 4 - Comparison)     │      │
│  │    • TableNet: CNN-based table detection              │      │
│  │    • DETR: Transformer-based detection                │      │
│  │    • Load pre-trained weights, run inference          │      │
│  └──────────────────────────────────────────────────────┘      │
│                           │                                     │
│  ┌──────────────────────────────────────────────────────┐      │
│  │ 3. CELL SEGMENTATION                                  │      │
│  │    • Slice image into individual cells                │      │
│  │    • Store (row, col) mapping with cell images        │      │
│  │    • Handle merged cells (optional - Month 5)         │      │
│  └──────────────────────────────────────────────────────┘      │
│                           │                                     │
│  ┌──────────────────────────────────────────────────────┐      │
│  │ 4. OPTICAL CHARACTER RECOGNITION (OCR)                │      │
│  │    • Tesseract OCR (primary - Month 3)                │      │
│  │    • Extract text from each cell                      │      │
│  │    • Return confidence scores                         │      │
│  │    • Optional: Google Vision / Azure OCR (fallback)   │      │
│  │    • Post-processing: trim, spell-check               │      │
│  └──────────────────────────────────────────────────────┘      │
│                           │                                     │
│  ┌──────────────────────────────────────────────────────┐      │
│  │ 5. DATA RECONSTRUCTION & EXPORT                        │      │
│  │    • Build 2D array from OCR results                  │      │
│  │    • Generate JSON, CSV, Excel formats                │      │
│  │    • Return to backend for download                   │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Detailed Module Breakdown

### 2.1 Image Preprocessing Module (`src/preprocessing.py`)

**Inputs:** Image file path (JPG, PNG, PDF)

**Processing Steps:**
```python
1. Read image → cv2.imread()
2. Grayscale conversion → cv2.cvtColor()
3. Denoise:
   - Gaussian blur: cv2.GaussianBlur()
   - OR Bilateral filter: cv2.bilateralFilter()
4. Contrast enhancement:
   - CLAHE (Contrast Limited Adaptive Histogram Equalization)
   - cv2.createCLAHE()
5. Binarization:
   - Adaptive threshold: cv2.adaptiveThreshold()
   - OR Otsu's method: cv2.threshold()
6. Deskew (if rotated):
   - Hough lines: cv2.HoughLines()
   - Estimate rotation angle
   - Apply rotation: cv2.warpAffine()
```

**Outputs:** Preprocessed image (grayscale, binarized)

**Config:** Stored in `src/config.py` (blur kernel size, threshold values, etc.)

---

### 2.2 Table Detection Module (`src/table_detection.py`)

**Method A: Classical Approach (OpenCV)**

```python
1. Hough Line Detection:
   - cv2.HoughLines() on preprocessed image
   - Detect horizontal and vertical lines
   - Filter by line length (minimum threshold)

2. Line Intersection:
   - Find intersections of H-lines and V-lines
   - Grid of intersections = cell corners

3. Cell Extraction:
   - Use intersections to define cell boundaries
   - Return: list of (row, col, x1, y1, x2, y2) for each cell
   - Compute: table dimensions (n_rows, n_cols)
```

**Method B: Deep Learning (Month 4)**

```python
1. Load Model:
   - TableNet pre-trained weights
   - OR DETR with table-specific fine-tuning

2. Inference:
   - Pass preprocessed image to model
   - Get predicted table bounding box
   - Get cell predictions (if model supports)

3. Post-processing:
   - NMS (Non-Maximum Suppression) if needed
   - Convert model outputs to cell coordinates
```

**Outputs:** 
- Table region (x1, y1, x2, y2)
- Cell grid: 2D array of (row, col, bbox)
- Confidence scores (if DL model)

---

### 2.3 Cell Segmentation Module (`src/table_detection.py`)

```python
1. For each detected cell:
   - Extract ROI (Region of Interest) from original image
   - Apply minimal preprocessing (if needed)
   - Save or pass to OCR

2. Build 2D structure:
   - cells[row][col] = cell_image_data
   - Maintain row/col indices for reconstruction
```

---

### 2.4 OCR Module (`src/ocr_engine.py`)

**Tesseract OCR:**

```python
1. Initialize Tesseract:
   - pytesseract.image_to_string(image)
   - Config: PSM (Page Segmentation Mode) = 6 or 7 (single line)
   
2. For each cell:
   - Extract text: pytesseract.image_to_data()
   - Get confidence scores
   - Threshold low-confidence text (flag for review)

3. Post-processing:
   - Strip whitespace
   - Spell-check (optional)
   - Encode to UTF-8
```

**Cloud OCR (Optional Fallback):**

```python
- Google Vision API / Azure Computer Vision
- Use for comparison or low-confidence cells
```

**Outputs:**
```python
ocr_results = {
    (row, col): {
        'text': 'extracted_text',
        'confidence': 0.95,
        'flagged': False
    },
    ...
}
```

---

### 2.5 Data Reconstruction Module (`src/utils.py`)

```python
1. Convert OCR results to 2D table:
   table_data = [[ocr_results[(r,c)]['text'] for c in range(n_cols)] 
                 for r in range(n_rows)]

2. Generate formats:
   - JSON: {rows: n_rows, cols: n_cols, data: table_data}
   - CSV: Write using pandas.to_csv()
   - Excel: Write using openpyxl or pandas.to_excel()
```

---

## 3. Data Flow Diagram

```
User uploads image
        ↓
    ┌───────────────────────────────────────┐
    │ Preprocessing Pipeline                │
    │ (denoise, threshold, deskew)          │
    └─────────────┬───────────────────────┘
                  ↓
    ┌───────────────────────────────────────┐
    │ Table Detection                       │
    │ (Hough lines or Deep Learning)        │
    └─────────────┬───────────────────────┘
                  ↓
    ┌───────────────────────────────────────┐
    │ Cell Segmentation                     │
    │ (slice image into cells)              │
    └─────────────┬───────────────────────┘
                  ↓
    ┌───────────────────────────────────────┐
    │ OCR (Tesseract + confidence scoring)  │
    └─────────────┬───────────────────────┘
                  ↓
    ┌───────────────────────────────────────┐
    │ Data Reconstruction (2D array)        │
    └─────────────┬───────────────────────┘
                  ↓
    ┌───────────────────────────────────────┐
    │ Export (JSON → Frontend)              │
    └───────────────────────────────────────┘
                  ↓
    Web UI displays editable grid
    User can edit, review, export
```

---

## 4. Backend API Design

### Endpoints

**POST `/upload`**
```
Request: 
  - Content-Type: multipart/form-data
  - File: image (JPG, PNG, PDF)

Response (JSON):
{
  "success": true,
  "rows": 3,
  "cols": 4,
  "data": [
    ["S.No", "Name", "Roll No", "Signature"],
    ["1", "T. Sai Manoj", "22WU0104120", "Sai..."],
    ...
  ],
  "confidence_scores": {
    "(0,0)": 0.98,
    "(1,1)": 0.85,
    ...
  },
  "flagged_cells": [(1,1), (2,2)],  // Low confidence
  "processing_time": 3.5  // seconds
}
```

**POST `/export`**
```
Request (JSON):
{
  "format": "csv" | "excel" | "json",
  "data": [[edited_table_data]],
  "filename": "extracted_table"
}

Response:
  - File download (CSV/Excel/JSON)
```

**GET `/status`**
```
Response:
{
  "status": "processing" | "completed" | "error",
  "progress": 50,
  "message": "OCR in progress..."
}
```

---

## 5. Frontend Design

### Key Components

1. **Upload Section**
   - Drag-and-drop area
   - File input button
   - Image preview

2. **Processing Status**
   - Loading bar
   - Progress messages

3. **Results Display**
   - Editable table grid (Handsontable library)
   - Confidence heatmap (optional)
   - Cell highlighting for low-confidence

4. **Export Options**
   - Download as CSV
   - Download as Excel
   - Download as JSON
   - Copy to clipboard

---

## 6. Technology Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| **Frontend** | HTML5, CSS3, JavaScript | ES6+ |
| **UI Library** | Handsontable / Tabulator | Latest |
| **Backend** | Flask / FastAPI | 2.3+ |
| **Computer Vision** | OpenCV | 4.8+ |
| **OCR** | Tesseract | 5.0+ |
| **ML Framework** | PyTorch | 2.0+ |
| **Data Processing** | Pandas, NumPy | Latest |
| **Web Server** | Gunicorn (production) | - |
| **Containerization** | Docker (optional) | - |

---

## 7. Deployment Architecture

```
┌─────────────────────────┐
│   End User (Browser)    │
└────────────┬────────────┘
             │ HTTPS
             ▼
┌─────────────────────────┐
│   Nginx/Apache Proxy    │
│   (Reverse Proxy)       │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│   Gunicorn Workers      │
│   (Flask App)           │
│   • Worker 1            │
│   • Worker 2            │
│   • Worker 3            │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│   Shared File Storage   │
│   • Uploads folder      │
│   • Temp processing     │
│   • Results             │
└─────────────────────────┘
```

**For your 6-month project:** Start with simple Flask development server; optionally add Docker for final polish.

---

## 8. Database Schema (if needed)

**Note:** For this project, database is optional (stateless processing). If you add user accounts/history later:

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(50),
    email VARCHAR(100)
);

CREATE TABLE processing_jobs (
    job_id INT PRIMARY KEY,
    user_id INT,
    upload_timestamp DATETIME,
    image_filename VARCHAR(255),
    accuracy FLOAT,
    processing_time FLOAT,
    status VARCHAR(20),  -- 'completed', 'failed', etc.
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
```

**Current design:** No database (file-based, in-memory)

---

## 9. Error Handling & Validation

1. **Input Validation:**
   - File size limit (5 MB)
   - File format check (JPG, PNG, PDF only)
   - Image dimensions check

2. **Processing Errors:**
   - Table not detected → User-friendly error message
   - OCR failed for cell → Flag cell for manual review
   - Empty image → Error response

3. **Output Validation:**
   - Cell count mismatch → Log warning
   - Invalid characters → Replace with placeholder
   - Export format errors → Fallback to JSON

---

## 10. Performance Optimization

| Optimization | Method | Target |
|--------------|--------|--------|
| **Image loading** | Progressive JPEG, thumbnails | Faster preview |
| **OCR** | GPU acceleration (if available) | <2 sec/table |
| **Table detection** | Model quantization, caching | <1 sec/table |
| **Frontend** | Lazy loading, async upload | Responsive UI |
| **Caching** | Redis (optional) | Reduce reprocessing |

---

## 11. Monitoring & Logging

```python
# Log all operations
import logging

logging.basicConfig(
    filename='app.log',
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

# Track:
# - Processing time per module
# - OCR accuracy
# - Error rates
# - API response times
```

---

## 12. Future Extensions (Out of scope for initial project)

1. Batch processing (multiple images)
2. User accounts & history
3. Advanced cell detection (merged cells)
4. Multi-language support
5. Real-time preview
6. Mobile app
7. Integration with Google Sheets / Microsoft Excel Online