# Interpretability of Protein Function Prediction using ESM-2 Embeddings
 
Predicting protein molecular functions directly from amino acid sequences using transfer learning from ESM-2, with a strong emphasis on interpretability through attention visualization, integrated gradients, and systematic in silico mutagenesis.
 
# Overview
 
This project addresses the protein annotation gap: millions of sequenced proteins lack functional characterization. Traditional homology-based approaches often fail for proteins without close relatives, limiting large-scale functional inference. Here, we demonstrate that ESM-2, a protein language model pre-trained on 250 million sequences, encodes rich functional information that enables accurate Gene Ontology (GO) molecular function prediction from sequence alone.
 
Beyond predictive performance (8–23× improvement over random baselines), the central contribution of this work is interpretability. We investigate why predictions are made by analyzing which sequence regions and which embedding dimensions drive model decisions, and we validate these computational insights through targeted in silico mutagenesis experiments.
 
# Key Results
## Predictive Performance
 
Test accuracy: 98.58% (inflated due to class imbalance)
 
Mean PR-AUC:
 
- 0.131 (column-wise, function-level)
- 0.349 (row-wise, protein-level)
- 8.4× improvement over random baselines at the function level
- 23.0× improvement over random baselines at the protein level
## Strong performance for conserved protein families:
 
- GPCRs: PR-AUC 0.996
- Protein kinases: PR-AUC 0.892
## Interpretability Insights
 
### Function-specific embedding dimensions emerge without explicit supervision:
 
- Kinases rely on dimension 493
- Receptors rely on dimensions 532 and 565
- Transcription factors rely on dimensions 350 and 385
- Dimension 553 acts as a universal "functional protein" signal across all protein families
- Single and combinatorial mutations cause minimal prediction changes (<5%)
- Deletion scanning reveals truly critical regions: deletion of the kinase activation loop causes a −10.7% prediction drop
## Biological Validation
 
- Catalytic proteins (e.g. kinases) exhibit localized functional fragility
- Binding proteins (e.g. receptors) are globally robust due to architectural redundancy
- Observed robustness is consistent with evolutionary principles of mutational buffering
- Attention highlights structural landmarks, while integrated gradients identify the embedding dimensions driving predictions
## Dataset and Availability
### Data Sources
 
- Gene Ontology Consortium (human annotations)
- UniProt human proteome
### Dataset Characteristics
 
- 8,704 proteins × 202 GO molecular function terms
- Multi-label setting (3–6 functions per protein on average)
- Proteins longer than 500 amino acids excluded for computational reasons
### Important Note on Data Access
 
To comply with data usage policies and keep the repository lightweight:
 
- The dataset is not distributed in this repository
- Pre-computed embeddings are not shared
- All data loading, preprocessing, and embedding generation steps are fully implemented in code. Public data are downloaded directly from the Gene Ontology Consortium and UniProt within the notebook
## Methods Summary
### Modeling Pipeline
 
- Feature extraction using ESM-2 (facebook/esm2_t30_150M_UR50D)
- Mean pooling yields 640-dimensional protein embeddings
- Multi-label classifier architecture: 640 → 512 → 256 → 202
- Batch normalization, ReLU activations, dropout (0.3), sigmoid outputs
- Training with binary cross-entropy loss and early stopping
### Interpretability Techniques
 
Attention visualization using ESM-2 attention weights averaged across layers and heads
 
Integrated gradients for attribution of predictions to embedding dimensions (Captum)
 
#### In silico mutagenesis:
 
- Alanine scanning
- Combinatorial mutations
- Deletion scanning
### Reproducibility and Notebooks
 
All analyses are implemented in a single Jupyter notebook that contains the complete pipeline:
 
- Data loading and preprocessing
- ESM-2 embedding extraction
- Classifier training and evaluation
- Interpretability analyses (attention, integrated gradients, mutagenesis)
- Figures and Visualizations: All figures shown in the technical report—including attention profiles, integrated gradients analyses, and deletion scanning heatmaps—are generated programmatically within the notebook. No pre-rendered images are stored separately; running the notebook reproduces all figures and results.
## Installation
pip install torch transformers captum scikit-learn pandas numpy matplotlib seaborn biopython obonet
 
## Tested On
 
Python 3.11
 
PyTorch 2.0+
 
transformers 4.35+
 
macOS (Apple Silicon, MPS) and Linux (CUDA)
 
## Limitations
 
Single organism analysis (human proteome only)
 
Strong class imbalance limits rare function prediction
 
Proteins longer than 500 amino acids excluded
 
Single train/validation/test split
 
No explicit structural information (e.g. AlphaFold) incorporated
 
## Future Work
 
Hierarchical GO-aware loss functions
 
Few-shot learning for rare functions
 
Cross-species generalization
 
Integration with AlphaFold2 structural predictions
 
Scaling to larger protein language models
 
Moving from raw embedding dimensions toward monosemantic features via sparse autoencoders, causal intervention in activation space, and circuit reconstruction across layers
 
Variant effect prediction and protein engineering applications
 
## Citation
 
If you use or build upon this work, please cite:
 
@misc{gonzalez2025interpretability,
  title={Protein Function Prediction with Frozen ESM-2 Embeddings: An Interpretability Approach},
  author={Gonzalez, Daniela Alejandra},
  year={2025},
  institution={Northeastern University}
}
 
## License
 
Code is released under the MIT License.
Data usage is subject to Gene Ontology Consortium and UniProt terms.
 
## Contact
 
Daniela Alejandra Gonzalez
 
GitHub: https://github.com/DaniAGonzalez
 
Email: danigonz37@gmail.com
 
LinkedIn: https://www.linkedin.com/in/daniela-alejandra-gonz%C3%A1lez/
 
For questions, feedback, or collaboration, feel free to open an issue or contact me directly.
 
