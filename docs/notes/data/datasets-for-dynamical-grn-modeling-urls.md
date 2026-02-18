# GRN Dataset Download URLs: A Complete Reference Guide

Gene regulatory network (GRN) modeling benefits enormously from high-quality perturbation, multi-omic, and time-series datasets. This comprehensive guide provides **direct download URLs** for 20+ priority datasets across GEO, EBI, Zenodo, and specialty portals—organized for immediate programmatic access.

## Time-series + perturbation datasets

These datasets combine temporal dynamics with genetic or chemical perturbations, providing essential ground truth for causal GRN inference.

### GSE213069 (RENGE) - Human iPSC TF knockouts

**Primary accession:** https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE213069

| Resource | Direct URL |
|----------|------------|
| Supplementary files | `ftp://ftp.ncbi.nlm.nih.gov/geo/series/GSE213nnn/GSE213069/suppl/` |
| Series matrix | `ftp://ftp.ncbi.nlm.nih.gov/geo/series/GSE213nnn/GSE213069/matrix/` |
| SOFT format | `ftp://ftp.ncbi.nlm.nih.gov/geo/series/GSE213nnn/GSE213069/soft/` |
| GitHub (RENGE code) | https://github.com/masastat/RENGE |

### ZSCAPE - Zebrafish perturbed embryos (Nature 2023)

**GEO accession:** GSE202639 | **Portal:** https://cole-trapnell-lab.github.io/zscape/

**AWS S3 direct downloads (no authentication required):**
```bash
# Reference atlas (~1.25M cells)
wget http://trapnell-lab-s3-zscape.s3-website-us-west-2.amazonaws.com/zscape_reference_cds.RDS.gz

# Raw count matrix
wget http://trapnell-lab-s3-zscape.s3-website-us-west-2.amazonaws.com/zscape_reference_raw_counts.mtx.gz

# Cell metadata
wget http://trapnell-lab-s3-zscape.s3-website-us-west-2.amazonaws.com/zscape_reference_cell_metadata.csv.gz

# Genetic perturbation atlas (2M cells, 23 perturbations)
wget http://trapnell-lab-s3-zscape.s3-website-us-west-2.amazonaws.com/zscape_perturb_cds.RDS
wget http://trapnell-lab-s3-zscape.s3-website-us-west-2.amazonaws.com/zscape_perturb_full_raw_counts.mtx.gz
wget http://trapnell-lab-s3-zscape.s3-website-us-west-2.amazonaws.com/zscape_perturb_full_cell_metadata.csv.gz

# Temperature perturbation atlas
wget http://trapnell-lab-s3-zscape.s3-website-us-west-2.amazonaws.com/zscape_temp_cds.rds
```

**GitHub:** https://github.com/cole-trapnell-lab/sdg-zfish

### Mesendoderm multilineage atlas (Scientific Data 2025)

**Publication DOI:** https://doi.org/10.1038/s41467-025-56533-2

This dataset covers days 2-9 of human iPSC differentiation with **WNT/BMP4/VEGF perturbations** across 62,208 cells. Check the companion Scientific Data paper for specific GEO/Zenodo accessions in the Data Availability statement.

---

## Multiome datasets (joint RNA + ATAC)

Joint profiling of transcription and chromatin accessibility enables direct linking of regulatory elements to target genes.

### GSE140203 (SHARE-seq mouse skin)

**Primary accession:** https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE140203

| Resource | Direct URL |
|----------|------------|
| RAW data tarball (7.4 GB) | `ftp://ftp.ncbi.nlm.nih.gov/geo/series/GSE140nnn/GSE140203/suppl/GSE140203_RAW.tar` |
| Supplementary files | `ftp://ftp.ncbi.nlm.nih.gov/geo/series/GSE140nnn/GSE140203/suppl/` |
| SRA Run Selector | https://www.ncbi.nlm.nih.gov/Traces/study/?acc=PRJNA588784 |
| Pipeline (Broad) | https://github.com/broadinstitute/epi-SHARE-seq-pipeline |
| FigR tutorial | https://buenrostrolab.github.io/FigR/articles/FigR_shareseq.html |

### GSE117089 (sci-CAR) - Dexamethasone time-course

**Primary accession:** https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE117089

```bash
# Direct downloads
wget ftp://ftp.ncbi.nlm.nih.gov/geo/series/GSE117nnn/GSE117089/suppl/GSE117089_RAW.tar  # 232.2 MB
wget ftp://ftp.ncbi.nlm.nih.gov/geo/series/GSE117nnn/GSE117089/suppl/GSE117089_README.txt
```

**GitHub:** https://github.com/JunyueC/sci-CAR_analysis  
**Human Cell Atlas:** https://explore.data.humancellatlas.org/projects/0792db34-8047-4e62-802c-9177c9cd8e28

### Drosophila DEAP v2 (atlas.gs.washington.edu)

**Portal:** https://atlas.gs.washington.edu/deap_v2/  
**Download resources:** https://shendure-web.gs.washington.edu/content/members/DEAP_website/public/  
**GitHub:** https://github.com/asddf123789/deap_website  
**Original sci-ATAC-seq:** GEO GSE116240

### GSE194122 (NeurIPS 2021 BMMC) - Multimodal benchmark

**Primary accession:** https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE194122

```bash
# Processed h5ad files (recommended)
wget "ftp://ftp.ncbi.nlm.nih.gov/geo/series/GSE194nnn/GSE194122/suppl/GSE194122_openproblems_neurips2021_cite_BMMC_processed.h5ad.gz"      # 587 MB
wget "ftp://ftp.ncbi.nlm.nih.gov/geo/series/GSE194nnn/GSE194122/suppl/GSE194122_openproblems_neurips2021_multiome_BMMC_processed.h5ad.gz"  # 2.7 GB
```

**AWS S3 (post-competition fragments):**
```bash
aws s3 sync s3://openproblems-bio/public/post_competition/multiome/ ./multiome_fragments/
```

| Resource | URL |
|----------|-----|
| SRA BioProject | PRJNA799242 |
| Open Problems portal | https://openproblems.bio/events/2021-09_neurips/ |
| Publication | https://proceedings.mlr.press/v176/lance22a.html |

### NeurIPS 2022 CD34+ Time-Course (Kaggle) - Time-series multiome

**Primary access:** https://www.kaggle.com/competitions/open-problems-multimodal/data

```bash
# Requires Kaggle API authentication
pip install kaggle
kaggle competitions download -c open-problems-multimodal
```

| Resource | URL |
|----------|-----|
| Competition page | https://www.kaggle.com/competitions/open-problems-multimodal |
| Open Problems portal | https://openproblems.bio/events/2022-08_neurips/ |
| Saturn Cloud notebooks | https://github.com/openproblems-bio/neurips_2022_saturn_notebooks |

**Note:** The 2022 time-course data (~300K CD34+ cells, days 2/3/4/7/10) does not have a separate GEO accession—it is primarily distributed through Kaggle. Contains both CITE-seq and 10x Multiome from 4 donors.

---

## Triple modality datasets

These rare datasets capture RNA, chromatin accessibility, and DNA methylation simultaneously—critical for understanding epigenetic regulation.

### GSE121708 (scNMT-seq gastrulation)

**Primary accession:** https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE121708

| Resource | Direct URL |
|----------|------------|
| **EBI FTP (recommended)** | `ftp://ftp.ebi.ac.uk/pub/databases/scnmt_gastrulation` |
| GEO supplementary | `ftp://ftp.ncbi.nlm.nih.gov/geo/series/GSE121nnn/GSE121708/suppl/` |
| GitHub (main) | https://github.com/rargelaguet/scnmt_gastrulation |
| GitHub (EB analysis) | https://github.com/rargelaguet/scnmt_eb |

**Bioconductor R access:**
```R
# SingleCellMultiModal package
BiocManager::install("SingleCellMultiModal")
library(SingleCellMultiModal)
scNMT("mouse_gastrulation", version = "2.0.0")
```

**Modalities:** RNA expression, CpG methylation, GpC accessibility (NOME-seq)

### Paired-Tag mouse brain - scRNA + 5 histone marks

**Portal:** http://catlas.org/pairedTag  
**Allen Brain Map:** https://knowledge.brain-map.org/data/717LLMGF4TOCC860LH5  
**GitHub:** https://github.com/cxzhu/Paired-Tag  
**NEMO Archive:** https://nemoanalytics.org (RRID:SCR_016152)

Contains **64,849 nuclei** with H3K4me1, H3K4me3, H3K27ac, H3K27me3, H3K9me3 profiles from mouse frontal cortex and hippocampus.

---

## Population genetics eQTL datasets

Large-scale population studies link genetic variants to gene expression, providing statistical ground truth for regulatory relationships.

### GSE196830 (OneK1K) - 982 donors, 1.27M PBMCs

**Primary accession:** https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE196830

**AWS S3 eQTL summary statistics (public bucket):**
```bash
# All cell types
wget https://onek1k.s3.ap-southeast-2.amazonaws.com/eqtl/eqtl_table.tsv.gz
wget https://onek1k.s3.ap-southeast-2.amazonaws.com/esnp/esnp_table.tsv.gz

# Cell-type specific (examples)
wget https://onek1k.s3.ap-southeast-2.amazonaws.com/eqtl/cd4nc_eqtl_table.tsv.gz
wget https://onek1k.s3.ap-southeast-2.amazonaws.com/eqtl/cd8nc_eqtl_table.tsv.gz
wget https://onek1k.s3.ap-southeast-2.amazonaws.com/eqtl/nk_eqtl_table.tsv.gz
wget https://onek1k.s3.ap-southeast-2.amazonaws.com/eqtl/monoc_eqtl_table.tsv.gz
```

**CZ CELLxGENE:** https://cellxgene.cziscience.com/collections/dde06e0f-ab3b-46be-96a2-a8082383c4a1  
**Zenodo (analysis code):** https://zenodo.org/records/5084396  
**Zenodo (SAIGE-QTL):** https://zenodo.org/records/10811106  
**GitHub:** https://github.com/powellgenomicslab/onek1k_phase1

**Python API access:**
```python
import cellxgene_census
census = cellxgene_census.open_soma()
```

### HipSci DA neurons (Jerber et al.) - 215 donors

**ENA accession:** ERP121676 (open access) | **EGA:** EGAS00001002885 (managed)

**Zenodo (DOI: 10.5281/zenodo.4333872):**
```bash
# Normalized counts by timepoint
wget "https://zenodo.org/records/4333872/files/D11.h5?download=1" -O D11.h5
wget "https://zenodo.org/records/4333872/files/D30.h5?download=1" -O D30.h5
wget "https://zenodo.org/records/4333872/files/D52.h5?download=1" -O D52.h5

# eQTL summary statistics
wget "https://zenodo.org/records/4333872/files/eqtl_summary_stats.tar.gz?download=1"

# All files (39.2 GB)
wget "https://zenodo.org/api/records/4333872/files-archive" -O zenodo_4333872.zip
```

**GitHub:** https://github.com/single-cell-genetics/singlecell_neuroseq_paper/

### HipSci endoderm differentiation (Cuomo et al.)

**ENA:** ERP016000 | **EGA:** EGAS00001002278

**Zenodo (DOI: 10.5281/zenodo.3625024):**
```bash
wget "https://zenodo.org/records/3625024/files/raw_counts.csv.zip?download=1"
wget "https://zenodo.org/records/3625024/files/log_normalised_counts.csv.zip?download=1"
wget "https://zenodo.org/records/3625024/files/cell_metadata_cols.tsv?download=1"
```

**HipSci FTP:** `ftp://ftp.hipsci.ebi.ac.uk/vol1/ftp`  
**GitHub:** https://github.com/single-cell-genetics/singlecell_endodiff_paper

---

## Genome-scale perturbation screens

These massive CRISPR screens provide systematic perturbation data across thousands of genes.

### Replogle Perturb-seq (gwps.wi.mit.edu) - 9,866 genes

**Portal:** https://gwps.wi.mit.edu  
**SRA BioProject:** PRJNA831566

**Figshare+ (159.71 GB total):**
```bash
# Download all files
wget "https://plus.figshare.com/ndownloader/articles/20029387/versions/1" -O replogle_all_files.zip
```

**scPerturb Zenodo (pre-harmonized h5ad):**
```bash
wget "https://zenodo.org/records/13350497/files/ReplogleWeissman2022_K562_essential.h5ad?download=1"  # 1.5 GB
wget "https://zenodo.org/records/13350497/files/ReplogleWeissman2022_K562_gwps.h5ad?download=1"      # 8.8 GB
wget "https://zenodo.org/records/13350497/files/ReplogleWeissman2022_rpe1.h5ad?download=1"          # 1.2 GB
```

**GitHub:** https://github.com/josephreplogle/guide_calling

### ERP165335 (HipSci CRISPRi) - 7,226 genes, 34 iPSC lines

**ENA:** https://www.ebi.ac.uk/ena/browser/view/PRJEB81502

| Resource | URL |
|----------|-----|
| LFC tables | https://doi.org/10.6084/m9.figshare.26819743 |
| Count data | https://doi.org/10.6084/m9.figshare.27989294 |
| GitHub | https://github.com/claudiafeng123/crispri_scrnaseq_hipsci |
| Project page | https://www.sanger.ac.uk/tool/crispri-scrna-seq-hipsci/ |

### Yeast genome-scale Perturb-seq (Nature Communications 2025)

**Publication:** https://doi.org/10.1038/s41467-025-57600-4

Contains **4,162 mutant strains** from the RNA-barcoded YKOC with ~1 million cells profiled under control and stress (0.4M NaCl) conditions. The RNA-barcoded YKOC library is available upon request from the authors. Check the Data Availability section of the paper for specific GEO accession.

### GSE139944 (sci-Plex) - 188 drugs, 650K cells

**Primary accession:** https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE139944

```bash
# Processed data from GEO (9.2 GB)
wget ftp://ftp.ncbi.nlm.nih.gov/geo/series/GSE139nnn/GSE139944/suppl/GSE139944_RAW.tar

# theislab h5ad version
wget https://ndownloader.figshare.com/files/33979517 -O sciplex_raw.h5ad
```

**GitHub:** https://github.com/cole-trapnell-lab/sci-plex

---

## Lineage ground truth datasets

Datasets with lineage tracing provide explicit cell fate relationships for validating trajectory inference.

### Weinreb LARRY hematopoiesis

**GitHub:** https://github.com/AllonKleinLab/paper-data  
**GEO:** GSE140802

```bash
# In vitro data
wget "https://raw.githubusercontent.com/AllonKleinLab/paper-data/master/Lineage_tracing_on_transcriptional_landscapes_links_state_to_fate_during_differentiation/in_vitro/stateFate_inVitro_normed_counts.mtx.gz"
wget "https://raw.githubusercontent.com/AllonKleinLab/paper-data/master/Lineage_tracing_on_transcriptional_landscapes_links_state_to_fate_during_differentiation/in_vitro/stateFate_inVitro_cell_metadata.txt.gz"
wget "https://raw.githubusercontent.com/AllonKleinLab/paper-data/master/Lineage_tracing_on_transcriptional_landscapes_links_state_to_fate_during_differentiation/in_vitro/stateFate_inVitro_clone_matrix.mtx.gz"

# In vivo data
wget "https://raw.githubusercontent.com/AllonKleinLab/paper-data/master/Lineage_tracing_on_transcriptional_landscapes_links_state_to_fate_during_differentiation/in_vivo/stateFate_inVivo_normed_counts.mtx.gz"
```

**scPerturb version:**
```bash
wget "https://zenodo.org/records/13350497/files/WeinrebKlein2020.h5ad?download=1"  # 228.5 MB
```

### C. elegans embryogenesis (Zenodo)

**DOI:** 10.5281/zenodo.15236812 | **GEO:** GSE126954

```bash
wget "https://zenodo.org/records/15236812/files/celegans_global_adata.h5ad?download=1"  # 710.7 MB
```

**VisCello tool:** https://github.com/qinzhu/VisCello.celegans

---

## Dense temporal datasets

High temporal resolution captures regulatory dynamics during rapid developmental transitions.

### E-MTAB-6967 (Pijuan-Sala mouse gastrulation)

**ArrayExpress:** https://www.ebi.ac.uk/biostudies/arrayexpress/studies/E-MTAB-6967

| Resource | URL |
|----------|-----|
| ArrayExpress FTP | `ftp://ftp.ebi.ac.uk/pub/databases/arrayexpress/data/experiment/MTAB/E-MTAB-6967/` |
| BioStudies FTP | `ftp://ftp.ebi.ac.uk/biostudies/fire/E-MTAB-/967/E-MTAB-6967/Files/` |
| GitHub (Marioni Lab) | https://github.com/MarioniLab/EmbryoTimecourse2018 |
| Interactive browser | https://marionilab.cruk.cam.ac.uk/MouseGastrulation2018/ |

**Bioconductor:**
```R
BiocManager::install("MouseGastrulationData")
library(MouseGastrulationData)
```

**116,312 cells** from E6.5-E8.5 mouse embryos (9 timepoints).

### iPSC cardiac differentiation (eLife 2023)

**Publication:** https://doi.org/10.7554/eLife.80075

12 consecutive days of cardiac differentiation across 2 iPSC lines (WTC-11 and SCVI-111), with 27,595 cells. Check the Figure 4—source data and Data Availability section for GEO accessions.

---

## Mouse Mutant Cell Atlas (MMCA)

**Portal:** https://atlas.gs.washington.edu/mmca_v2/  
**Download page:** https://atlas.gs.washington.edu/mmca_v2/public/download.html  
**GitHub:** https://github.com/shendurelab/MMCA/

Contains **1.67 million nuclei** from 103 E13.5 embryos across 22 mutant strains + 4 wild-type backgrounds.

---

## scPerturb aggregation resource

**Portal:** https://scperturb.org  
**Zenodo DOI:** 10.5281/zenodo.13350497 (43.0 GB total, 53 h5ad files)

```bash
# Download entire collection
wget "https://zenodo.org/api/records/13350497/files-archive" -O scperturb_all.zip

# Individual datasets (examples)
wget "https://zenodo.org/records/13350497/files/NormanWeissman2019_filtered.h5ad?download=1"  # 698.7 MB
wget "https://zenodo.org/records/13350497/files/SrivatsanTrapnell2020_sciplex3.h5ad?download=1"  # 2.5 GB
wget "https://zenodo.org/records/13350497/files/JoungZhang2023_atlas.h5ad?download=1"  # 5.8 GB
```

**scPerturb ATAC data (Zenodo 7058382):**
```bash
wget "https://zenodo.org/api/records/7058382/files-archive" -O scperturb_atac.zip  # 6.9 GB
```

**GitHub:** https://github.com/sanderlab/scPerturb

---

## Programmatic access patterns

### GEO FTP structure
```bash
# General pattern
ftp://ftp.ncbi.nlm.nih.gov/geo/series/GSEnnn/GSExxxxx/suppl/
ftp://ftp.ncbi.nlm.nih.gov/geo/series/GSEnnn/GSExxxxx/matrix/

# SRA Toolkit for raw reads
prefetch SRRxxxxxxx
fasterq-dump SRRxxxxxxx
```

### EBI FTP patterns
```bash
# ArrayExpress
ftp://ftp.ebi.ac.uk/pub/databases/arrayexpress/data/experiment/MTAB/E-MTAB-XXXX/

# ENA fastq
ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERRxxx/ERRxxxxxx/
```

### Zenodo direct download
```bash
# Pattern: https://zenodo.org/records/{record_id}/files/{filename}?download=1
# Archive: https://zenodo.org/api/records/{record_id}/files-archive
```

### Cloud resources

**NCBI SRA on AWS:** Data accessible via SRA Toolkit with AWS backend  
**CellxGene Census (AWS S3 us-west-2):** Python/R API access via `cellxgene-census` package  
**OneK1K eQTL (AWS S3 ap-southeast-2):** Direct public bucket URLs listed above

## Conclusion

This reference consolidates **direct download URLs** for GRN-relevant datasets across major archives. Key takeaways: **Zenodo** provides the most reliable direct h5ad downloads for processed data; **scPerturb** serves as an excellent aggregator for harmonized perturbation datasets; **specialty portals** (ZSCAPE, MMCA, gwps.wi.mit.edu) often host processed data on AWS S3 with no authentication required. For raw sequencing data, GEO and ENA FTP servers remain the primary sources, accessible via standard `wget`/`curl` commands or dedicated tools like SRA Toolkit.