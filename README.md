# Handwritten Answer Sheet Auto Grader

An AI-powered system that automatically grades handwritten student answer sheets by extracting handwriting via OCR and semantically comparing it against a reference answer key — no manual checking required.

## Overview

This project automates the grading of handwritten exams. It takes a photo of a student's handwritten answer sheet, the question paper (PDF), and a reference/model answer sheet (PDF), then:

1. Extracts question numbers and maximum marks from the question paper
2. Extracts and segments the model (reference) answers per question
3. Uses handwriting OCR to convert the student's handwritten answers into text
4. Splits the recognized text by question number
5. Computes semantic similarity between each student answer and its reference answer
6. Converts similarity scores into awarded marks
7. Generates a final grading report (CSV) with per-question scores and total marks

## Features

- **Handwriting OCR** using Microsoft's TrOCR (`trocr-base-handwritten`) for recognizing handwritten text from scanned/photographed answer sheets
- **Image preprocessing** with OpenCV (adaptive thresholding, morphological operations) to clean up uneven lighting and segment text into lines
- **PDF text extraction** using `pdfplumber` for question papers and reference answer keys
- **Automatic question/marks parsing** from the question paper using regex patterns
- **Semantic answer grading** using Sentence Transformers (`all-MiniLM-L6-v2`) and cosine similarity, instead of exact keyword matching
- **Custom similarity-to-marks scoring curve** with a penalty zone for partially correct answers
- **CSV grading report** with per-question breakdown and total score, auto-downloaded

## Tech Stack

- Python
- [Transformers](https://huggingface.co/docs/transformers) (TrOCR — `microsoft/trocr-base-handwritten`)
- [Sentence-Transformers](https://www.sbert.net/) (`all-MiniLM-L6-v2`)
- OpenCV (`opencv-python-headless`)
- PyTorch
- pdfplumber
- pandas
- Google Colab (`google.colab.files` for uploads/downloads)

## How It Works

### 1. Input
- A photo/scan of the student's handwritten answer sheet (image)
- The question paper (PDF)
- The reference/model answer sheet (PDF)

### 2. Question & Marks Extraction
Regex-based parsing extracts question numbers, question text, and max marks from the question paper.

### 3. Reference Answer Segmentation
The reference PDF text is split into per-question answers using the extracted question numbers.

### 4. Handwriting Recognition
The answer sheet image is preprocessed (grayscale → adaptive threshold → morphological closing) and segmented into individual lines, which are then passed through TrOCR to generate text, with post-processing to clean up common OCR artifacts.

### 5. Student Answer Segmentation
The reconstructed OCR text is split by question number, mirroring the reference answer segmentation.

### 6. Semantic Grading
Each student answer is compared to its corresponding reference answer using sentence embeddings and cosine similarity. A custom scoring function converts the similarity score into awarded marks:
- Below 0.5 similarity → 0 marks
- Above 0.88 similarity → full marks
- In between → scaled/penalized proportionally

### 7. Output
A `grading_report.csv` file containing:
- Question number
- Student's extracted answer
- Reference answer
- Similarity score
- Marks awarded / max marks
- Total score summary

## Usage

This notebook is designed to run in **Google Colab** (uses `google.colab.files` for file uploads/downloads).

1. Open the notebook in Google Colab
2. Run the setup cell to install dependencies
3. Upload the student's handwritten answer sheet (image) when prompted
4. Upload the question paper (PDF) when prompted
5. Upload the reference/model answer sheet (PDF) when prompted
6. Run the remaining cells sequentially
7. Download the generated `grading_report.csv` with the final grades

## Limitations / Notes

- Grading accuracy depends heavily on OCR quality, which can be affected by handwriting legibility and image quality
- Question numbering format in the question paper/reference sheet must follow a consistent pattern for parsing to work correctly
- Semantic similarity grading rewards conceptual overlap and doesn't verify factual correctness in the way a human grader might
- A GPU runtime (e.g., Colab GPU) is recommended for faster OCR and embedding inference

## Future Improvements

- Support for multi-page answer sheets
- Better handling of diagrams/equations in handwritten answers
- Configurable scoring thresholds per subject/question type
- Web-based interface instead of Colab notebook workflow
