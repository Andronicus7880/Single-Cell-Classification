# Overview
This notebook is a consolidated version of three separate notebooks originally intended for a BioHack event. It streamlines the scRNA-seq analysis pipeline into a single, Colab-friendly environment, addressing common issues like runtime resets and package reinstallation. The primary goal is to demonstrate and compare traditional scRNA-seq analysis methods with the application of a pretrained foundation model (Geneformer) for cell type classification, particularly in few-shot learning scenarios.

# Key areas explored:

Part 1: Data Ingest & Quality Control (QC): Processing raw scRNA-seq counts and filtering out low-quality cells.
Part 2: Classical Clustering & Cell Type Annotation: Applying traditional methods like PCA, UMAP, Leiden clustering, and manual marker-gene-based annotation.
Part 3: A Single-Cell Foundation Model for Cell Type Classification: Utilizing the Geneformer model to extract cell embeddings and evaluating its performance against classical features in a few-shot classification task.



# Setup and Prerequisites
To run this notebook successfully in Google Colab, please follow these steps:

Pin the runtime to Python 3.12: Colab's default runtime has moved to Python 3.13, but the pyproject.toml for this repository requires Python >=3.10,<3.13. To change the runtime:

Go to Runtime -> Change runtime type.
Select Python 3.12 (Fallback / Previous runtime) under Python version.
Click Save.
You will need to reconnect the runtime.
Clone the Repository and Install Dependencies: The notebook will automatically clone the biohack-2026 repository and install its pinned Python dependencies, including scanpy, torch, transformers, and numpy<2. The specific NumPy version is crucial for ABI compatibility with torch==2.2.x.

Download Geneformer Checkpoint: The necessary Geneformer-V1-10M checkpoint (~50MB) will be downloaded as part of the setup script (scripts/setup_geneformer.sh).

Restart the Runtime: After installing dependencies, the notebook will intentionally kill and restart the Colab kernel. This is essential because Colab's base image ships with NumPy 2.x, and installing NumPy 1.x (required by this project) requires a clean kernel restart to avoid ValueError: numpy.dtype size changed errors. After the restart, ignore any "session crashed" notices and continue running the cells from where you left off (i.e., after the restart cell).



## Workflow Details

### Part 1: Data Ingest & Quality Control

This section focuses on preparing the raw scRNA-seq data. It involves:

*   **Loading data**: Reading raw (unfiltered, unnormalized) counts into an `AnnData` object.
*   **Computing QC metrics**: Calculating `total_counts`, `n_genes_by_counts`, and `pct_counts_mt` to assess cell quality.
*   **Filtering low-quality cells**: Removing cells based on thresholds derived from the QC metrics (e.g., too few/many genes, high mitochondrial content). The filtered data (`pbmc3k_qc.h5ad`) is saved for downstream analysis.

### Part 2: Classical Clustering & Cell Type Annotation

This part implements the standard "classical" scRNA-seq analysis pipeline:

*   **Normalization and Transformation**: Normalizing total counts per cell and log-transforming the data.
*   **Highly Variable Genes (HVGs)**: Identifying genes that show significant variability across cells to focus downstream analysis.
*   **Dimensionality Reduction**: Performing Principal Component Analysis (PCA) and Uniform Manifold Approximation and Projection (UMAP).
*   **Leiden Clustering**: Grouping cells into clusters based on their proximity in the reduced dimension space.
*   **Marker Gene Identification**: Ranking genes to find those most differentially expressed per cluster.
*   **Manual Annotation**: Assigning biological cell type labels to clusters using known marker genes. The resulting cell type labels and PCA features are saved for comparison in Part 3 (`pbmc3k_cell_type_labels.csv`, `pbmc3k_pca_features.csv`).

### Part 3: A Single-Cell Foundation Model for Cell Type Classification

This section explores the utility of a pretrained single-cell foundation model (Geneformer) for cell type classification, comparing its performance against classical methods. It includes:

*   **Data Preparation**: Loading raw, unnormalized counts and cell type labels. Subsampling cells per type for manageability.
*   **Gene Mapping**: Translating gene symbols to Ensembl IDs using Geneformer's internal dictionary.
*   **Tokenization**: Converting gene expression profiles into token sequences using `TranscriptomeTokenizer`, which performs rank-based normalization.
*   **Embedding Extraction**: Loading the Geneformer model and extracting cell embeddings by mean-pooling the last hidden layer across each cell's token sequence. Special attention is given to device-agnostic execution and efficient batching of variable-length sequences.
*   **Visualization**: UMAP visualization of Geneformer embeddings to qualitatively assess cell type separation.
*   **Few-shot Classification**: A quantitative comparison of cell type classification accuracy using a simple logistic regression model. This compares performance when training on a limited number of labeled examples (few-shot learning) for both classical PCA features and Geneformer embeddings. This highlights the foundation model's advantage when labeled data is scarce.





## Key Findings

The few-shot classification results demonstrate a significant advantage for Geneformer embeddings, particularly when labeled data is scarce. In the provided example, Geneformer embeddings achieved high accuracy with a single labeled cell per type, whereas classical PCA features required 10-20 labeled cells to reach comparable performance. This underscores the value of pretrained foundation models in biological contexts where expert-annotated data is expensive and limited.

As the number of labeled examples increases, the performance gap between the two methods tends to shrink, indicating that foundation models are most valuable when labels are scarce rather than being a universal replacement for classical analysis.



## Usage
To run this notebook:

Open the notebook in Google Colab.
Follow the "Setup and Prerequisites" instructions to ensure the correct Python runtime and installed dependencies.
Run all cells sequentially. The notebook is designed to be executed from top to bottom.



## Data
The primary dataset used is a preprocessed version of PBMC 3k single-cell RNA-seq data, specifically pbmc3k_qc.h5ad, which has undergone initial quality control. Additional intermediate files such as pbmc3k_cell_type_labels.csv and pbmc3k_pca_features.csv are generated and used across the different parts of the notebook.




## Acknowledgements
This notebook is part of the BioHack 2026 initiative, exploring advanced methods in bioinformatics. It utilizes the Geneformer model and various tools from the single-cell community, including scanpy, numpy, pandas, torch, transformers, and matplotlib.


## BioHack 2026 — Combined Colab Notebook
This is the single-notebook, Colab-friendly version of this repo's three notebooks (01_data_ingest_and_qc.ipynb, 02_classical_clustering_and_annotation.ipynb, 03_geneformer_foundation_model.ipynb), concatenated in order:

Part 1 — Data Ingest & Quality Control
Part 2 — Classical Clustering & Cell Type Annotation
Part 3 — A Single-Cell Foundation Model for Cell Type Classification
Opening the three original notebooks separately in Colab each spins up its own fresh runtime, so Part 2 can't find the data Part 1 wrote, packages have to be reinstalled, etc. This notebook exists so you can open one file and run straight through — everything below (repo clone, install, Geneformer checkpoint download, kernel restart) only needs to happen once, right here, at the top.






