# Multimedia Systems: Video Compression Fundamentals

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/matinapap/Multimedia-Systems/blob/main/Multimedia_Systems.ipynb)
![Python](https://img.shields.io/badge/Python-3.12-blue)
![OpenCV](https://img.shields.io/badge/OpenCV-cv2-green)

This project implements the main building blocks of **temporal video compression** from scratch in Python. It covers inter-frame prediction, entropy coding and motion compensation, then measures how much each technique lowers the bit rate of a real video sequence.

The pipeline follows the same ideas used in standards like MPEG and H.264, simplified so that every step is visible and measurable.

---

## What it does

| Part | Technique | Description |
|------|-----------|-------------|
| **A** | Frame differencing | Converts each frame to grayscale and computes the absolute difference between consecutive frames, `|Fₙ − Fₙ₋₁|`. Static regions become near-zero, which removes temporal redundancy. |
| **B** | Huffman coding | Builds a Huffman tree from the symbol frequencies of the difference images (min-heap based), assigns variable-length prefix codes and reports the compression ratio and average bits per symbol. |
| **C** | Motion compensation | Uses exhaustive **block matching** (SAD criterion, 32×32 blocks, ±8 px search window) to predict each frame from the previous one. The motion-compensated residual is then Huffman-coded and compared with Part B. |

## Results

Test video: 114 frames at 1920×1080, giving 113 difference frames.

| Method | Original size | Compressed size | Compression ratio | Avg. bits / symbol |
|--------|--------------:|----------------:|------------------:|-------------------:|
| Raw 8-bit grayscale | 1,874,534,400 bits | n/a | 1.00 | 8.00 |
| (B) Frame differences + Huffman | 1,874,534,400 bits | 690,429,588 bits | **2.72** | 2.95 |
| (C) Motion compensation + Huffman | 1,874,534,400 bits | 497,511,204 bits | **3.77** | 2.12 |

**Takeaway:** Motion compensation reduces the coded bitstream by about **28% compared with plain frame differencing**. Block matching predicts moving content much better, so the residual has lower entropy and Huffman coding works better on it.

## Project structure

```
.
├── Multimedia_Systems.ipynb   # Full implementation and results
├── requirements.txt           # Python dependencies
├── LICENSE                    # MIT License
└── README.md
```

Running the notebook produces:

```
results/
├── frame_diffs_A/        # PNG of every frame difference
├── huffman_B/            # Huffman codebook for (A) residuals
├── mc_diffs_C/           # PNG of every motion-compensated residual
└── huffman_C/            # Huffman codebook for (C) residuals
```

Each `huffman_codebook.txt` lists `symbol`, `frequency`, `code` and `code_length` for every pixel value.

## Getting started

### Option 1: Google Colab (recommended)

1. Click the **Open in Colab** badge above.
2. Upload your video to Google Drive as `MyDrive/input_video.avi`.
3. Run all cells. The notebook mounts Drive automatically.

### Option 2: Run locally

```bash
git clone https://github.com/matinapap/Multimedia-Systems.git
cd Multimedia-Systems
pip install -r requirements.txt jupyter
jupyter notebook Multimedia_Systems.ipynb
```

When running locally, the notebook detects that it isn't on Colab and reads `input_video.avi` from the project folder. Change `video_path` in the *Input video* cell to use a different file.

## Implementation notes

- **Huffman coding** is written from scratch with `heapq` and `collections.Counter`. No compression libraries are used. A unique ID breaks ties in the heap, so tree nodes never need to be compared directly.
- **Block matching** is a full search that evaluates all (2·8+1)² = 289 candidate positions per block. The search is accurate but slow. Faster searches such as three-step or diamond search would be natural extensions.
- Blocks that don't fit entirely inside the frame are not predicted, so their pixels pass through to the residual unchanged.
- The project was developed for a university Multimedia Systems course.

## Possible extensions

- Fast motion search (three-step search, diamond search)
- Encoding motion vectors and counting their bit cost in the total bitstream
- DCT and quantization of residuals (lossy, JPEG/MPEG-style)
- Reporting PSNR and plotting entropy per frame

## Tech stack

Python · NumPy · OpenCV · Matplotlib · Jupyter / Google Colab

## License

Released under the [MIT License](LICENSE).

## Author

**Matina Papadakou** · [GitHub](https://github.com/matinapap)
