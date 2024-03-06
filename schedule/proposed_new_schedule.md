# Schedule for the single-cell RNA-seq data analysis workshop

## Pre-reading

* [Introduction to scRNA-seq](../lessons/01_intro_to_scRNA-seq.md)
* [Raw data to count matrix](../lessons/02_SC_generation_of_count_matrix.md)
* [Download this project](https://www.dropbox.com/s/vop78wq76h02a2f/single_cell_rnaseq.zip?dl=1)

## Day 1

| Time |  Topic  | Instructor |
|:-----------:|:----------:|:--------:|
| 09:30 - 09:45 | [Workshop introduction](../slides/Intro_to_workshop_all_in_person.pdf) | Meeta |
| 09:45 - 11:00| [Introduction to Single Cell RNA-sequencing: a practical guide](../slides/101023_Chan_workshop_AK_latest.pdf) | [Dr. Arpita Kulkarni](https://singlecellcore.hms.harvard.edu/people/arpita-kulkarni-phd) |
| 11:00 - 11:05 | Break |
| 11:05 - 11:15 | scRNA-seq pre-reading discussion | All |
| 11:15 - 11:55 | [Quality control set-up](../lessons/03_SC_quality_control-setup.md) | Noor |
| 11:55 - 12:00 | Overview of self-learning materials and homework submission | Meeta |


### Before the next class:

I. Please **study the contents** and **work through all the code** within the following lessons:
   1. [Overview of scRNA-seq Analysis Workflow](../lessons/postQC_workflow.md)
      <details>
       <summary><i>Click here for a preview of this lesson</i></summary>
         <br>Let's do a quick overview of what's in store for us during the workshop. For each step we describe Goals, Challenges and Reccomendations to guide you on your way. This is a great reference page for you when you run your own analyses!<br><br>
         </details>
   2. [Quality control of cellranger counts]()
      <details>
       <summary><i>Click here for a preview of this lesson</i></summary>
        </details>

 3.  [Quality control with additional metrics](../lessons/04_SC_quality_control.md)
      <details>
       <summary><i>Click here for a preview of this lesson</i></summary>
         <br>Before you start any analysis, it’s important to know whether or not you have good quality cells. At these early stages you can flag or remove samples that could produce erroneous results downstream. <br><br>In this lesson you will:<br>
             - Compute essential QC metrics for each sample<br>
             - Create plots to visualize metrics per sample<br>
             - Critically evaluate each plot and learn what each QC metric means<br><br>
        </details>

         
   4. [Theory of PCA](../lessons/05_theory_of_PCA.md)
      <details>
       <summary><i>Click here for a preview of this lesson</i></summary>
         <br>Before we can begin the next steps of the workflow, we need to make sure you have a good understanding of Principal Components Analysis (PCA). This method will be utilized in the scRNA-seq analysis workflow, and this foundation will help you better navigate those steps and interpretation of results.<br><br>
        </details>
        
II. **Submit your work**:
   * Each lesson above contains exercises; please go through each of them.
   * **Submit your answers** to the exercises using [this Google form](https://docs.google.com/forms/d/e/1FAIpQLSf4zw37eUcuNCmHxCLikg3JENoxGWIuv5c_5pqHdYNId81Arg/viewform) on **the day *before* the next class**.
   

### Questions?
* ***If you get stuck due to an error*** while runnning code in the lesson, [email us](mailto:hbctraining@hsph.harvard.edu) 
* Post any **conceptual questions** that you would like to have **reviewed in class** [here](https://PollEv.com/hbctraining945).

***

## Day 2

| Time |  Topic  | Instructor |
|:-----------:|:----------:|:--------:|
| 09:30 - 10:15 | Self-learning lessons discussion | All |
| 10:15 - 10:25 | Break |
| 10:25 - 11:25|  [Normalization and regressing out unwanted variation](../lessons/06_SC_SCT_normalization.md) | Meeta |
| 11:25 - 12:00| [An brief introduction to Integration](../lessons/06_integration.md) | Meeta |

### Before the next class:
I. Please **study the contents** and **work through all the code** within the following lessons:
    1. [Integration cont'd](../lessons/../lessons/06_integration.md)
      <details>
       <summary><i>Click here for a preview of this lesson</i></summary>
         <br>From the UMAP visualization of our data  we can see that the cells are positioned into groups. Our next task is to isolate clusters of cells that are most similar to one another based on gene expression. <br><br>In this lesson you will:<br>
             - Learn the theory behind clustering and how it is performed in Seurat<br>
             - Cluster cells and visualize them on the UMAP<br>
        </details>
   
   1. [Clustering](../lessons/07_SC_clustering_cells_SCT.md)
      <details>
       <summary><i>Click here for a preview of this lesson</i></summary>
         <br>From the UMAP visualization of our data  we can see that the cells are positioned into groups. Our next task is to isolate clusters of cells that are most similar to one another based on gene expression. <br><br>In this lesson you will:<br>
             - Learn the theory behind clustering and how it is performed in Seurat<br>
             - Cluster cells and visualize them on the UMAP<br>
        </details>

   2. [Clustering quality control](../lessons/08_SC_clustering_quality_control.md)
      <details>
       <summary><i>Click here for a preview of this lesson</i></summary>
         <br>After separating cells into clusters, it is crtical to evaluate whether they are biologically meaningful or not. At this point we can also decide if we need to re-cluster and/or potentialy go back to a previous QC step.
         <br><br>In this lesson you will:<br>
           - Check to see that clusters are not influenced by uninteresting sources of variation<br>
           - Check to see whether the major principal components are driving the different clusters<br>
           - Explore the cell type identities by looking at the expression for known markers across the clusters.<br>
        </details>
         
   3. [Marker identification](../lessons/09_merged_SC_marker_identification.md)
      <details>
       <summary><i>Click here for a preview of this lesson</i></summary>
         <br>By this point, you have defined most of your clusters as representative populations of particular cell types. However, there may still some uncertanity and/or unknowns. This step in workflow is about using the gene expression data to identify genes that exhibit a significantly higher (or lower) level of expression for a partcular cluster of cells. <br><br>In this lesson, we idenitfy these lists of genes and use them to:<br>
           - Verify the identity of certain clusters <br>
           - Help surmise the identity of any unknown clusters<br>
        </details>


II. **Submit your work**:
   * Each lesson above contains exercises; please go through each of them.
   * **Submit your answers** to the exercises using [this Google form](https://docs.google.com/forms/d/e/1FAIpQLSd8-k5YwfQekQQdFuDVPhGCy-eK70CqPiUkDNYDeWSEQYBAZg/viewform?usp=sf_link) on **the day *before* the next class**.

### Questions?
* ***If you get stuck due to an error*** while runnning code in the lesson, [email us](mailto:hbctraining@hsph.harvard.edu) 
* Post any **conceptual questions** that you would like to have **reviewed in class** [here](https://PollEv.com/hbctraining945).

***

## Day 3

| Time |  Topic  | Instructor |
|:-----------:|:----------:|:--------:|
| 09:30 - 10:30 | Self-learning lessons discussion | All |
| 10:30 - 10:40 | Workflow summary | Noor |
| 10:40 - 10:45 | Break |
| 10:45 - 11:30 | Discussion, Final Q & A | All |
| 11:30 - 12:00 | [Wrap up](../slides/Workshop_wrapup_in_person.pdf) | Meeta |

***

## Answer Keys

* **[Answer key - assignment #1](../homework/Day1_exercise_answer_key.md)**
* **[Answer key - assignment #2](../homework/Day2_exercise_answer_key.R)**

## Downstream analyses

[Differential expression between conditions](../lessons/pseudobulk_DESeq2_scrnaseq.md)

***

## Resources
We have covered the analysis steps in quite a bit of detail for scRNA-seq exploration of cellular heterogeneity using the Seurat package. For more information on topics covered, we encourage you to take a look at the following resources:

* [Seurat vignettes](https://satijalab.org/seurat/vignettes.html)
* [Seurat cheatsheet](https://satijalab.org/seurat/essential_commands.html)
* [Satija Lab: Single Cell Genomics Day](https://satijalab.org/scgd21/)
* ["Principal Component Analysis (PCA) clearly explained"](https://www.youtube.com/watch?v=_UVHneBUBW0), a video from [Josh Starmer](https://twitter.com/joshuastarmer)
* [Additional information about cell cycle scoring](../lessons/cell_cycle_scoring.md)
* Using RStudio on O2
    * [HMSRC wiki page](https://harvardmed.atlassian.net/wiki/spaces/O2/pages/1623425967/RStudio+on+O2)
    * [HBC RStudio on O2 tutorial](https://hbctraining.github.io/Intro-to-Unix-QMB/lessons/R_studio_on_02.html)
* Highlighted papers for sample processing steps (pre-sequencing):
  - ["Sampling time-dependent artifacts in single-cell genomics studies."](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-020-02032-0) *Massoni-Badosa et al.* 2019
  - ["Dissociation of solid tumor tissues with cold active protease for single-cell RNA-seq minimizes conserved collagenase-associated stress responses."](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-019-1830-0) *O'Flanagan et al.* 2020
  - ["Systematic assessment of tissue dissociation and storage biases in single-cell and single-nucleus RNA-seq workflows."](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-020-02048-6) *Denisenko et al.* 2020
* ["Confronting false discoveries in single-cell differential expression", _Nature Communications_ 2021](https://www.nature.com/articles/s41467-021-25960-2)
* [Azimuth reference-based analysis](https://azimuth.hubmapconsortium.org/)
* [CellMarker resource](http://biocc.hrbmu.edu.cn/CellMarker/)
* Highlighted papers for single-nuclei RNA-seq:
  - [Single-nucleus and single-cell transcriptomes compared in matched cortical cell types](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6306246/)
  - [A single-cell and single-nucleus RNA-Seq toolbox for fresh and frozen human tumors](https://www.nature.com/articles/s41591-020-0844-1)
* [Ligand-receptor analysis with CellphoneDB](https://www.nature.com/articles/s41576-020-00292-x)
* [Best practices for single-cell analysis across modalities](https://www.nature.com/articles/s41576-023-00586-w)


## Building on this workshop

* Other online scRNA-seq courses:
  - [http://bioconductor.org/books/release/OSCA/](http://bioconductor.org/books/release/OSCA/)
  - [https://liulab-dfci.github.io/bioinfo-combio/](https://liulab-dfci.github.io/bioinfo-combio/)
  - [https://hemberg-lab.github.io/scRNA.seq.course/](https://www.singlecellcourse.org/)
  - [https://github.com/SingleCellTranscriptomics](https://github.com/SingleCellTranscriptomics)
  - [https://broadinstitute.github.io/2020_scWorkshop/](https://broadinstitute.github.io/2020_scWorkshop/)
  - [https://nbisweden.github.io/workshop-scRNAseq/schedule.html](https://nbisweden.github.io/workshop-scRNAseq/schedule.html)

* Resources for scRNA-seq Sample Prep:
  - [https://www.protocols.io/](https://www.protocols.io/)
  - [https://support.10xgenomics.com/single-cell-gene-expression/sample-prep](https://support.10xgenomics.com/single-cell-gene-expression/sample-prep)
  - [https://community.10xgenomics.com/](https://community.10xgenomics.com/)

## Other helpful links
* [Online learning resources](https://hbctraining.github.io/bioinformatics_online/lists/online_trainings.html)
* [All hbctraining training materials](https://hbctraining.github.io/main)


