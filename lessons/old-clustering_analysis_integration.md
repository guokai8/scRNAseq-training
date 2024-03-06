---
title: "Single-cell RNA-seq: Clustering Analysis"
author: "Mary Piper, Lorena Pantano, Meeta Mistry, Radhika Khetani"
date: Wednesday, June 19, 2019
---

Approximate time: 90 minutes

## Learning Objectives:

* Utilize integration methods to generate cell type-specific clusters across conditions, batches, or technologies
* Perform clustering and marker identification workflow with integrated data

# Single-cell RNA-seq clustering analysis

After determining the cell types expected in the control sample, we would like to include the interferon-stimulated sample together with the control to compare the cell types present and/or genes differentially expressed between conditions. 

Oftentimes, if samples are created using different conditions (or batches), the effect of the condition can be so great that the cells will cluster by condition instead of cell type. To ensure the same cell types cluster together, we can perform an integration analysis detailed in the paper by [Stuart and Bulter et al. (2018)](https://www.biorxiv.org/content/early/2018/11/02/460147). 
Therefore, we are going to integrate the cells from our control and stimulated samples together.

<img src="../img/sc_workflow_integration.png" width="800">

_**Goals:**_ 
 
- _To **align the cells** corresponding to the same cell types across conditions, batches or technologies_

_**Challenges:**_
 
 - _Clustering so that **cells of the same cell type from different conditions cluster together**_
 - _Ensuring the alignment of cells is appropriate with respect to the metadata of the cells_

_**Recommendations:**_
 
 - _For multi-sample experiments, err on the side of **aligment of cells** rather than not aligning. Generally, alignment shouldn't negatively affect the analysis, although less processing is always preferred_
 - _Following integration, err on the side of using **more PCs for clustering** than fewer PCs. Usually use at least 20 - 30 PCs for clustering; additional PCs do not affect the clustering very much after ~30 PCs depending on the size of the dataset and the variability present._

 
## Integration


To integrate multiple samples together, we need to perform the following steps:

1. **Extract each sample** as a Seurat object
2. **Normalize** the cell counts for library depth and identify **highly variable genes** for each sample
4. **Integrate** samples using shared highly variable genes

### Extract each sample as a Seurat object

The first thing we do is extract each sample as an individual Seurat object.

```r
# Create Seurat object from filtered SingleCellExperiment object
data_dir <- "data"

# Check names of samples
levels(factor(seurat_raw@meta.data$sample))
```

Now we can extract each sample as an individual Seurat object

```r
# Check names of samples
sample_names <- levels(factor(seurat_raw@meta.data$sample))

for (sample in sample_names){

        # Determine cell IDs of cells in each sample
        cell_ids <- seurat_raw@meta.data[which(seurat_raw@meta.data$sample == sample), ] %>%
                rownames()

        # Subset the Seurat object to single sample
        seurat_obj <- subset(seurat_raw,
                     		  cells = cell_ids)

        # Save sample names to a variable
        seurat_sample <- sample
		 
		 # Assign the variable as an object in the environment
        assign(seurat_sample, seurat_obj)
}

```

We should now have variables/objects in our environment called `ctrl` and `stim`.

### Normalize the cell counts for library depth and identify highly variable genes

Before comparing expression profiles of cells to determine similarity, we need to normalize the cell counts for sequencing depth per cell for each sample separately. We perform this step as done previously using the `LogNormalize` method.

Then, to align similar cells across samples we need to identify the most variable genes for each of the samples. Similar to previously, the mean-variance relationship of the data is modeled and the 2,000 most variable genes are returned.

```r
for (i in 1:length(seurat_list)) {
        seurat_list[[i]] <- NormalizeData(seurat_list[[i]], 
                                          verbose = FALSE)
        seurat_list[[i]] <- FindVariableFeatures(seurat_list[[i]], 
                                                 selection.method = "vst", 
                                                 nfeatures = 2000,
                                                 verbose = FALSE)
}                         
```

We can check the most highly variable genes for each sample, to see if the genes make sense regarding the cell types expected.

**Control**

```r
# Identify the 20 most highly variable genes
top20 <- head(x = VariableFeatures(object = seurat_list$ctrl), 
              n = 20)

# Variable gene plot
unlabelled <- VariableFeaturePlot(object = ctrl)

# With labels
LabelPoints(plot = unlabelled,
            points = top20,
            repel = TRUE)
```

<p align="center">
<img src="../img/Ctrl_integ_var_genes_plot.png" width="800">
</p>

**Interferon-stimulated**

```r
# Identify the 20 most highly variable genes
top20 <- head(x = VariableFeatures(object = seurat_list$stim), 
              n = 20)

# Variable gene plot
unlabelled <- VariableFeaturePlot(object = stim)

# With labels
LabelPoints(plot = unlabelled,
            points = top20,
            repel = TRUE)
```

<p align="center">
<img src="../img/Stim_integ_var_genes_plot.png" width="800">
</p>

You will notice that many of the most highly variable genes are present between the two samples, which is a good sign for aligning cells.


### **Integrate** samples using shared highly variable genes

_**This step can greatly improve your clustering when you have multiple samples**. It can help to first run samples individually if unsure what clusters to expect, but when clustering the cells from multiple conditions, integration can help ensure the same cell types cluster together._

Using the shared highly variable genes from each sample, we integrate the samples to overlay cells that are similar or have a "common set of biological features". The process of integration uses canonical correlation analysis (CCA) and mutual nearest neighbors (MNN) methods to identify shared subpopulations across samples or datasets [[Stuart and Bulter et al. (2018)](https://www.biorxiv.org/content/early/2018/11/02/460147)]. 

Specifically, this integration method expects "correspondences" or **shared biological states** among at least a subset of single cells across the samples. The steps applied are as follows:

1. CCA is performed, which uses **shared highly variable genes** to reduce the dimensionality of the data and align the cells in each sample into the maximally correlated space (based on sets of genes exhibiting robust correlation in expression). **Shared highly variable genes are most likely to represent those genes distinguishing the different cell types present.**
2. Identify mutual nearest neighbors, or 'anchors' across datasets (sometimes incorrect anchors are identified)
	> MNNs identify the cells that are most similar to each other across samples or conditions. "The difference in expression values between cells in an MNN pair provides an estimate of the batch effect, which is made more precise by averaging across many such pairs. A correction vector is obtained from the estimated batch effect and applied to the expression values to perform batch correction. Our approach automatically identifies overlaps in population composition between batches and uses only the overlapping subsets for correction, thus avoiding the assumption of equal composition required by other methods." 
3. Assess the similarity between anchor pairs by the overlap in their local neighborhoods (incorrect anchors will have low scores)
4. Use anchors and corresponding scores to transform cell expression values, allowing for the integration of the datasets (different samples, datasets, modalities)
	- Transformation of each cell uses a weighted average of the two cells of each anchor across anchors of the datasets. Weights determined by cell similarity score (distance between cell and k nearest anchors) and anchor scores, so cells in the same neighborhood should have similar correction values.

If cell types are present in one dataset, but not the other, then the cells will still appear as a separate sample-specific cluster.

<p align="center">
<img src="../img/integration.png" width="600">
</p>

_Image credit: Stuart T and Butler A, et al. Comprehensive integration of single cell data, bioRxiv 2018 (https://doi.org/10.1101/460147)_

```r
# Identify anchors
anchors <- FindIntegrationAnchors(object.list = seurat_list, 
                                  dims = 1:30)

# Integrate samples
combined <- IntegrateData(anchorset = anchors, 
                          dims = 1:30)
                          
DefaultAssay(object = combined) <- "integrated"                                                            
```

## Scaling and regression of sources of unwanted variation

In addition to there being interesting variation that separates the different cell types in your dataset, there is also "uninteresting" sources of variation present that can obscure the cell type-specific differences. This can include technical noise, batch effects, and/or uncontrolled biological variation (e.g. cell cycle).

### Cell cycle scoring

Let's score each gene for cell cycle phase, then perform PCA using the expression of cell cycle genes. Remember, if the cells group by cell cycle in the PCA, then we would want to regress out cell cycle variation, **unless cells are differentiating**.  

<p align="center">
<img src="../img/SC_preregressed_phase_pca.png" width="600">
</p>

```r
combined <- ScaleData(object = combined, 
                      verbose = FALSE)

cycle_genes <- read.csv("data/Homo_sapiens.txt")

cycle_genes <- inner_join(cycle_genes, annotations, by = c("geneID" = "gene_id"))

g2m_genes <- cycle_genes %>%
        dplyr::filter(phase == "G2/M") %>%
        pull(gene_name)

s_genes <- cycle_genes %>%
        dplyr::filter(phase == "S") %>%
        pull(gene_name)


# Perform cell cycle scoring
combined <- CellCycleScoring(
        combined,
        g2m.features = g2m_genes,
        s.features = s_genes)

# Perform PCA and color by cell cycle phase
combined <- RunPCA(combined,
                   features = c(s_genes, g2m_genes),
                   verbose = FALSE)

DimPlot(object = combined, 
        reduction = "pca",
        group.by= "Phase")

```

<p align="center">
<img src="../img/pca_cellcycle_pre_regress.png" width="600">
</p>

### Apply regression variables

Similar to the analysis with the `control` sample, we will regress the uninteresting sources of variation, including number of UMIs, mitochondrial ratio, and cell cycle scores to **improve downstream identification of principal components and clustering.** 

```r
# Define variables in metadata to regress
vars_to_regress <- c("nUMI", "mitoRatio", "S.Score", "G2M.Score")

# Regress out the uninteresting sources of variation in the data
combined <- ScaleData(combined, 
                      vars.to.regress = vars_to_regress)
                      
# Perform PCA and color by cell cycle phase
combined <- RunPCA(combined,
                   verbose = FALSE)

DimPlot(object = combined, 
        reduction = "pca",
        group.by= "Phase")
```

<p align="center">
<img src="../img/pca_cellcycle_post_regress.png" width="600">
</p>



## Clustering cells based on top PCs (metagenes)

### Identify significant PCs

For integrated data, we usually use the number of PCs utilized in the integration of the data. In addition, having a higher number of PCs does not normally change the clustering drastically after about 30 PCs. However, it's always a good idea to double-check to make sure that the majority of the variation is explained.

Let's to make sure that 30 PCs capture the most significant PCs by plotting the elbow plot.

<p align="center">
<img src="../img/sc_integ_elbow_plot.png" width="600">
</p>

Based on this elbow plot, it appears that 30 PCs capture the majority of the variation. We could quantify just to make sure:

```r
# Determine percent of variation associated with each PC
pct <- combined[["pca"]]@stdev / sum(combined[["pca"]]@stdev) * 100

# Calculate cumulative percents for each PC
cumu <- cumsum(pct)

# Determine which PC exhibits cumulative percent greater than 90% and % variation associated with the PC as less than 5
co1 <- which(cumu > 90 & pct < 5)[1]

co1

# Determine the difference between variation of PC and subsequent PC
co2 <- sort(which((pct[1:length(pct)-1] - pct[2:length(pct)]) > 0.1),
            decreasing = T)[1] + 1 
# last point where change of % of variation is more than 0.1%.
co2

# Minimum of the two calculations
min(co1, co2)
```

Quantification indicates that 30 PCs more than captures the majority of the variation based on the difference in variation between subsequent PCs. We will continue to use the 30 PCs that we had initially used for the integration.

## Cluster the cells

We can now use these PCs to determine which cells exhibit similar expression patterns for clustering using the K-nearest neighbor (KNN) graph, with edges drawn between cells with similar gene expression patterns partitioned into highly interconnected ‘quasi-cliques’ or ‘communities’.

```r
# Determine the K-nearest neighbor graph
combined <- FindNeighbors(object = combined, 
                                dims = 1:30)

# Determine the clusters for various resolutions                                
combined <- FindClusters(object = combined,
                               resolution = c(0.4, 0.6, 0.8, 1.2, 1.8))
```                               


### Creating non-linear dimensional reduction (UMAP/tSNE)

To visualize the clusters, we will use UMAP and PCA. The goals of these methods is to have similar cells closer together in low-dimensional space.

```r
# Choose clustering resolution for first round of analysis
colnames(combined@meta.data)

Idents(object = combined) <- "integrated_snn_res.0.8"

# Calculate UMAP coordinates
combined <- RunUMAP(combined,
                    reduction = "pca",
                    dims = 1:30)

# Plot UMAP
DimPlot(combined,
        reduction = "umap",
        label = TRUE,
        label.size = 6,
        plot.title = "UMAP")
```


<p align="center">
<img src="../img/integ_umap.png" width="800">
</p>

This may be too many clusters, but we can re-analyze with a lower resolution if it appears like we have too many after the marker identification. 


***

**Exercise**

1. **Exploration of quality control metrics:**

To determine whether our clusters might be due to artifacts such as cell cycle phase or mitochondrial expression, it can be useful to explore these metrics visually to see if any clusters exhibit enrichment or are different from the other clusters. However, if enrichment or differences are observed for particular clusters it may not be worrisome if it can be explained by the cell type. 

Using the `control` sample as a reference, perform all of the QC checks on the clusters.

```r
# Extract identity and sample information from seurat object to determine the number of cells per cluster per sample
n_cells <- FetchData(combined, 
                     vars = c("ident", "sample")) %>%
        group_by(sample) %>%
        dplyr::count(ident) %>% 
        spread(ident, n) 

# View table
View(n_cells)
```

<p align="center">
<img src="../img/integ_ncells.png" width="800">
</p>

We can additionally visualize whether we have any sample-specific clusters by using the `split.by` argument:

```r
# Plot UMAP split by sample
DimPlot(combined,
        reduction = "umap",
        split.by = "sample",
        label = TRUE,
        label.size = 6,
        plot.title = "UMAP")

```

<p align="center">
<img src="../img/integ_umap_split.png" width="800">
</p>

There doesn't appear to be any sample-specific clusters present. 

We can also perform the standard QC that we had gone through previously with the control sample.


We can explore cell cycle by cluster by splitting by `Phase`. 

```r
DimPlot(combined,
        reduction = "umap",
        split.by = "Phase",
        label = TRUE,
        label.size = 6,
        plot.title = "UMAP")
```
<p align="center">
<img src="../img/integ_cluster_phase.png" width="800">
</p>

Next, we will explore additional metrics, such as the number of UMIs and genes per cell, S-phase and G2M-phase markers, and mitochondrial gene expression:

<img src="../img/integ_cluster_qc.png" width="800">

We can also explore how well our clusters separate by the different PCs; we hope that the defined PCs separate the cell types well. In the tSNE plots below, the cells are colored by their PC score for each respective principal component.

<p align="center">
<img src="../img/integ_cluster_qc_pcs.png" width="600">
</p>

# Evaluating clustering

To determine whether our clustering and resolution are appropriate for our experiment, it is helpful to explore a handful of markers for each of the major cell types that we expect to be in our data and see how they segregate.

We will explore known immune cell markers for expected clusters:

| Cell Type | Marker |
|:---:|:---:|
| CD14+ monocytes | CD14, LYZ | 
| FCGR3A+ monocytes | FCGR3A, MS4A7 |
| Conventional dendritic cells | FCER1A, CST3 |
| Plasmacytoid dendritic cells | IL3RA, GZMB, SERPINF1, ITM2C |
| B cells | CD79A, MS4A1 |
| T cells | CD3D |
| CD4+ T cells | CD3D, IL7R, CCR7 |
| CD8+ T cells| CD3D, CD8A |
| NK cells | GNLY, NKG7 |
| Megakaryocytes | PPBP |
| Erythrocytes | HBB, HBA2 |


Let's remind ourselves of our clusters:

```r
# Plot UMAP
DimPlot(combined,
        reduction = "umap",
        label = TRUE,
        label.size = 6,
        plot.title = "UMAP")
```


<p align="center">
<img src="../img/integ_umap.png" width="800">
</p>


**CD14+ monocyte markers**

```r
FeaturePlot(combined, 
            reduction = "umap", 
            features = c("CD14", "LYZ"))
```

<p align="center">
<img src="../img/integ_markers_CD14_monocytes.png" width="800">
</p>

Cluster 0 appears to have expression of both CD14 and LYZ. Cluster 7 also appears to express both markers, but at a lower level.

**FCGR3A+ monocyte markers**

```r
FeaturePlot(combined, 
            reduction = "umap", 
            features = c("FCGR3A", "MS4A7"))
```

<p align="center">
<img src="../img/integ_markers_FCGR3A_monocytes.png" width="800">
</p>

Expression is strong for only cluster 7. Therefore, we will surmise that cluster 0 is CD14+ monocytes, while cluster 7 represents FCGR3A+ (CD16+) monocytes.

**Dendritic cell (DC) markers**

```r
FeaturePlot(combined, 
            reduction = "umap", 
            features = c("FCER1A", "CST3"))
```

<p align="center">
<img src="../img/integ_markers_DCs.png" width="800">
</p>

Although the expression is not that impressive, the only cluster to exhibit expression of both markers is cluster 13. Since these markers are weak, we would likely want to explore the marker identification markers in more detail for this cell type.

**Plasmacytoid DCs**

```r
FeaturePlot(combined, 
            reduction = "umap", 
            features = c("IL3RA", "GZMB", "SERPINF1", "ITM2C"))
```

<p align="center">
<img src="../img/integ_markers_pDCs.png" width="800">
</p>

The expression of several markers suggest cluster 18 corresponds to plasmacytoid DCs.
            
**B cell markers**

```r
FeaturePlot(combined, 
            reduction = "umap", 
            features = c("CD79A", "MS4A1"))
```

<p align="center">
<img src="../img/integ_markers_Bcells.png" width="800">
</p>

These B cell markers have a good amount of expression for both markers for clusters 4, 14, and 15.

**T cell markers**

```r
FeaturePlot(combined, 
            reduction = "umap", 
            features = c("CD3D"))
```

<p align="center">
<img src="../img/integ_markers_Tcells.png" width="600">
</p>

The T cell marker maps to many clusters, including 1, 2, 3, 6, 9, 10, 12, 16, 17, and 19.

**CD4+ T cell markers**

```r
FeaturePlot(combined, 
            reduction = "umap", 
            features = c("CD3D", "IL7R"))
```

<p align="center">
<img src="../img/integ_markers_CD4Tcells.png" width="800">
</p>

These markers correspond to clusters 1, 2, 3, 9 and 16. IL7R appears to map to cluster 10.

**CD8+ T cell markers**

```r
FeaturePlot(combined, 
            reduction = "umap", 
            features = c("CD3D", "CD8A"))
```

<p align="center">
<img src="../img/integ_markers_CD8Tcells.png" width="800">
</p>

Markers for CD8+ T cells map to clusters 6, 10 and 11.

**NK cell markers**

```r
FeaturePlot(combined, 
            reduction = "umap", 
            features = c("GNLY", "NKG7"))
```

<p align="center">
<img src="../img/integ_markers_NKcells.png" width="800">
</p>

The NK cell markers correspond to clusters 5 and 6. However, we know that cluster 6 has T cell markers, therefore, this cluster is not representing NK cells. Likely cluster 6 represents activated CD8+ T cells.

**Megakaryocyte markers**

```r
FeaturePlot(combined, 
            reduction = "umap", 
            features = c("PPBP"))
```

<p align="center">
<img src="../img/integ_markers_megakaryocytes.png" width="600">
</p>

This marker distinguishes cluster 12 as megakaryocytes.

**Erythrocyte markers**

```r
FeaturePlot(combined, 
            reduction = "umap", 
            features = c("HBB", "HBA2"))
```
<p align="center">
<img src="../img/integ_markers_erythrocytes.png" width="600">
</p>

Erythocytes correspond to cluster 19.

We identified in the Control analysis that we had markers differentiating the naive from the activated T cells. We can look whether specific clusters correspond to these subsets.

### Cell state exploration

Based on the analysis with the `Control` sample, we have some additional cell markers to determine **immune cell activation status** and **stressed/dying** cells:

| Cell State | Marker |
|:---:|:---:|
| Naive T cells | CCR7, SELL | 
| Activated B and T cells | CREM, CD69 |
| Stressed/dying cells | HSPB1, DNAJB6, HSPH1, GADD45B |

**Naive or memory T cells**

```r
FeaturePlot(combined, 
            reduction = "umap", 
            features = c( "CCR7", "SELL"))
```

<p align="center">
<img src="../img/integ_markers_t_naive.png" width="600">
</p>

The naive CD4+ T cells correspond to clusters 1, 2, 9, and 11.

**Activated T and B cells**

```r
FeaturePlot(combined, 
            reduction = "umap", 
            features = c("CREM", "CD69"))
```            

<p align="center">
<img src="../img/integ_markers_activated_t_cells.png" width="600">
</p>

Activated CD4+ T cells correspond to cluster 3, while activated B cells correspond to cluster 14.

**Stressed or dying cells**

Based on our previous analysis with the control sample, we know there should be a cluster containing the stressed or dying cells with heat shock genes, cell cycle arrest genes, and DNA damage genes overexpressed. We can determine which cluster corresponds to these cells using the identified markers from the previous analysis.

```r
FeaturePlot(combined, 
            reduction = "umap", 
            features = c("HSPB1", "DNAJB6", "HSPH1", "GADD45B"))
```           
        
<p align="center">
<img src="../img/integ_markers_stressed.png" width="600">
</p>

This cluster of stressed cells corresponds to cluster 8.
            
Based on these results, we can associate the majority of the clusters with cell types. However, we would like to perform a deeper analysis using marker identification before performing a final assignment of the clusters. 

| Cell Type | Clusters |
|:---:|:---:|
| CD14+ Monocytes | 0 | 
| FCGR3A+ Monocytes | 7 |
| Conventional dendritic cells | 13 |
| Plasmacytoid dendritic cells | 18 |
| Naive B cells | 4, 15 |
| Activated B cells | 14 |
| Naive CD4+ T cells | 1, 2, 9, 16 |
| Activated CD4+ T cells | 3 |
| Naive CD8+ T cells| 11 |
| Activated (cytotoxic) CD8+ T cells| 6, 10 |
| NK cells | 5 |
| Megakaryocytes | 12 |
| Erythrocytes | 19 |
| Stressed / dying cells | 8 |
| Unknown | 17, 20 |

While we have a good idea of the cell types for the different clusters, it's always a good idea to perform marker identification to ensure the hypothesized cell identities make sense with the enriched genes. Also, we can explore identities for the unknown clusters 17 and 20.

[Click here for next lesson]()

***

*This lesson has been developed by members of the teaching team at the [Harvard Chan Bioinformatics Core (HBC)](http://bioinformatics.sph.harvard.edu/). These are open access materials distributed under the terms of the [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0), which permits unrestricted use, distribution, and reproduction in any medium, provided the original author and source are credited.*

* *A portion of these materials and hands-on activities were adapted from the [Satija Lab's](https://satijalab.org/) [Seurat - Guided Clustering Tutorial](https://satijalab.org/seurat/pbmc3k_tutorial.html)*

