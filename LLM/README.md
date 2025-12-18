# Explainable Protein Function Prediction using ESM-2 Embeddings

Predicting protein molecular functions from amino acid sequences using transfer learning from ESM-2, with comprehensive explainability analysis through attention visualization, integrated gradients, and systematic mutagenesis experiments.

## Overview

This project addresses the protein annotation gap: millions of sequenced proteins lack functional characterization. Traditional homology-based methods fail for proteins without known relatives. We demonstrate that ESM-2, a large protein language model pre-trained on 250 million sequences, encodes functional information that enables accurate prediction of Gene Ontology molecular functions.

Beyond prediction accuracy (8-23× improvement over baselines), this work focuses on explainability: understanding which sequence features and embedding dimensions drive predictions, and validating these computational insights through in silico mutagenesis.

## Key Results

Predictive Performance:
- Test accuracy: 98.58%
- Mean PR-AUC: 0.131 (column-wise), 0.349 (row-wise)
- 8.4× improvement over random baselines (function-level)
- 23.0× improvement over random baselines (protein-level)
- Exceptional performance for conserved motifs: GPCRs (PR-AUC 0.996), kinases (0.892)

Explainability Insights:
- Function-specific embedding dimensions: kinases use dimension 493, receptors use dimensions 532/565, transcription factors use 350/385
- Dimension 553 acts as universal "functional protein detector" across all families
- Model correctly learned biological robustness: single mutations cause minimal prediction changes (<5%)
- Deletion scanning identifies critical regions: kinase activation loop deletion causes -10.7% prediction drop

Biological Validation:
- Catalytic proteins (kinases) have fragile functional loops
- Binding proteins (receptors) are globally robust through architectural redundancy
- Results consistent with evolutionary principles: proteins buffer against genetic variation
- Attention identifies structural landmarks; integrated gradients reveals predictive features

## Repository Structure
```
.
├── _LLM_proteins.ipynb              # Main analysis notebook (complete pipeline)
├── embeddings_train.pkl            # Pre-computed ESM-2 embeddings (training)
├── embeddings_valid.pkl            # Pre-computed ESM-2 embeddings (validation)
├── embeddings_test.pkl             # Pre-computed ESM-2 embeddings (test)
├── best_model_batch_norm.pth       # Trained classifier checkpoint
├── figures/                        # Generated visualizations
│   ├── attention_*.png
│   ├── deletion_scanning_*.png
│   └── integrated_gradients_*.png
└── README.md                       # This file
```

## Dataset

Source: Gene Ontology Consortium + UniProt human proteome

Statistics:
- 8,704 proteins × 202 GO molecular function terms
- Training: 5,222 proteins | Validation: 1,741 | Test: 1,741
- Multi-label: proteins average 3-6 functions each

Filtering rationale:
- Molecular Function Ontology (MFO) only
- Removed generic terms: "binding", "molecular_function", "protein binding"
- Minimum 50 training examples per GO term (addresses class imbalance)
- Maximum 500 amino acids (computational constraint for ESM-2)

## Methods

Pipeline:
1. Embedding extraction: ESM-2 (facebook/esm2_t30_150M_UR50D) with mean pooling → 640D vectors
2. Multi-label classifier: Feed-forward network (640→512→256→202)
3. Architecture: Batch normalization + ReLU + Dropout (0.3) + Sigmoid output
4. Training: Binary cross-entropy loss, Adam optimizer (lr=0.001), early stopping

Explainability techniques:
- Attention visualization: Extract ESM-2 attention weights (30 layers × 20 heads averaged)
- Integrated gradients: Attribute predictions to embedding dimensions using Captum
- In silico mutagenesis: Test prediction robustness through alanine scanning, combinatorial mutations, and deletion scanning

## Installation

Requirements:
```bash
pip install torch transformers captum scikit-learn pandas numpy matplotlib seaborn biopython obonet
```

Tested on:
- Python 3.11
- PyTorch 2.0+
- transformers 4.35+
- macOS with Apple Silicon (MPS) / Linux with CUDA

## Usage

Quick start:
```python
# Load pre-computed embeddings and trained model
import pickle
import torch

# Load embeddings
with open('embeddings_test.pkl', 'rb') as f:
    test_data = pickle.load(f)

# Load classifier
from model import ProteinClassifier
model = ProteinClassifier(640, [512, 256], 202, dropout=0.3, use_batch_norm=True)
model.load_state_dict(torch.load('best_model_batch_norm.pth'))
model.eval()

# Predict functions for a protein
protein_embedding = test_data.loc[protein_id, 'embeddings']
predictions = model(torch.FloatTensor(protein_embedding).unsqueeze(0))
# predictions: [202] probabilities for each GO term
```

Complete pipeline:
```bash
# Open Jupyter notebook
jupyter notebook LLM_proteins.ipynb

# Follow sections sequentially:
# 1. Data loading and preprocessing
# 2. ESM-2 embedding extraction (takes 2-3 hours)
# 3. Classifier training and hyperparameter tuning
# 4. Evaluation (column-wise and row-wise)
# 5. Explainability analysis (attention, IG, mutagenesis)
```

For detailed visualizations and comprehensive analysis, see the complete technical report in `LLM_proteins.ipynb`. The notebook contains:
- Interactive attention heatmaps for top-performing and worst-performing proteins
- Integrated gradients analysis showing function-specific embedding dimensions
- Deletion scanning results with publication-quality figures
- Comparative analysis across protein families (kinases, receptors, transcription factors)

## Key Findings

Model learns function-specific representations:
- Different protein families activate distinct embedding dimensions
- Dimension 493 critical for kinases (ATP-binding loop encoding)
- Dimensions 532/565 critical for receptors (transmembrane architecture)
- Dimension 350/385 critical for DNA-binding transcription factors

Biological robustness correctly captured:
- Single point mutations cause <5% prediction change
- Combinatorial mutations (2-5 residues) still minimal effect
- Deletion scanning reveals truly critical regions (kinase loops: -10.7%)
- Model learned that most proteins tolerate mutation (evolutionary buffering)

Explainability reveals interpretability:
- Attention identifies structural landmarks (domain boundaries, conserved regions)
- Integrated gradients identifies predictive features (embedding dimensions)
- Mutagenesis validates functional importance (causal testing)
- Framework distinguishes computational focus from functional criticality

## Limitations

- Single organism (human only) - generalization to other species not tested
- Class imbalance: 90% of GO terms have <50 examples, limiting rare function predictions
- Sequence length restriction: proteins >500 amino acids excluded (15% of proteome)
- Single train/validation/test split - cross-validation would provide more robust estimates
- No structural information - incorporating AlphaFold predictions could improve performance

## Future Work

- Hierarchical loss functions exploiting GO parent-child relationships
- Few-shot learning for rare functions
- Multi-task learning across MFO, BPO, and CCO ontologies
- Integration with AlphaFold2 structural predictions
- Scaling to larger ESM-2 models (650M, 3B parameters)
- Active learning for experimental prioritization

## Citation

If you use this work, please cite:
```
@misc{gonzalez2025explainable,
  title={Explainable Protein Function Prediction using ESM-2 Embeddings},
  author={Gonzalez, Daniela Alejandra},
  year={2025},
  institution={Northeastern University},
}
```

## Related Work

- ESM-2: Lin et al. "Language models of protein sequences at the scale of evolution" (Science 2023)
- DeepGO: Kulmanov et al. "DeepGO: predicting protein functions from sequence and interactions using a deep ontology-aware classifier" (Nature Methods 2018)
- DeepFRI: Gligorijević et al. "Structure-based protein function prediction using graph convolutional networks" (Nature Communications 2021)
- Integrated Gradients: Sundararajan et al. "Axiomatic attribution for deep networks" (ICML 2017)


## License

Code released under MIT License. Data usage subject to Gene Ontology Consortium and UniProt terms.

## Contact

Daniela Alejandra Gonzalez
- GitHub: [DaniAGonzalez](https://github.com/DaniAGonzalez)
- Email: gonzalez.daniel@northeastern.edu
- LinkedIn: [linkedin.com/in/danielaalejandra](https://linkedin.com/in/danielaalejandra)

For questions about methods, results, or collaboration opportunities, please open an issue or contact directly.
