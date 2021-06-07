# Studying role of rare variants on whole genome sequencing data of Parkinson’s disease (PD) patients

## Aim of the project

Gene-based, genome-wide SKAT and SKAT-O test associations of rare variants based on WGS (Whole Genome Sequencing) data of patients with Parkinson's disease

## Sourse data

All data located in AMP-PD (<https://amp-pd.org>) database (Tier 2) and both accessed and analyzed via cloud-native platform for biomedical researchers Terra (<https://app.terra.bio>). Access to database require singing up the AMP-PD Data Use Agreement.

10247 WGS samples in AMP-PD database

- 3009 cases (Parkinson’s disease)
- 4306 controls (No neurological disorders)
- 2932 other phenotype

There are [7 cohorts](https://amp-pd.org/unified-cohorts) in the last release of AMP-PD database.

The last release of AMP-PD database created based on hg38 human genome assembly.

All variants in database splitted by chromosomes. There are 3 files per each chromosome:

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
- GATK 4.1.4.1 (Van der Auwera and O’Connor, 2020)
- R 3.6.3

Additional software need to be installed in the cloud environment

- PLINK v1.90b6.9 (Chang *et al*., 2015)
- GCTA 1.93.2 (Yang *et al*., 2011)
- ANNOVAR  (Wang *et al*., 2010)  
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
- rvtests 20190205 (Zhan *ey al*., 2016)

[Jupyter Notebook](rare_variants_lysosomal_genes.ipynb) contains full pipeline for quality checking and annotation of 8 lysomal genes. Notebook can be directly imported in Terra Cloud Environment and run to replicate all pipeline steps.

### Data filtering steps

1. Update sex for all samples  
   Male was encoded as 1 and female as 2 in .fam file of each chromosome using PLINK.
2. Update phenotype for all samples  
   All samples with 'other' phenotypes was discarded at this stage. Control was encoded as 1 and Case as 2. Phenotype was updated in .fam file of each chromosome using PLINK.
3. Filter by ancestry  
   All non-european individuals was discarded based on IID (Individual ID) and FID (Family ID) using PLINK.
4. Filter by relatedness  
   GRM (Genetic Relationship Matrix) was calculated for all pair of individuals using GCTA and in pairs where individuals have more than 12.5% of shared alleles one of the individuals was discarded using PLINK
5. Filter by missingness per variant  
   Sites with statistically significant difference in missing call frequency between cases and controls (p-value <=0.0001) was discarded using PLINK
6. Filter by missingness per haplotype  
   Variants whose current status can be statistically significant (p-value <=0.0001) predicted based on adjacent sites was discarded using PLINK
7. Filter by HWE (Hardy-Weinberg Equilibrium)  
   Filters out all variants which have Hardy-Weinberg equilibrium exact test p-value below the provided threshold (0.0001) using PLINK (Wigginton *et al*., 2005)

These steps are general QC. Next steps are performs based on aim of study

8. Filter by cohort  
   For this study we took all cohorts except cases from LBD because this cohort contain cases with Lewy Body Dementia. Only Neurologically Healthy Controls from LBD cohort was taken into analisys.
9. Variants extraction  
   Variants of specific genes was extracted using PLINK based on gene coordinate in hg38
10. Annotation  
    Annotation was performed using ANNOVAR.
11. SKAT and SKAT-O association analisys  
    Association analisys for rare variants (Minor Allele Frequency <= 1%) was performed using rvtests.
    Four variant of gene-based and region-based filtration of rare variants was used:
    - All rare variants (without filtration)
    - All coding variants (non-synonomous)
    - All functional variants (non-synonomous, nearby to splicing site, loss-of-function)
    - Variants with CADD (Combined Annotation Dependent Depletion) > 12.37

## Results

Overall 8 lysosomal genes was used in SKAT and SKAT-O association analysis.

| Gene   | Protein                             |
| ------ | ----------------------------------- |
| GALC   | Galactosylceramidase                |
| ARSA   | Arylsulfatase A                     |
| GRN    | Progranulin                         |
| CTSB   | Cathepsin B                         |
| CTSD   | Cathepsin D                         |
| SCARB2 | Scavenger Receptor Class B Member 2 |
| FUCA1  | Alpha-L-Fucosidase 1                |
| GBA2   | Glucosylceramidase Beta 2           |

Based on results of SKAT and SKAT-O association tests two genes showed significant association (Table 1) in different groups of rare variants

*Table 1*. Statistically significant (p-value < 0.05) results of SKAT and SKAT-O tests

| Gene | Group                      | Test   | p-value     |
| ---- | -------------------------- | ------ | ----------- |
| ARSA | All rare variants          | SKAT   | 0.000461001 |
| ARSA | All rare variants          | SKAT-O | 0.000231025 |
| GALC | All functional variants    | SKAT-O | 0.0273533   |
| GALC | Variants with CADD > 12.37 | SKAT   | 0.0226957   |
| GALC | Variants with CADD > 12.37 | SKAT-O | 0.0118832   |

Full results of SKAT test: [SKAT](Assoc_results/Skat_summary.csv)

Full results of SKAT-O test: [SKAT-O](/Assoc_results/SkatO_summary.csv)

## Next step

On the next step genome-wide burden analysis will be perfored in order to reveal other genes associated with PD

## References

1. Chang, Christopher C., *et al*. "Second-generation PLINK: rising to the challenge of larger and richer datasets." Gigascience 4.1 (2015): s13742-015.
2. Wigginton, Janis E., David J. Cutler, and Gonçalo R. Abecasis. "A note on exact tests of Hardy-Weinberg equilibrium." The American Journal of Human Genetics 76.5 (2005): 887-893.
3. Yang, Jian, et al. "GCTA: a tool for genome-wide complex trait analysis." The American Journal of Human Genetics 88.1 (2011): 76-82.
4. Wang, Kai, Mingyao Li, and Hakon Hakonarson. "ANNOVAR: functional annotation of genetic variants from high-throughput sequencing data." Nucleic acids research 38.16 (2010): e164-e164.
5. Zhan, Xiaowei, *et al*. "RVTESTS: an efficient and comprehensive tool for rare variant association analysis using sequence data." Bioinformatics 32.9 (2016): 1423-1426.
6. Van der Auwera, G. A., and Brian D O’Connor. "Genomics in the Cloud: Using Docker, Gatk, and Wdl in Terra. CA 95472 Sebastopol, Canada." (2020).
