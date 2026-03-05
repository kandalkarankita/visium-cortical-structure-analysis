# Visium Cortical Structure Analysis

## The Question

Visium spots are 55 micrometers wide and typically capture RNA from multiple cells at once. That raises a practical question before you even think about deconvolution: how much of the underlying tissue organization is still detectable from those mixed spot signals?

This project tests that directly. I ran unsupervised clustering on a public 10x Genomics mouse brain Visium section and checked whether the resulting clusters align with visible anatomy and known marker gene distributions, without using any cell type labels or a single cell reference.

---

## Dataset

10x Genomics Visium coronal mouse brain section, loaded via the Squidpy datasets API. 2,688 spots covering cortex, hippocampus, white matter, and subcortical structures. After QC filtering, 16,957 genes remained. The dataset loads with one line of code and runs under 2 GB RAM.

```python
import squidpy as sq
adata = sq.datasets.visium_hne_adata()
```

---

## What I Did

Library-size normalization to 10,000 counts per spot, log transformation, top 2,000 highly variable genes selected using the Seurat method. PCA on scaled values, 50 components computed, top 30 used for the neighbor graph. The variance ratio plot confirmed biological signal drops off before component 30. Leiden clustering at resolution 0.5 using 15 neighbors per spot. Spatially variable genes identified with Moran's I, 100 permutations, FDR corrected.

---

## Results

Leiden clustering produced 11 clusters. Every single one was spatially contiguous on the tissue. None were scattered randomly. The cluster boundaries matched the H&E image: a curved outer region consistent with cortical layers, a central and lower domain consistent with hippocampus, a left-side region consistent with white matter tracts, and a small concentrated cluster that looks like dentate gyrus or a hippocampal subfield.

Moran's I identified 10 strongly spatially structured genes, all with FDR p < 0.001.

| Gene | Moran's I | Known Association |
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

Three oligodendrocyte markers and three excitatory neuron markers in the top ten results is consistent with the dominant cell type differences between white matter and gray matter in this section. Slc17a7 and Nrgn showed overlapping spatial enrichment in hippocampal and cortical domains, which independently validated the hippocampal cluster without needing any anatomical annotation.

---

## What This Does and Does Not Show

The clustering and Moran's I results are consistent with each other and with published mouse brain atlas data. But two limitations apply here and to any Visium-based analysis.

Spot mixing means clusters represent the dominant cell type mixture at each location, not a pure cell population. What looks like a hippocampal cluster is really a region where hippocampal neuron signal is dominant. Confirming actual cell type proportions requires deconvolution against a single cell reference. Cell2location with the Allen Brain Atlas dataset is the obvious next step here.

The second concern is technical capture variation. Uneven tissue thickness or permeabilization can create spatial count gradients that drive cluster separation independently of biology. That said, a purely technical gradient would not be expected to consistently recover known brain region markers like Nrgn and Mbp. The convergence of marker identity with cluster boundaries makes a pure artifact explanation unlikely, though not impossible to fully exclude.

---

## Next Steps

Deconvolution with Cell2location using the Allen Brain Atlas single cell reference to get cell type proportions per spot. Then neighborhood enrichment analysis in Squidpy to identify which cell types co-localize beyond chance. That is where ligand-receptor interaction analysis and niche-level biology become accessible.

---

## Reproducibility

Python 3.9. All packages are publicly available. No manual data download needed.

```
pip install scanpy squidpy matplotlib seaborn leidenalg igraph
```

Runs on a standard laptop or Google Colab free tier under 2 GB RAM.

---

## References

Staahl et al. 2016. Visualization and analysis of gene expression in tissue sections by spatial transcriptomics. Science.

Kleshchevnikov et al. 2022. Cell2location maps fine-grained cell types in spatial transcriptomics. Nature Biotechnology.

Palla et al. 2022. Squidpy: a scalable framework for spatial omics analysis. Nature Methods.

Moran P.A.P. 1950. Notes on continuous stochastic phenomena. Biometrika.
