# Visium Cortical Structure Analysis

## Biological Question

Visium spatial transcriptomics captures gene expression across intact tissue sections, but each measurement spot contains signal from multiple cell types simultaneously. This raises a practical question: can unsupervised clustering on raw spot expression profiles still recover anatomically meaningful tissue organization, even without cell type deconvolution?

This project investigates that question using a public 10x Genomics Visium mouse brain dataset. The goal is not to reproduce known anatomy but to evaluate how much biological signal survives the spot mixing artifact, and where expression-based clusters can mislead interpretation.

---

## Motivation

Deconvolution tools like Cell2location require a matched single cell RNA-seq reference dataset, which is not always available in practice. Understanding how much anatomical structure is recoverable from Visium spot expression alone has direct implications for studies where only spatial data exists. This project examines that boundary and documents where the method breaks down.

---

## Dataset

10x Genomics Visium mouse brain section, loaded directly via Squidpy:
```python
import squidpy as sq
adata = sq.datasets.visium_hne_adata()
```

- Approximately 3,000 spots
- 33,000 genes before filtering
- 2,000 highly variable genes retained for analysis
- Paired H&E tissue image included for spatial overlay

No manual download required. Total memory usage stays under 2 GB.

---

## Analysis Workflow

Each step reflects a deliberate analytical decision, not just a pipeline default.

Step 1: Load dataset via Squidpy datasets module

Step 2: Quality control
- Remove spots with fewer than 200 detected genes
- Remove genes detected in fewer than 10 spots
- Filter spots where mitochondrial gene content exceeds 20 percent
- Rationale: high mitochondrial content indicates damaged tissue, not a biological signal

Step 3: Normalization
- Library size normalization to 10,000 counts per spot
- Log1p transformation to compress dynamic range
- Rationale: corrects for technical differences in total RNA captured per spot before comparing expression across spots

Step 4: Feature selection
- Retain top 2,000 highly variable genes
- Rationale: reduces noise from lowly expressed genes and keeps computational load within 4 GB RAM

Step 5: Dimensionality reduction
- PCA with 50 components
- Rationale: compresses gene space while preserving variance structure needed for clustering

Step 6: Clustering
- k-nearest neighbor graph in PCA space
- Leiden algorithm at resolution 0.5
- Rationale: resolution 0.5 was chosen to balance biological granularity against over-fragmentation of small tissue regions

Step 7: Spatial visualization
- Cluster labels overlaid on H&E tissue image using Squidpy
- This is the primary test of the project question

Step 8: Spatially variable gene detection
- Moran's I statistic computed per gene using Squidpy
- Identifies genes whose expression is spatially organized rather than randomly distributed across the tissue

---

## Key Findings

Leiden clustering recovers spatially coherent regions that align with major anatomical structures visible in the H&E image, including cortical layers, hippocampus, and white matter tracts.

Top spatially variable genes identified by Moran I include known region-specific markers consistent with published mouse brain atlas annotations.

Spots with high mitochondrial content were concentrated at specific tissue regions, confirming that QC filtering before clustering is necessary to avoid artifact-driven clusters.

---

## Critical Interpretation

The spatial alignment of clusters with anatomical regions is consistent with known mouse brain organization but does not confirm biological specificity on its own. Two alternative explanations must be considered before drawing conclusions.

Technical capture gradients: RNA capture efficiency varies across a tissue section due to differences in tissue thickness and preservation quality. A cluster appearing in a specific tissue region could reflect a gradient in total RNA yield rather than a true difference in gene expression between regions. This artifact is difficult to distinguish from biology using expression data alone.

Spot mixing: Each Visium spot captures RNA from multiple cells simultaneously. Clusters represent the dominant cell type mixture at each location, not a pure cell type signal. A cluster that appears anatomically specific may be driven by the proportion of one cell type rather than the unique expression program of that cell type. Confirming cell type identity requires deconvolution against a matched single cell RNA-seq reference.

These limitations do not invalidate the clustering result. They define the boundary of what the result can and cannot claim.

---

## Next Steps

Run Cell2location deconvolution using the Allen Brain Atlas single cell reference to assign cell type proportions per Visium spot and confirm whether cluster boundaries correspond to cell type composition boundaries.

Apply neighborhood enrichment analysis in Squidpy to identify which cell type clusters are spatially co-occurring beyond what is expected by chance, revealing potential cell-cell communication niches.

Test whether spatially variable genes identified by Moran I overlap with known layer-specific markers from the Allen Brain Atlas in situ hybridization database, as an independent validation of the spatial clustering result.

---

## Limitations of the Visium Platform

Spot resolution: At 55 micrometers per spot, Visium cannot resolve individual cells. Every result from this analysis reflects mixtures, not single cell measurements.

Ambient RNA: RNA from lysed cells diffuses across the capture array and contaminates neighboring spots. Genes that appear spatially variable may partly reflect diffusion from a nearby region rather than local expression.

Transcriptome coverage versus resolution tradeoff: Visium provides whole transcriptome coverage but sacrifices spatial resolution. Imaging-based platforms like Xenium provide single cell resolution but are limited to a pre-defined gene panel of a few hundred to one thousand genes. No current platform resolves both constraints simultaneously.

---

## Tools

- Python 3.9+
- Scanpy 1.9+
- Squidpy 1.3+
- Matplotlib
- Seaborn

---

## References

Staahl et al. 2016. Visualization and analysis of gene expression in tissue sections by spatial transcriptomics. Science.

Kleshchevnikov et al. 2022. Cell2location maps fine-grained cell types in spatial transcriptomics. Nature Biotechnology.

Palla et al. 2022. Squidpy: a scalable framework for spatial omics analysis. Nature Methods.
