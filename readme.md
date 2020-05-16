Single cell RNA-seq data analysis
=======================

An analysis of four samples of Small Cell Lung Cancer (SCLC) from patient autopsy.


Project Objectives:
-----------

- assaying patient samples from both primary tumor and distal metastasis after rapid autopsy collection of samples;
- assess the heterogeneity of NAPY sub-classification (sub-classification system in SCLC);
- analyze Intratumoral Heterogeneity of SCLC subgroups; 
- analyze potential groups of interest (groups that have changed based on therapy; 
- analyze general state of immune infiltrate in patient samples

Methods and Samples
-----------

- Collecting samples from patients after Rapid Autopsy, with dissociation and capture as soon as physically possible after patients pass – 4 samples currently sequenced, more coming. 
- 9 reference cell lines spread across the 4 NAPY subgroups of SCLC.
- using single cell sequencing of known lines as "reference points" for patient samples already collected, and patient samples collected moving forward.
- NAPY subgroups refer to dominance of the transcription factors Neurod1, ASCL1, Pou2F3 and YAP1 – all cell lines captured, not sequenced yet


Analysis main steps:
-----------

1. pre-processing (performed on Biowulf HPC)

- Filtering out cells based on mean absolute deviation (MAD) of nCount, nFeature, pct.mitochondria: above 3
- Dimension reduction (PCA, URD), 
- Clustering (Smart Local Moving) with resolution 0.2-1.2, 
- Doublet removal(doubletFinder_v3)
- cell identity annotation with "monaco_main","BP_encode_main","Novershtern_main","HPCA_main", "dice_main", "HumanLung.P3" (singleR)
- Integrate batches (four samples)


2. Assessing cluster separation and batch-correction
-----------

- The four samples (merged/integrated object) were clustered with resolution 0.4,0.6,0.8,1,1.2. Silhouette plot was used to assess the cluster seperation. The silhouette width for each cell indicates  difference between distances across all clusters, as well as the average distance to cells in the same cluster. Cells with large positive silhouette widths are closer to other cells in the same cluster than to cells in different clusters. Resolution of 0.6 has the highest average silhouette width, thus was choosen for the subsequent analysis

- Well mixture of small anchor populations (e.g cluster4, connective tissues) in the pre-batch corrected tSNE plot indicates good integration. In addition, seperation of "SCAF1355_6", "SCAF1357_6PDC" samples from the other two samples (SCAF1356_16 and SCAF1358_16PDC) is also expected since they are metastatic tumor cells taken from lymph node and liver respectively. Taken together, the subsequent analysis is carried out using 0.6 resolution and merged object (without batch-correction).


3. Assessing cell identity annotation
-----------

- cell identity was annotated with "monaco_main","BP_encode_main","Novershtern_main","HPCA_main", "dice_main" (singleR). However none of these databases were specific enough for the SCLC. Therefore human lung cell atlas was downloaded from Chan-Zuckerberg Biohub and used to annotate the cell types in the in the datasets. Human lung cell atlas includes fresh blood and lung tissue obtained across the major tissue compartments. It has ~30,000 cells, including all the classical lung cell types (41 of 45, 91%), from the most abundant cell types (e.g. capillary endothelial cells, ~23% of lung cells) down to exceedingly rare ones, with estimated abundances as low as 0.01% (e.g., neuroendocrine cells, ionocytes). SCLC cells have a high mitotic index, display neuroendocrine markers and are believed to derive from NECs or neuroendocrine progenitors (NEPs) in the lung. Achaete-scute homolog 1 (ASCL1) is a neuroendocrine transcription factor. THe majority of the cells (tumor cells) are annotated as neuroendocrine cells, witch makes sense. 


4. Clustering
-----------

- Cluster1,2,3,5,6,7 are mostly neuroendocrine cells which has high level of ASCL1 expression indicating these six clusters are SCLC. 
- Cluster4 are mostly connective tissue cells including fibroblasts and vascular smooth muscle. 
- Cluster8-12 are mostly immune cells. Cluster8 has mostly T cell, Cluster9 dendritic cells, Cluster10 basophil, Cluster11 B cell, Cluster12 T cell. 
- In cluster8-12, there are some cells expressing ASCL1, indicating mixture of small number of SCLC cells in these clusters
- In cluster11, and cluster 12 the number of cells is very low, <50 cells
- transformed RNA expression data for each cluster/sample/met was exported and also plotted on heatmap to show unsupervised cluster/sameple clustering


5. Assessing ASCL1 expression across groups, samples and clusters
-----------

- "SCAF1355_6" and "SCAF1357_6PDC" were grouped as met1(lymph node metastasis) while the other two samples were grouped as met2(liver metastasis).
- DE analysis was carried out between ASCL1 high and ASCL1 low groups. These are met2(ASCL1 high)-met1(ASCL1 low), cluster6(ASCL1 high)-cluster3(ASCL1 low), and 
cluster1(ASCL1 high)-cluster7(ASCL1 low). Over-representation analysis was performed on the DEGs of these ASCL1 high - ASCL1 low contrasts, suing GO BP, KEGG and Reactome gene sets. 


6. GSVA analysis
-----------

Gene Set Variation Analysis (GSVA) estimates variation of pathway activity over a sample population in an unsupervised manner. Here I have used the ssGSEA method from GSVA to calculate enrichment of these following gene sets for each cluster. 

- H: hallmark gene sets
- C2 CP : Canonical pathways (these gene sets are canonical representations of a biological process compiled by domain experts)
- C5 BP: GO gene sets (Gene sets derived from the GO Biological Process Ontology)
- C6: oncogenic signatures (Gene sets that represent signatures of cellular pathways which are often dis-regulated in cancer)
- C7: immunologic signatures (Gene sets that represent cell states and perturbations within the immune system)



Folder structure
-----------

```
ccbr1044_singleCell
├── rawData                  # merged seurat object 
│                            #  
├── analysis                 # analysis folder that contains script and results
│   ├── processedData        # 
|       ├── compositions     # breakdown of cell numbers based on cell identity/clusters/samples/condion etc.
|       ├── expression       # expression level of genes based on cell identity/clusters/samples/condion etc.
|       ├── geneSets         # gene sets downloaded from Molecular Signatures Database (MSigDB), for GSVA and ORA analysis
│   ├── results              # 
|       ├── UMAPs            # umap of all cells based on cell identity/clusters/samples etc.
|       ├── DEG              # differential expression analysis and volcano plots showing hightly differentially expressed genes
|       ├── GSVA             # overlay of GSVA enrichemnt, with cell compositions by sample, with ASCL1 expression level
|       ├── ORA              # over-representation of pathways in the differentially expressed genes between contrasts
|       ├── silhouettePlot   # silhouette plot was used to assess the cluster seperation based on resolution. 0.6 has the highest average width
│   ├── *.Rmd                # R scripts 
│   ├── *.html               # Rmarkdown html knit file (run Rmd to get html)
├── reports                  # Pdfs and ppts slides for presentation.
├── ref                      # methods and references
├── readme.md                # analysis summary                       
└── ...
```
