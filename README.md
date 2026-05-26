# Protein Function Prediction with ESM-2: From Frozen Embeddings to Fine-Tuning

Predicting protein molecular functions from amino acid sequences using ESM-2, a protein language model pre-trained on 250 million sequences. This project explores two transfer learning strategies — frozen embeddings with comprehensive explainability analysis, and end-to-end fine-tuning — demonstrating how task-specific adaptation of foundation models improves performance in computational biology.

## Project Structure

```
├── 01_frozen_embeddings_interpretability.ipynb   # Frozen ESM-2 + full XAI pipeline
├── 02_finetuning.ipynb                           # Fine-tuned ESM-2 + performance comparison
├── figures/
└── README.md
```

## Motivation

Millions of sequenced proteins lack functional characterization. Traditional homology-based approaches fail for proteins without close relatives. Protein language models like ESM-2 capture evolutionary and biochemical patterns from sequence alone, but a key question remains: **should we use these models as fixed feature extractors, or adapt their weights for specific tasks?**

This project answers that question systematically, first establishing a strong frozen baseline with deep interpretability analysis, then demonstrating that fine-tuning substantially improves predictive performance.

## Part 1: Frozen Embeddings + Interpretability

**Notebook:** `01_frozen_embeddings_interpretability.ipynb`

ESM-2 is used as a frozen feature extractor: protein sequences are encoded into fixed 640-dimensional representations, and a lightweight classifier is trained on top for multi-label Gene Ontology molecular function prediction.

The central contribution is interpretability — understanding *why* predictions are made.

### Predictive Performance (Frozen)

| Metric | Value |
|--------|-------|
| Column-wise PR-AUC (per GO term) | 0.131 (8.4× over random) |
| Row-wise PR-AUC (per protein) | 0.349 (23.0× over random) |
| GPCRs | PR-AUC 0.996 |
| Protein kinases | PR-AUC 0.892 |

### Explainability Analysis

Three complementary techniques applied to 40 systematically selected proteins across 10 molecular function categories:

**Attention visualization:** Well-predicted proteins show focused attention at specific residue positions (structural landmarks, domain boundaries), while poorly predicted proteins show dispersed attention. Attention identifies computational landmarks rather than directly marking functional residues.

**Integrated gradients:** Function-specific embedding dimensions emerge without explicit supervision. Kinases rely on dimension 493, receptors on dimensions 532 and 565, transcription factors on dimensions 350 and 385. Dimension 553 acts as a universal functional protein signal across all families.

**In silico mutagenesis:** Single and combinatorial mutations cause minimal prediction changes (<5%), consistent with evolutionary robustness. Deletion scanning reveals critical regions: kinase activation loop deletion causes −10.7% prediction drop, while receptors remain robust due to architectural redundancy. This validates that the model learned genuine protein biology.

## Part 2: Fine-Tuning

**Notebook:** `02_finetuning.ipynb`

The same ESM-2 model is fine-tuned end-to-end: the last 10 of 30 transformer layers are unfrozen and trained jointly with the classification head. This allows ESM-2 to adapt its internal protein representations specifically for molecular function prediction.

### Fine-Tuning Strategy

| Parameter | Value |
|-----------|-------|
| Frozen layers | First 20 of 30 (embedding + early layers) |
| Trainable parameters | 50.1M (33.7% of total) |
| ESM-2 learning rate | 2e-5 |
| Classifier learning rate | 1e-3 |
| Epochs | 5 |
| Training time | ~10 min on A100 |

### Results: Frozen vs Fine-Tuned

| Metric | Frozen | Fine-Tuned | Change |
|--------|--------|------------|--------|
| Column-wise PR-AUC | 0.131 | 0.472 | +260% |
| Row-wise PR-AUC | 0.349 | 0.607 | +74% |
| Test Loss | 0.0737 | 0.0541 | -27% |

### Function-Specific Comparison

| Function | Frozen | Fine-Tuned | Change |
|----------|--------|------------|--------|
| Olfactory receptor | 0.996 | 1.000 | +0.4% |
| GPCR activity | 0.991 | 0.954 | -3.7% |
| Protein kinase | 0.892 | 0.954 | +7.0% |
| DNA-binding TF | 0.903 | 0.974 | +7.9% |
| Serine peptidase | 0.813 | 0.913 | +12.3% |
| Kinase activity | 0.650 | 0.989 | +52.1% |

**Key insight:** Fine-tuning improves performance most for functions where frozen embeddings were weakest, suggesting the model learns task-specific features that complement the general evolutionary patterns captured during pre-training. Functions already near ceiling (olfactory receptors, GPCRs) show minimal change.

## Dataset

| Property | Value |
|----------|-------|
| Source | Gene Ontology Consortium + UniProt human proteome |
| Proteins | ~7,300–8,700 (varies by GO version) |
| GO terms | 103–202 molecular function terms |
| Max sequence length | 500 amino acids |
| Split | 60% train / 20% valid / 20% test |

Data is not distributed in this repository. All data loading, preprocessing, and embedding generation steps are fully implemented in code — public data are downloaded directly from GO Consortium and UniProt within the notebooks.

## Methods Summary

### Modeling Pipeline

- Feature extraction / fine-tuning: ESM-2 (`facebook/esm2_t30_150M_UR50D`, 150M parameters)
- Mean pooling → 640-dimensional protein embeddings
- Classifier: 640 → 512 → 256 → n_labels (batch normalization, ReLU, dropout 0.3, sigmoid)
- Binary cross-entropy loss, early stopping (frozen) / linear warmup scheduler (fine-tuning)

### Interpretability Techniques (Frozen)

- Attention visualization (averaged across 30 layers × 20 heads)
- Integrated gradients (Captum, zero-embedding baseline)
- In silico mutagenesis (alanine scanning, combinatorial, deletion scanning)

## Future Work

- Apply the full XAI pipeline (attention, integrated gradients, in silico mutagenesis) to the fine-tuned model and compare how fine-tuning changes interpretability patterns
- Extract fine-tuned embeddings for integration with GNN-based drug-drug interaction prediction
- Analyze how fine-tuning reshapes the embedding space (t-SNE, dimension-level comparison)
- Hierarchical GO-aware loss functions
- Few-shot learning for rare functions
- Cross-species generalization
- Integration with AlphaFold2 structural predictions
- Scaling to larger ESM-2 variants (650M, 3B)

## Installation

```bash
pip install torch transformers captum scikit-learn pandas numpy matplotlib seaborn biopython obonet
```

## Tested On

- Python 3.11+
- PyTorch 2.0+
- transformers 4.35+
- macOS (Apple Silicon, MPS), Linux (CUDA, A100)

## Citation

```bibtex
@misc{gonzalez2025esm2,
  title={Protein Function Prediction with ESM-2: From Frozen Embeddings to Fine-Tuning},
  author={Gonzalez, Daniela Alejandra},
  year={2025},
  institution={Northeastern University}
}
```

## License

Code is released under the MIT License.
Data usage is subject to Gene Ontology Consortium and UniProt terms.

## Contact

Daniela Alejandra Gonzalez

GitHub: [github.com/DaniAGonzalez](https://github.com/DaniAGonzalez)
