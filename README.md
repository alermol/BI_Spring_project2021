# Studying role of rare variants on whole genome sequencing data of Parkinson’s disease (PD) patients

## Aim of the project

Gene-based, genome-wide SKAT and SKAT-O test associations of rare variants based on WGS (Whole Genome Sequencing) data of patients with Parkinson's disease

## Sourse data

All data located in AMP-PD (<https://amp-pd.org>) database (Tier 2) and both accessed and analyzed via cloud-native platform for biomedical researchers Terra (<https://app.terra.bio>)

10247 WGS samples in AMP-PD database

- 3009 cases (Parkinson’s disease)
- 4306 controls (No neurological disorders)
- 2932 other phenotype

All variants in darabase splitted by chromosomes. There are 3 files per each chromosome:

- .bed file (PLINK binary biallelic genotype table)  
  Primary representation of genotype calls at biallelic variants
- .bim file (PLINK extended MAP file)  
  Extended variant information file accompanying a .bed binary genotype table  
  A text file with no header line, and one line per variant with the following six fields:

  - Chromosome code (either an integer, or 'X'/'Y'/'XY'/'MT'; '0' indicates unknown) or name  
  - Variant identifier  
  - Position in morgans or centimorgans (safe to use dummy value of '0')  
  - Base-pair coordinate (1-based; limited to 231-2)  
  - Allele 1 (corresponding to clear bits in .bed; usually minor)  
  - Allele 2 (corresponding to set bits in .bed; usually major)  
  - Allele codes can contain more than one character. Variants with negative bp coordinates are ignored by PLINK  
- .fam file (PLINK sample information file)  
  Sample information file accompanying a .bed binary genotype table.  
  A text file with no header line, and one line per sample with the following six fields:

  - Family ID ('FID')  
  - Within-family ID ('IID'; cannot be '0')  
  - Within-family ID of father ('0' if father isn't in dataset)  
  - Within-family ID of mother ('0' if mother isn't in dataset)  
  - Sex code ('1' = male, '2' = female, '0' = unknown)  
  - Phenotype value ('1' = control, '2' = case, '-9'/'0'/non-numeric = missing data if case/control)  
  

## Workflow

### Cloud environement setup

- 8 CPUs
- 52 Gb RAM
- 500 Gb disk space
- Python 3.7.7
- GATK 4.1.4.1
- R 3.6.3

Additional software for the cloud environment

- plink v1.90b6.9
- GCTA 1.93.2
- ANNOVAR  
  Databases
  - refGene
  - cytoBand
  - ensGene
  - exac03
  - avsnp150
  - dbnsfp41c
  - ljb26_all
  - gnomad211_genome
  - clinvar_20210501
- rvtests 20190205

### Data filtering steps

1. Update sex for all samples  
   Male was encoded as 1 and female as 2 in .fam file of each chromosome using plink.
2. Update phenotype for all samples  
   All samples with 'other' phenotypes was discarded at this stage. Control was encoded as 1 and Case as 2. Phenotype was updated in .fam file of each chromosome using plink.
3. Filter by ancestry  
   All non-european individuals was discarded based on IID (Individual ID) and FID (Family ID) using plink
4. Filter by relatedness  
   GRM (Genetic Relationship Matrix) was calculated for all pair of individuals using GATC and in pairs where individuals have more than 12.5% of shared alleles one of the individuals was discarded using plink
5. Filter by missingness per variant
   Sites with statistically significant difference in missing call frequency between cases and controls (p-value $\le0.0001$)

On the first step of quality filtering samples with other phenotypes was discarded. After that based on data about ancestry of patients non-european individuals was discarded too. Remaining variants was filtered by relatedness. Pairwise distances between all pairs of parients was calculated. From pairs of samples shared more than 12.5% of 
