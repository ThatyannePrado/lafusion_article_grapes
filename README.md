# Counting Grape Bunches in Pergola Vineyards with Weightless Neural Networks

Code and experiments for the submitted paper evaluating weightless neural networks (WiSARD) for counting grape bunches in pergola vineyards, via patch classification followed by connected component analysis.

> Work under blind review. Repository kept private until submission is complete.

## Abstract

Counting grape bunches supports yield estimation in viticulture, a task traditionally manual and costly. This work evaluates WiSARD, a weightless neural network that classifies image patches, followed by connected component analysis that converts the patch map into a bunch count. The classifier trains in a single pass of about 30 seconds on a CPU. The paper describes the pipeline and reports an empirical study covering the choice of patch dimension, labeling threshold, binarization schemes, address size of the network, and a comparison against the ClusWiSARD variant. The selected configuration reaches a patch classification F1 of 0.598. On the test set, the counting pipeline achieves a mean absolute error of 14.69 bunches per image with a near-zero bias of −0.38 and an object-level detection rate of 50.2%.

## Pipeline

The method decomposes counting into five stages, grouped into three phases:

1. **Patch extraction and labeling** — a sliding window extracts patches from the images; a patch is labeled positive when the fraction of its area covered by annotated boxes reaches a positive threshold, negative below a negative threshold, and discarded as ambiguous in between.
2. **Binarization (HSV thermometer)** — each color channel is ordinally quantized into L=4 levels; the hue channel isolates the fruit's color, separated from saturation and brightness.
3. **WiSARD training** — classifier trained in a single pass (~30 s on CPU), address size N=32, HSV thermometer encoding.
4. **Grid inference** — the trained classifier is applied over the patch grid of an unseen image, producing a binary map of positive patches.
5. **Connected components and count** — connected components of the map, subject to a minimum size filter, are counted as bunches.

Stages 1 and 2 form the preprocessing phase, stage 3 the training phase, and stages 4 and 5 the application phase. Each stage produces an artifact that can be inspected in isolation.

## Repository structure

```
.
├── 02_extract_patches.ipynb      # patch extraction and labeling from YOLO annotations
├── 03_binarize_patches.ipynb     # comparison of binarization schemes and HSV thermometer encoding
├── 04_train_wisard.ipynb         # WiSARD/ClusWiSARD training and ablation (address size, patch dim, threshold)
├── 05_counting_components.ipynb  # grid inference, connected components, and counting evaluation
├── img/                           # paper figures
└── .gitignore
```

Notebooks are numbered by pipeline execution order (numbering starts at 02, inherited from the original course project's organization).

### Untracked folders

`binarized/`, `patches/`, and `results/` are pipeline outputs (extracted patches, binarized data, result artifacts) and are listed in `.gitignore` since they are large intermediate data, not source code. Reproduce them by running the notebooks in the order above over the dataset.

## Dataset

Images of pergola vineyards, uniform resolution of 640×640 pixels: 936 training, 88 validation, and 45 test images, with bounding box annotations (YOLO format). Thirty-six training images contain no bunches at all, providing pure negative material.

## Dataset Access

> **The dataset is not included in this repository.** Access can be requested from the authors of this paper, or from the authors of the original Sasse et al. work.

## Key results

| Selected configuration | Value |
|---|---|
| Encoding | HSV thermometer, L=4 levels |
| Patch dimension | 32×32 |
| Address size (N) | 32 |
| Patch classification F1 (validation) | 0.598 |
| Counting MAE (test) | 14.69 bunches/image |
| Bias (test) | −0.38 |
| RMSE (test) | 17.35 |
| Object-level detection rate (test) | 50.2% |

The paper details the address size sweep, the negative ClusWiSARD result (F1 of 0.597, a technical tie with standard WiSARD), and the geometric interaction between patch dimension and labeling threshold, which excludes entire object size classes from training when miscalibrated.

## Comparison with the literature

The work is compared against Sasse et al. (YOLOv8 + ByteTrack, detection F1 of 69.9%, mAP@50 of 75.1%), highlighting the trade-off between a deep detector (GPU, hours of training, black box) and a weightless network (CPU, ~30 s training, inspectable state), without the two paradigms' F1 numbers being directly comparable.

## Requirements

- Python 3.x
- WiSARD/ClusWiSARD (IAZero/develop fork — the PyPI version 1.6.3 has a different API and is not compatible with the notebooks)
- Standard image processing and data analysis libraries (OpenCV, NumPy, scikit-image)

## How to reproduce

```bash
# 1. Patch extraction and labeling
jupyter notebook 02_extract_patches.ipynb

# 2. Binarization (HSV thermometer)
jupyter notebook 03_binarize_patches.ipynb

# 3. WiSARD training
jupyter notebook 04_train_wisard.ipynb

# 4. Inference and counting via connected components
jupyter notebook 05_counting_components.ipynb
```

## Acknowledgements

The authors used an AI assistant to translate the text of this paper from Portuguese into English and as an aid in the development of the Python code for the experiments. All scientific decisions, the analysis of the results, and the final wording were reviewed and validated by the authors, who take full responsibility for the content.
