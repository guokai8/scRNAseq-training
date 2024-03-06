## Clustering
1. In the following line of code ' Idents(object = seurat_integrated) <- "integrated_snn_res.0.4" ' how do we know to specifically use "integrated_snn_res.0.4" as the input to Idents() to set the resolution at 0.4?
    1. Seurat always uses the "integrated_snn_res." prefix before the resolution.
    1. After clustering that information is available in the "@clustering" slot in the Seurat object.
    1. **After clustering new column(s) are created in the "@meta.data" slot for each resolution, and the appropriate column header can be used as input to Idents().**

1. During QC of clustered cells we explore a lot of different things; which of the following is NOT assessed at this stage in the analysis?
    1. whether integration of our data resulted in cell type-specific clusters.
    1. whether the cells cluster by the number of genes, cell cycle phase or mitochondrial gene expression.
    1. whether we observe particular PCs driving specific clusters.
    1. **whether a specific library has aberrantly low numbers of UMI.**
    1. whether we see the cell types that we expect to be present using known cell type markers.
    1. whether our resolution is appropriate for identifying cell type-specific clusters.

## Marker Identification
1. Suppose your dataset has 2 experimental conditions - wildtype & knockout. You have finished with the clustering step and want to find the markers for a specific cluster. What is the appropriate function to use in Seurat?
    1. FindAllMarkers() 
    1. **FindConservedMarkers()**
    1. FindMarkers()
    
2. Which of the following statement about the FindAllMarkers() arguments is **NOT** correct?
    1. **A logfc.threshold value of 1 indicates that the expression is the same between compared clusters.**
    1. A higher value of min.diff.pct will result in more stringent filtering criteria.
    1. We can use a combination of arguments to adjust filtering stringency. 

3. Which of the following statement about the **output** of marker identification is **NOT** correct?
    1. We identify genes with positive and negative expression changes.
    1. The output p-value for each condition should not be used to interpret significance.
    1. **After identifying marker genes, we typically visualize their expression with a PCA plot.**
