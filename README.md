Explainable Protein Function Prediction using ESM-2 Embeddings

Predicting protein molecular functions directly from amino acid sequences using transfer learning from ESM-2, with a strong focus on explainability through attention visualization, integrated gradients, and systematic in silico mutagenesis.

Overview

This project addresses the protein annotation gap: millions of sequenced proteins lack functional characterization. Traditional homology-based approaches fail for proteins without close relatives, limiting large-scale functional inference. Here, we show that ESM-2, a protein language model pre-trained on 250 million sequences, encodes rich functional information that enables accurate Gene Ontology (GO) molecular function prediction from sequence alone.

Beyond predictive performance (8–23× improvement over random baselines), the main contribution of this work is interpretability. We investigate why predictions are made by analyzing which sequence regions and which embedding dimensions drive model decisions, and we validate these computational insights through targeted in silico mutagenesis experiments.

Key Results

Predictive performance

Test accuracy: 98.58% (inflated by class imbalance)

Mean PR-AUC: 0.131 (column-wise), 0.349 (row-wise)

8.4× improvement over random baselines (function-level)

23.0× improvement over random baselines (protein-level)

Strong performance for conserved families: GPCRs (PR-AUC 0.996), kinases (0.892)

Explainability insights

Function-specific embedding dimensions emerge without supervision

Kinases: dimension 493

Receptors: dimensions 532 / 565

Transcription factors: dimensions 350 / 385

Dimension 553 acts as a universal “functional protein” signal across families

Single and combinatorial mutations cause minimal prediction changes (<5%)

Deletion scanning identifies truly critical regions (e.g., kinase activation loop: −10.7%)

Biological validation

Catalytic proteins (kinases) exhibit localized fragility

Binding proteins (receptors) are globally robust through architectural redundancy

Results align with evolutionary principles of mutational buffering

Attention highlights structural landmarks; integrated gradients identify predictive features

Dataset and Availability

Data sources

Gene Ontology Consortium (human annotations)

UniProt human proteome

Dataset characteristics

8,704 proteins × 202 GO molecular function terms

Multi-label setting (3–6 functions per protein on average)

Proteins >500 amino acids excluded for computational reasons

Important note on data access

To comply with data usage policies and to keep the repository lightweight:

The dataset is not distributed in this repository

Pre-computed embeddings are not shared

All data loading, preprocessing, and embedding generation steps are fully implemented in code and can be reproduced by downloading the original public sources directly from Gene Ontology and UniProt

Methods (Summary)

Pipeline

Feature extraction using ESM-2 (facebook/esm2_t30_150M_UR50D)
Mean pooling → 640-dimensional protein embeddings

Multi-label classifier (640 → 512 → 256 → 202)

Batch normalization, ReLU, dropout (0.3), sigmoid outputs

Training with binary cross-entropy loss and early stopping

Explainability techniques

Attention visualization: averaged across layers and heads of ESM-2

Integrated gradients: attribution of predictions to embedding dimensions using Captum

In silico mutagenesis: alanine scanning, combinatorial mutations, and deletion scanning

Reproducibility and Notebooks

All analyses are implemented in a Jupyter notebook that contains the complete pipeline:

Data loading and preprocessing

ESM-2 embedding extraction

Classifier training and evaluation

Explainability analyses (attention, integrated gradients, mutagenesis)

Figures and visualizations

All figures shown in the technical report (attention profiles, integrated gradients, deletion scanning heatmaps) are generated programmatically within the notebook

No pre-rendered images are stored separately; running the notebook reproduces all results and plots

Installation
pip install torch transformers captum scikit-learn pandas numpy matplotlib seaborn biopython obonet


Tested on

Python 3.11

PyTorch 2.0+

transformers 4.35+

macOS (Apple Silicon, MPS) and Linux (CUDA)

Limitations

Single organism (human proteome only)

Strong class imbalance limits rare function prediction

Proteins >500 amino acids excluded

Single train/validation/test split

No explicit structural information (AlphaFold not integrated)

Future Work

Hierarchical GO-aware loss functions

Few-shot learning for rare functions

Cross-species generalization

Integration with AlphaFold2 structures

Scaling to larger protein language models

Variant effect prediction and protein engineering applications

Citation

If you use or build upon this work, please cite:

@misc{gonzalez2025explainable,
  title={Explainable Protein Function Prediction using ESM-2 Embeddings},
  author={Gonzalez, Daniela Alejandra},
  year={2025},
  institution={Northeastern University}
}

License

Code released under the MIT License.
Data usage is subject to Gene Ontology Consortium and UniProt terms.

Contact

Daniela Alejandra Gonzalez

GitHub: https://github.com/DaniAGonzalez

Email: gonzalez.daniel@northeastern.edu

LinkedIn: https://linkedin.com/in/danielaalejandra

For questions, feedback, or collaboration, feel free to open an issue or reach out directly.
