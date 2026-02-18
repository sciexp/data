# Publicly available datasets for dynamical GRN modeling

Single-cell datasets with perturbations, time-series data, and multimodal profiling are increasingly available for gene regulatory network inference. **Over 80 datasets** across GEO, Human Cell Atlas, ENCODE, and specialized portals meet the criteria for dynamical systems modeling analogous to GSE213069 (RENGE). The most valuable resources combine temporal sampling with either genetic perturbations or chromatin accessibility—enabling causal regulatory inference rather than correlational analysis alone.

---

## Perturbation datasets: the gold standard for causal GRN inference

Time-series perturbation experiments provide the strongest foundation for dynamical GRN modeling because they capture both regulatory topology and kinetics.

### GSE213069 — RENGE time-series scCRISPR (reference dataset)
- **Organism/cells**: Human iPSCs (OILG-3 line)
- **Modalities**: scRNA-seq with CRISPR-Cas9 knockouts
- **Perturbations**: 23 transcription factors (POU5F1, SOX2, NANOG, PRDM14)
- **Time-series**: Days 2, 3, 4, 5 post-transduction (4 timepoints)
- **Scale**: ~5,000 cells/sample, ~100 cells/gRNA
- **Genetic variants**: No population-level variants
- **Publication**: Ishikawa et al., Communications Biology 2023
- **GRN suitability**: Purpose-built for time-resolved GRN inference; directly benchmarks RENGE method

### GSE90063 — Original Perturb-seq with temporal sampling
- **Organism/cells**: Mouse BMDCs; Human K562
- **Modalities**: scRNA-seq with CRISPR knockouts
- **Perturbations**: 10–24 TFs per experiment
- **Time-series**: 0h, 3h post-LPS stimulation (BMDCs); 7d, 14d (K562)
- **Scale**: ~200,000 cells total
- **Genetic variants**: No
- **Publication**: Dixit et al., Cell 2016
- **GRN suitability**: Foundational dataset for Perturb-seq; immune response dynamics with defined perturbations

### Genome-scale Perturb-seq (Replogle et al.)
- **Accession**: Figshare/gwps.wi.mit.edu; GEO GSE168620
- **Organism/cells**: Human K562, RPE1
- **Modalities**: scRNA-seq with CRISPRi (dCas9-KRAB)
- **Perturbations**: **9,866 expressed genes** (genome-scale)
- **Time-series**: Single timepoint (day 6–8)
- **Scale**: **>2.5 million cells**
- **Genetic variants**: Cell line background
- **Publication**: Replogle et al., Cell 2022
- **GRN suitability**: Comprehensive TF-target relationships; enables systematic network reconstruction at genome scale

### GSE139944 — sci-Plex chemical transcriptomics
- **Organism/cells**: Human A549, K562, MCF7 cancer lines
- **Modalities**: scRNA-seq (sci-RNA-seq3)
- **Perturbations**: **188 small molecules** across ~5,000 conditions
- **Time-series**: 24h and 48h post-treatment
- **Scale**: ~650,000 cells
- **Genetic variants**: Cell line characterization
- **Publication**: Srivatsan et al., Science 2020
- **GRN suitability**: Drug-gene regulatory relationships; dose-response modeling

### GSE120861 — Gasperini enhancer CRISPRi screen
- **Organism/cells**: Human K562
- **Modalities**: scRNA-seq with CRISPRi targeting enhancers
- **Perturbations**: **5,779 candidate enhancers**; 78,562 enhancer-gene pairs tested
- **Scale**: ~207,324 cells (high MOI)
- **Time-series**: No
- **Genetic variants**: No
- **Publication**: Gasperini et al., Cell 2019
- **GRN suitability**: Direct enhancer-gene causal links; regulatory element validation

### scPerturb harmonized database
- **URL**: https://scperturb.org
- **Content**: **44 harmonized perturbation datasets**, >2.5M cells
- **Modalities**: 32 CRISPR datasets, 9 drug datasets, 3 scATAC-seq
- **Publication**: Peidli et al., Nature Methods 2024
- **GRN suitability**: Standardized format enables cross-dataset GRN comparison; unified preprocessing

---

## Multiome datasets: chromatin accessibility enables regulatory inference

Joint scRNA-seq + scATAC-seq profiling reveals which regulatory elements control gene expression in the same cells—critical for mechanistic GRN models.

### GSE140203 — SHARE-seq mouse skin (hair follicle differentiation)
- **Organism/cells**: Mouse skin (keratinocytes, dermal papilla, melanocytes)
- **Modalities**: **scRNA-seq + scATAC-seq** (same cell)
- **Time-series**: Yes—hair follicle differentiation trajectory
- **Scale**: 34,774 joint profiles; 84,426 total cells
- **Genetic variants**: No
- **Publication**: Ma et al., Cell 2020
- **GRN suitability**: **Outstanding**—introduces "chromatin potential" showing accessibility precedes expression; 63,110 peak-gene associations; Domains of Regulatory Chromatin (DORCs) overlap super-enhancers

### GSE117089 — sci-CAR with dexamethasone time-course
- **Organism/cells**: Mouse kidney; Human A549
- **Modalities**: **scRNA-seq + scATAC-seq** (combinatorial indexing)
- **Time-series**: **Yes**—A549 treated with dexamethasone at 0h, 1h, 3h
- **Scale**: ~16,000 cells with both modalities
- **Genetic variants**: No
- **Publication**: Cao et al., Science 2018
- **GRN suitability**: **Excellent**—time-resolved chromatin-expression coupling; 2,613 DE genes linked to 4,763 differentially accessible sites

### SNARE-seq mouse brain
- **Organism/cells**: Mouse neonatal and adult cerebral cortex
- **Modalities**: **scRNA-seq + scATAC-seq** (droplet-based)
- **Time-series**: Developmental pseudotime (neurogenesis)
- **Scale**: ~15,000 nuclei
- **Genetic variants**: No
- **Publication**: Chen et al., Nature Biotechnology 2019
- **GRN suitability**: Cell-type-specific chromatin landscapes during neuronal differentiation

### GSE109262/GSE121708 — scNMT-seq (triple modality)
- **Organism/cells**: Mouse ESCs; Mouse gastrulation
- **Modalities**: **scRNA-seq + chromatin accessibility + DNA methylation**
- **Time-series**: ESC differentiation; gastrulation stages
- **Scale**: ~150 cells (ESC); larger for gastrulation
- **Genetic variants**: No
- **Publication**: Clark et al., Nature Communications 2018; Argelaguet et al., Nature 2019
- **GRN suitability**: **Exceptional**—three regulatory layers in same cells; reveals methylation-accessibility-transcription coupling dynamics

### Paired-Tag mouse brain (5 histone marks)
- **Organism/cells**: Mouse frontal cortex and hippocampus
- **Modalities**: **scRNA-seq + H3K4me1/H3K4me3/H3K27ac/H3K27me3/H3K9me3**
- **Time-series**: Multiple developmental stages
- **Scale**: ~70,000 nuclei (64,849 QC-passed)
- **Genetic variants**: No
- **Publication**: Zhu et al., Nature Methods 2021
- **GRN suitability**: Five histone marks reveal active vs. repressive states; comprehensive chromatin state + gene regulation

### GSE207308 — SHARE-seq human bone marrow
- **Organism/cells**: Human BMMC
- **Modalities**: scRNA-seq + scATAC-seq
- **Time-series**: Differentiation trajectories inferred
- **Scale**: 78,520 cells; 51,862 genes; 173,026 chromatin regions
- **Genetic variants**: No
- **GRN suitability**: Large-scale human multiome; comprehensive enhancer-gene coverage

### GSE194122 — NeurIPS 2021 BMMC multimodal atlas
- **Organism/cells**: Human bone marrow mononuclear cells (BMMCs)
- **Modalities**: **CITE-seq** (RNA + 134 proteins); **10x Multiome** (RNA + ATAC, joint)
- **Time-series**: No (cross-sectional)
- **Scale**: ~90,000 CITE-seq cells; ~69,249 Multiome cells
- **Donors**: 12 healthy donors across 4 sites
- **Genetic variants**: Limited (12 donors)
- **Publication**: Luecken et al., NeurIPS 2021
- **GRN suitability**: Benchmarking standard for multimodal integration; joint RNA+ATAC enables enhancer-gene linkage; nested batch design tests robustness

### NeurIPS 2022 CD34+ HSPC time-course (Kaggle)
- **Organism/cells**: Human CD34+ hematopoietic stem and progenitor cells
- **Modalities**: **CITE-seq** (RNA + protein); **10x Multiome** (RNA + ATAC)
- **Time-series**: **Days 2, 3, 4, 7, 10** (5 timepoints over 10-day differentiation)
- **Scale**: **~300,000 cells**
- **Donors**: 4 (IDs: 13176, 27678, 31800, 32606)
- **Cell types**: HSC, EryP, BP, NeuP, MkP, MasP, MoP
- **Genetic variants**: Limited (4 donors)
- **Publication**: Cellarity/Open Problems, NeurIPS 2022
- **GRN suitability**: **Excellent**—rare time-series multiome; captures HSPC differentiation dynamics; test set from unseen timepoint enables temporal extrapolation validation

### 10x Genomics Multiome datasets
- **PBMC reference**: ~10,000 cells, human immune types
- **Mouse brain aging (GSE294772)**: 2, 9, 18 months across 8 brain regions
- **Alzheimer's disease mouse model**: Disease progression time-course
- **Access**: 10x Genomics website and GEO
- **GRN suitability**: Commercial platform with high data quality; aging dataset provides **3-timepoint longitudinal data**

---

## Time-series developmental and differentiation datasets

Dense temporal sampling during biological processes enables dynamical systems approaches without requiring perturbations.

### E-MTAB-6967 — Pijuan-Sala mouse gastrulation atlas
- **Organism/cells**: Mouse E6.5–E8.5 (extended to E9.5)
- **Modalities**: scRNA-seq; Tal1−/− chimeras for perturbation
- **Time-series**: **9 timepoints at 6-hour intervals** (extended: 13 timepoints)
- **Scale**: 116,312 cells (original); **430,339 cells** (extended)
- **Genetic variants**: Tal1 knockout chimeras included
- **Publication**: Pijuan-Sala et al., Nature 2019
- **GRN suitability**: **Gold standard** for mouse development; Bioconductor package (MouseGastrulationData); highest temporal resolution available

### Weinreb hematopoiesis with LARRY lineage tracing
- **Accession**: github.com/AllonKleinLab/paper-data
- **Organism/cells**: Mouse bone marrow HSPCs
- **Modalities**: scRNA-seq + clonal lineage barcodes
- **Time-series**: Days 2, 4, 6 in vitro + in vivo transplantation
- **Scale**: ~130,000 cells; 49,297 with barcodes; 5,864 unique clones
- **Genetic variants**: Clonal identity provides "natural perturbations"
- **Publication**: Weinreb et al., Science 2020
- **GRN suitability**: **Benchmark standard** for fate prediction; ground-truth lineage enables validation; used for PRESCIENT dynamical modeling

### GSE75748 — Human ESC to definitive endoderm
- **Organism/cells**: Human H1 ESC
- **Modalities**: scRNA-seq + matched bulk RNA-seq
- **Time-series**: **0h, 12h, 24h, 36h, 72h, 96h** (6 densely sampled points)
- **Scale**: 1,776 cells
- **Genetic variants**: No
- **Publication**: Chu et al., Genome Biology 2016
- **GRN suitability**: Classic benchmark; includes Wave-Crest trajectory; regulators validated by CRISPR/Cas9

### iPSC cardiac differentiation (12 timepoints)
- **Accession**: eLife 2023 publication data
- **Organism/cells**: Human iPSC (WTC, SCVI-111 lines)
- **Modalities**: scRNA-seq + TBX5/MYL2 lineage reporters
- **Time-series**: **12 consecutive days** during cardiac differentiation
- **Scale**: 27,595 cells
- **Genetic variants**: Two iPSC lines
- **Publication**: eLife 2023
- **GRN suitability**: Densest temporal sampling for cardiac differentiation; lineage tracing validates fate predictions

### Mesendoderm multilineage atlas with perturbations
- **Accession**: Scientific Data 2025
- **Organism/cells**: Human iPSC (WTC CRISPRi GCaMP)
- **Modalities**: scRNA-seq
- **Time-series**: Days 2–9 (8 consecutive days)
- **Perturbations**: **WNT, BMP4, VEGF modulators**
- **Scale**: Multiple conditions
- **GRN suitability**: **Excellent**—combines time-series with signaling perturbations; ideal for causal GRN inference

### Human reprogramming with paired scATAC-seq
- **Accession**: Science Advances 2020
- **Organism/cells**: Human BJ fibroblasts → iPSC
- **Modalities**: **scRNA-seq + scATAC-seq** at matched timepoints
- **Time-series**: Day 0, 2, 8, 16+, H1 ESC reference
- **Scale**: Fluidigm + 10X libraries
- **Genetic variants**: No
- **Publication**: Science Advances 2020
- **GRN suitability**: Multi-omic time-series during reprogramming; identifies TF regulatory phasing

### Mouse organogenesis (MOCA)
- **Organism/cells**: Mouse E9.5–E13.5
- **Modalities**: scRNA-seq (sci-RNA-seq3)
- **Time-series**: E9.5, E10.5, E11.5, E12.5, E13.5 (5 stages)
- **Scale**: **~2 million cells**
- **Genetic variants**: No
- **Publication**: Cao et al., Nature 2019
- **GRN suitability**: Massive scale covers critical organogenesis window; 38 major cell types

---

## Datasets with genetic variant information

Natural genetic variation or defined mutations enable causal inference by linking genotype to expression.

### OneK1K — Population-scale single-cell eQTLs
- **Accession**: GSE196830; onek1k.org
- **Organism/cells**: Human PBMCs from **982 donors**
- **Modalities**: scRNA-seq
- **Time-series**: Dynamic eQTLs during B-cell naïve→memory transition
- **Scale**: **1.27 million cells**; 14 immune cell types
- **Genetic variants**: **26,597 independent cis-eQTLs**; 990 trans-eQTLs; full genotyping
- **Publication**: Yazar et al., Science 2022
- **GRN suitability**: **Gold standard for sc-eQTL**; causal SNP→gene effects enable directional GRN edges; 38.4% of GWAS colocalizations are cell-type specific

### HipSci dopaminergic neuron differentiation
- **Accession**: Nature Genetics 2021
- **Organism/cells**: Human iPSCs from **215 donors** → DA neurons
- **Modalities**: scRNA-seq
- **Time-series**: Day 11 (progenitors), Day 30 (young neurons), Day 52 (mature) + rotenone stress
- **Scale**: **>1 million cells**
- **Genetic variants**: Full genotyping; stage-specific eQTLs; **1,284 eQTLs colocalize with neurological GWAS**
- **Perturbations**: Rotenone oxidative stress
- **Publication**: Jerber et al., Nature Genetics 2021
- **GRN suitability**: **Outstanding**—time-series + population-scale genetics + stress perturbation; disease-relevant context

### HipSci endoderm differentiation
- **Accession**: ArrayExpress, hipsci.org
- **Organism/cells**: Human iPSCs from **125 donors** → definitive endoderm
- **Modalities**: scRNA-seq
- **Time-series**: Day 0, Day 1, Day 3
- **Scale**: ~37,000 cells
- **Genetic variants**: Full genotyping; 1,833 eGenes at iPSC stage; dynamic eQTLs
- **Publication**: Cuomo et al., Nature Communications 2020
- **GRN suitability**: Dynamic eQTLs reveal temporal gene regulation; context-specific regulatory effects

### HipSci CRISPRi screen with genetic backgrounds
- **Accession**: ERP165335; Sanger preprint 2024
- **Organism/cells**: Human iPSCs from **26 donors**
- **Modalities**: scRNA-seq + CRISPRi
- **Perturbations**: **7,226 genes targeted**
- **Genetic variants**: Full genotyping enables genetic interaction studies
- **GRN suitability**: **Exceptional**—perturbation × genetic background = powerful causal inference; regulatory networks directly testable

### CCLE pan-cancer single-cell collection
- **Accession**: DepMap Portal; Single Cell Portal SCP542
- **Organism/cells**: Human, **198 cancer cell lines** from 22 tumor types
- **Modalities**: scRNA-seq
- **Time-series**: No
- **Scale**: ~53,500 cells
- **Genetic variants**: Full CCLE characterization (WES, WGS for 329 lines, CNV, mutations)
- **Publication**: Kinker et al., Nature Genetics 2020
- **GRN suitability**: CNV, epigenetic variation, and ecDNA linked to expression heterogeneity; 12 recurrent programs identified

### Asian Immune Diversity Atlas (AIDA)
- **Organism/cells**: Human PBMCs from **474 Asian donors**
- **Modalities**: scRNA-seq
- **Time-series**: 107 dynamic sQTLs identified
- **Scale**: ~1 million cells
- **Genetic variants**: **11,577 cis-sQTLs** (splicing); 607 trans-sGenes; ancestry-specific variants
- **Publication**: Nature Genetics 2024
- **GRN suitability**: Splicing QTLs add post-transcriptional regulatory layer

---

## Model organism datasets with comprehensive genetic characterization

Model organisms offer complete genetic libraries and well-annotated genomes enabling systematic perturbation studies.

### Yeast genome-scale Perturb-seq atlas
- **Organism/cells**: S. cerevisiae, log-phase
- **Modalities**: scRNA-seq with RNA-traceable clone barcodes
- **Perturbations**: **4,162 mutant strains** (82% of YKOC); ~3,500 mutants profiled
- **Time-series**: Control vs. osmotic stress (0.4M NaCl, 15 min)
- **Scale**: ~1,061,865 cells
- **Genetic variants**: Genome-scale deletion library
- **Publication**: Sameith et al., Nature Communications 2025
- **GRN suitability**: **Outstanding**—systematic genotype-to-transcriptome mapping; causal inference for GRN construction

### GSE136049 — CeNGEN (C. elegans neuronal atlas)
- **Organism/cells**: C. elegans nervous system
- **Modalities**: scRNA-seq + bulk RNA-seq (160 samples) + alternative splicing
- **Time-series**: L1→L4→Adult developmental stages
- **Scale**: 100,955 cells; 70,296 neurons; **128 neuronal classes**
- **Genetic variants**: Complete nervous system annotation; WormBase integration
- **Publication**: Taylor et al., Cell 2021
- **GRN suitability**: Complete gene expression map of entire nervous system; links to known connectome; TF motif analysis (FIRE)

### C. elegans embryogenesis lineage atlas
- **Accession**: Zenodo 10.5281/zenodo.15236812
- **Organism/cells**: C. elegans embryo
- **Modalities**: scRNA-seq with lineage resolution
- **Time-series**: Fertilization through 500 minutes; **502 terminal cell types**
- **Scale**: 86,024 cells; 1,068 of 1,228 lineage branches covered
- **Genetic variants**: **Invariant cell lineage fully mapped**
- **Publication**: Packer et al., Science 2019
- **GRN suitability**: **Gold standard** for lineage-coupled transcriptomics; multilineage priming at branch points; exact cell position known

### Drosophila embryogenesis continuum (multiome)
- **Accession**: atlas.gs.washington.edu/deap_v2
- **Organism/cells**: Drosophila embryo
- **Modalities**: **scRNA-seq + scATAC-seq** (matched samples)
- **Time-series**: **11 overlapping time windows** (0–20 hours); 2-hour resolution early
- **Scale**: ~1.5 million nuclei (~500K scRNA, ~1M scATAC)
- **Genetic variants**: Well-annotated dm6 genome
- **Publication**: Calderon et al., Science 2022
- **GRN suitability**: **Excellent**—multi-omics enables direct enhancer-gene linkage; computed gene activity scores; TF motif activity

### Zebrafish perturbed embryos atlas
- **Organism/cells**: Zebrafish embryos
- **Modalities**: scRNA-seq (sci-RNA-seq3)
- **Time-series**: **19 timepoints** (18 hpf–96 hpf); 2-hour resolution until 48 hpf
- **Scale**: ~3.2 million cells; 1,812 embryos
- **Genetic variants**: **23 defined genetic knockouts/perturbations**
- **Publication**: Saunders et al., Nature 2023
- **GRN suitability**: **Excellent**—purpose-built for perturbation effects; individual embryo barcoding; cell type composition changes quantified

### Zebrahub multimodal atlas
- **URL**: zebrahub.sf.czbiohub.org
- **Organism/cells**: Zebrafish (10 hpf–10 dpf)
- **Modalities**: scRNA-seq + epigenomics + **live light-sheet imaging** (lineage)
- **Time-series**: 10 developmental stages
- **Scale**: 120,000+ cells across 40 single-embryo transcriptomes
- **Genetic variants**: Individual embryo resolution
- **Publication**: CZ Biohub, Cell 2024
- **GRN suitability**: Image-based lineage trees combined with transcriptomes; NMP pluripotency analysis

### Mouse Mutant Cell Atlas (MMCA)
- **Organism/cells**: Mouse E13.5 whole embryos
- **Modalities**: scRNA-seq (sci-RNA-seq3)
- **Perturbations**: **22 defined mutations** (knockouts, knock-ins, enhancer deletions) + 4 WT strains
- **Scale**: ~1.6 million nuclei; 103 embryos
- **Genetic variants**: Defined genetic perturbations
- **Publication**: Cao et al., Nature 2023
- **GRN suitability**: Systematic developmental disorder phenotyping; enhancer perturbations included

### Diversity Outbred mouse scRNA-seq
- **Organism/cells**: Mouse bone marrow stromal cells
- **Modalities**: scRNA-seq with genetic variant integration
- **Scale**: 80 DO mice profiled
- **Genetic variants**: **>6 million sequence variants** (comparable to human GWAS); GigaMUGA genotyping
- **Publication**: Calabrese et al., JBMR/eLife 2023–2024
- **GRN suitability**: Population-level scRNA-seq enables genetic effect estimation; BMD GWAS integration

---

## ENCODE and regulatory reference datasets

Gold-standard regulatory annotations complement scRNA-seq for mechanistic GRN modeling.

### ENCODE single-cell resources
- **Portal**: encodeproject.org/single-cell
- **Data types**: snATAC-seq, scRNA-seq
- **Organism**: Human and mouse tissues
- **GRN application**: Cell-type-specific regulatory elements; gold-standard TF binding from ChIP-seq

### Roadmap Epigenomics reference
- **URL**: egg2.wustl.edu/roadmap
- **Data types**: 15/25-state ChromHMM; 6 histone marks; DNase; methylation; RNA
- **Scale**: **127 reference epigenomes** across primary cells/tissues
- **Publication**: Nature 2015
- **GRN application**: 2.3 million promoter/enhancer regions; cell-type-specific regulatory modules

### 4D Nucleome Portal
- **URL**: data.4dnucleome.org
- **Data types**: Hi-C, sci-Hi-C, Micro-C, SPRITE, GAM
- **Scale**: >1,800 experiment sets; >19,000 single cells (sci-Hi-C)
- **GRN application**: Enhancer-promoter contacts; long-range chromatin interactions; TAD context

### FANTOM5 enhancer atlas
- **URL**: fantom.gsc.riken.jp/5
- **Data types**: CAGE for TSS/enhancer activity
- **Scale**: ~63,000 human enhancers; ~44,000 mouse enhancers
- **GRN application**: Active enhancer identification via eRNAs; promoter activity maps

### JASPAR 2024 and HOCOMOCO v12
- **URLs**: jaspar.elixir.no; hocomoco.autosome.org
- **Content**: 1,920+ TF binding profiles (JASPAR); 949 human + 720 mouse TF models (HOCOMOCO)
- **GRN application**: Essential for TF binding site prediction from ATAC-seq peaks

---

## Benchmark datasets for GRN method validation

### BEELINE benchmark
- **Zenodo**: doi.org/10.5281/zenodo.3378975
- **GitHub**: github.com/Murali-group/Beeline
- **Content**: 6 synthetic networks; 4 Boolean models; 5 experimental scRNA-seq datasets (hHEP, hESC, mESC, mDC, mHSC)
- **Ground truth**: Cell-type-specific ChIP-seq networks
- **Publication**: Nature Methods 2020

### Dictys GRN pipeline datasets
- **Zenodo**: zenodo.org/records/6904917
- **Content**: Human blood (separate scRNA+scATAC); Mouse skin (joint multiome); 10x Multiome examples

---

## Summary: recommended priority datasets by use case

| Use case | Top recommended datasets |
|----------|-------------------------|
| **Time-series + perturbation** | GSE213069 (RENGE), Mesendoderm atlas, Zebrafish perturbed embryos |
| **Multiome (scRNA+scATAC)** | SHARE-seq skin (GSE140203), sci-CAR (GSE117089), Drosophila embryo atlas, NeurIPS 2022 CD34+ time-course |
| **Triple modality** | scNMT-seq gastrulation (GSE121708), Paired-Tag brain |
| **Population-scale genetics** | OneK1K, HipSci DA neurons, HipSci endoderm |
| **Genome-scale perturbation** | Replogle Perturb-seq, HipSci CRISPRi, Yeast Perturb-seq atlas |
| **Lineage ground truth** | Weinreb hematopoiesis (LARRY), C. elegans embryo atlas |
| **Dense temporal sampling** | Pijuan-Sala gastrulation (E-MTAB-6967), iPSC cardiac (12 days), GSE75748 endoderm |
| **Model organism + genetics** | Yeast genome-scale Perturb-seq, Zebrafish perturbed embryos, Mouse MMCA |
| **Drug perturbations** | sci-Plex (GSE139944), MIX-Seq (DepMap) |
| **Regulatory element validation** | Gasperini enhancer CRISPRi (GSE120861), ENCODE ChIP-seq |
| **Method benchmarking** | BEELINE, Dictys datasets, NeurIPS 2021 BMMC (GSE194122) |

The **scPerturb database** (scperturb.org) provides unified access to 44 harmonized perturbation datasets. For multiome integration, the **scMMO-atlas** database aggregates 70 high-quality multiome datasets with 3.2 million cells. CellxGene Census (cellxgene.cziscience.com) enables programmatic filtering across >93 million cells to identify additional developmental/differentiation datasets with the Python or R APIs.