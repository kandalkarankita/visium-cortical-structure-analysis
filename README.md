# Visium Cortical Structure Analysis

## Overview

Spot-based spatial transcriptomics platforms like Visium measure gene expression across intact tissue sections but cannot resolve individual cells. Each 55-micrometer capture spot aggregates RNA from multiple cell types simultaneously, raising a fundamental analytical question: how much anatomical structure is recoverable from spot-level expression profiles without cell type deconvolution?

This project addresses that question directly using a public 10x Genomics Visium mouse brain dataset. Unsupervised Leiden clustering was applied to normalized spot expression profiles, and results were evaluated against H&E tissue morphology and spatially variable gene identity. The analysis documents both the signal that survives spot mixing and the interpretive limits that remain without a matched single cell reference.

---

## Motivation

Deconvolution methods such as Cell2location and Tangram resolve cell type composition within Visium spots but require a matched single cell RNA-seq reference, which is not always available. Understanding the extent to which tissue organization is detectable from spatial expression alone is practically relevant for studies operating without a reference dataset, and for evaluating how much confidence to place in cluster-based spatial annotations before deconvolution is performed.

---

## Dataset

10x Genomics Visium coronal mouse brain section, accessed via the Squidpy datasets API. The section captures 2,688 spots across cortex, hippocampus, white matter, and subcortical structures. After quality control filtering, 16,957 genes were retained. The top 2,000 highly variable genes were used for dimensionality reduction and clustering. The dataset loads programmatically with no manual download and runs within 2 GB RAM.

```python
import squidpy as sq
adata = sq.datasets.visium_hne_adata()
```

---

## Methods

Raw count data were library-size normalized to 10,000 counts per spot and log-transformed. The top 2,000 highly variable genes were selected using the Seurat mean-variance method. PCA was performed on scaled expression values, and a k-nearest neighbor graph was constructed in 30-component PCA space with 15 neighbors per spot. Leiden clustering at resolution 0.5 produced 11 clusters. Spatial coherence was assessed by overlaying cluster labels on the paired H&E tissue image. Spatially variable genes were identified using Moran's I spatial autocorrelation statistic with 100 permutations and FDR correction.

---

## Results

### Spatial Coherence of Expression Clusters

All 11 Leiden clusters occupied spatially contiguous domains in the tissue. No cluster was randomly distributed. Cluster boundaries aligned with visible anatomical structures in the H&E image, including cortical layers along the outer curvature of the section, a distinct hippocampal domain in the central and lower regions, white matter tracts on the left, and a small concentrated cluster consistent with the dentate gyrus or a specialized hippocampal subfield.

This result demonstrates that gene expression differences between tissue regions are sufficiently strong to drive spatially coherent clustering from spot-level data alone, without anatomical labels or prior cell type annotation.

### Spatially Variable Genes

Moran's I analysis identified 10 genes with strong spatial autocorrelation, all with FDR-corrected p-values below 0.001.

| Gene | Moran's I | Cell Type Association |
|---|---|---|
| Mbp | 0.788 | Oligodendrocyte |
| Slc17a7 | 0.775 | Excitatory neuron |
| Nrgn | 0.743 | Hippocampal neuron |
| Cck | 0.727 | Cortical interneuron |
| Itpka | 0.698 | Neuron |
| Mobp | 0.696 | Oligodendrocyte |
| Camk2n1 | 0.695 | Excitatory neuron |
| Plp1 | 0.689 | Oligodendrocyte |
| Baiap3 | 0.689 | Neuron |
| Ddn | 0.681 | Dendritic spine |

All ten genes are established brain region specific markers. The convergence of three oligodendrocyte markers (Mbp, Mobp, Plp1) and three excitatory neuron markers (Slc17a7, Nrgn, Camk2n1) in the top results is consistent with the dominant cell type composition differences between white matter and gray matter regions in this section.

### Spatial Expression Patterns

Spatial visualization of the top four genes confirmed expected anatomical distributions. Mbp was broadly enriched across the tissue with higher signal in white matter regions. Slc17a7 and Nrgn showed overlapping enrichment in hippocampal and cortical domains, providing convergent gene-level evidence for the hippocampal cluster identified by Leiden clustering. Cck showed a more restricted distribution consistent with its known enrichment in specific cortical layers and hippocampal subfields.

The spatial overlap between Slc17a7 and Nrgn enrichment zones independently validates the cluster boundary without requiring anatomical annotation.

---

## Interpretation and Limitations

The clustering and spatially variable gene results are mutually consistent and align with published mouse brain atlas data. However two sources of interpretive uncertainty apply to all Visium-based analyses and should be stated explicitly.

Spot mixing: Each Visium spot captures RNA from an average of two to ten cells depending on tissue density. Cluster identity reflects the dominant cell type mixture at each location rather than the expression program of any single cell type. The spatial patterns observed here are interpretable as regional cell type composition differences, not as pure cell type signatures. Confirming cell type identity and proportion within each cluster requires deconvolution against a single cell RNA-seq reference using a tool such as Cell2location or Tangram.

Technical capture variation: RNA capture efficiency varies across a tissue section as a function of tissue thickness, fixation quality, and proximity to the capture array surface. Spatial gradients in total counts can in principle drive cluster separation independent of true biological differences in gene expression. In this dataset, the convergence of clustering results with spatially variable gene identity reduces but does not eliminate this concern. Systematic capture variation would not be expected to produce the specific known marker genes observed here.

---

## Next Steps

The immediate next step is deconvolution using the Allen Brain Atlas single cell reference to assign cell type proportions per spot and determine whether cluster boundaries correspond to cell type composition transitions. Following deconvolution, neighborhood enrichment analysis in Squidpy would quantify which cell types are spatially co-enriched beyond chance, enabling ligand-receptor interaction analysis within identified tissue niches. Validation of spatially variable genes against Allen Brain Atlas in situ hybridization data would provide independent anatomical confirmation of the Moran's I results.

---

## Reproducibility

All analysis was performed in Python 3.9 using publicly available tools. The dataset loads automatically via Squidpy with no manual download. The full pipeline runs within 2 GB RAM and is suitable for execution on standard laptop hardware or Google Colab free tier.

```
pip install scanpy squidpy matplotlib seaborn leidenalg igraph
```

---

## References

Staahl et al. 2016. Visualization and analysis of gene expression in tissue sections by spatial transcriptomics. Science.

Kleshchevnikov et al. 2022. Cell2location maps fine-grained cell types in spatial transcriptomics. Nature Biotechnology.

Palla et al. 2022. Squidpy: a scalable framework for spatial omics analysis. Nature Methods.

Moran P.A.P. 1950. Notes on continuous stochastic phenomena. Biometrika.
