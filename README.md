# UK Biobank GWAS

**GOALS:**
 * Collect UK Biobank phenotypes from collaborating applications 
 * When necessary, convert phenotypes into accessible case/control or quantitative values using the PHESANT algorithm
 * Use the same sample and variant QC across all phenotypes
 * Run SNP association on UKBB imputed dosage BGEN files using hail on a google cloud
 * Provide per-SNP summary stats for all approved phenotypes
 * GWAS Results available for viewing and download via https://biobankengine.stanford.edu/search# (run by the Rivas Lab at Stanford)

* [Workflow](#workflow)
   1. [Files](#files)
   2. [UK Biobank updates](#updates)
   3. [Phenotypes and applications](#phenotypes-and-applications)
   4. [Sample and Variant QC](#sample-and-variant-qc)
   5. [Association in Hail](#hail-association)
   6. [Summary stat output](#summary-stat-output)

## Files

 * **ukb1859** - Liam Abbott's scripts to run Hail's linreg3 on application 1859
 * **PHESANT_pipeline.pdf** - Diagram of PHESANT phenotype curation strategy

## UK Biobank Updates

**July 27th, 2017 - Errors in imputation identified**
  * non-HRC imputation was mis-mapped. Details below:
```
We have identified a problem with the UK Biobank imputed data and which has come to light following discussion via the UKB-GENETICS mail list.
This problem relates to the imputed data and does not affect the genotyped data from the Affymetrix array. 
The genetic data was imputed using two different reference panels.
The Haplotype Reference Consortium (HRC) panel was used as first choice option, but for SNPs not in that reference panel the UK10K + 1000 Genomes panel was used.
The problem arose in the second set of imputed data from the UK10K + 1000 Genomes panel.
The genotypes at these SNPs are imputed correctly, but have not been recorded as having the correct genome position in the files.
We have established that the imputed data from the HRC panel is not affected and has the correct positions.
This is about ~40M sites and will include the majority of the common SNPs i.e. sites most likely to show genetic associations.
These sites are readily identified since the HRC site list is public
http://www.haplotype-reference-consortium.org/site
The problem is not easy to fix post-hoc, so we intend to re-impute the data from the UK10K + 1000 Genomes panel and re-release the imputed data.
For now we recommend that researchers focus exclusively on SNPs in the HRC panel, or work with the directly genotyped data until the new release is available.
We will progress the re-imputation as quickly as we can and expect to release a new version of the imputed files ideally in September.
We will send more details about this data release and confirm timelines in due course. 
We can only apologise that this error was not identified during the QA review and do not underestimate the frustration this will cause for the research community.
```

**July 26th, 2017 - Samples withdrawn from UK Biobank**
  * 23 samples have withdrawn consent for use of their data
  * 8 samples listed in imputed data sample file
  * 6 samples identified as being part of our QC positive sample set

## Phenotypes and applications

**Current list of participating UK Biobank Applications**
  * 18597 - Primary investigators: Benjamin Neale / Verneri Anttila
  * 11898 - Joel Hirschhorn
  * 11425 - Daniel Benjamin
  * 32568 - Jordan Smoller
  * 24983 - Manuel Rivas

**Phenotype Collection Strategy**
  * Collect as many phenotypes as possible from collaborating UK Biobank applications
  * Ensure all applications permit GWAS analysis and public release of summary statistics
  * Add participating Neale Lab members to all applications

**Phenotype Curation Strategy**
  * Auto-curation will be done by Duncan Palmer using PHESANT: 
    * Source repository: https://github.com/MRCIEU/PHESANT
    * Customized PHESANT repository: https://github.com/astheeggeggs/PHESANT

**Phenotype output**
  * PHESANT ukb[XXXX]_output.tsv = full phenotype file with rows=sample ID and columns=phenotype ID
  * Summary file: ukb[XXXX]_phenosummary_final.tsv
    * row number - numeric field matching UK Biobank data showcase field
    * Field - short description of phenotype
    * N.non.missing - number of non-missing QC positive samples
    * N.missing - number of missing QC positive samples
    * N.cases - number of QC positive samples responding affirmatively to phenotype designation (NA if quantitative) 
    * N.controls - number of QC positive samples responding negatively to phenotype designation (NA if quantitative)
    * Notes - extended description of phenotype
    * PHESANT.notes - categorizations of phenotype by PHESANT algorithm
    * PHESANT.reassignments - changes to phenotype values by PHESANT

**Phenotype / Genotype linking**
  * The main link files from application to UK Biobank imputed dataset are the .fam and .sample file
  * both files have the same IDs
  	* the .fam file is ordered the same way as the ukb_sqc_v2.txt file
  	* the .sample file is ordered the same way as the .bgen file

## Sample and Variant QC

**Sample QC** 

**Primary sample QC parameters for GWAS from ukb_sqc_v2.txt file:**
  * in.white.British.ancestry.subset==1
  * used.in.pca.calculation==1
  * excess.relatives==0
  * putative.sex.chromosome.aneuploidy==0

**Additional QC parameters**
  * Samples withdrawn = 8 
  * Samples redacted = 3 ([-3,-2,-1] in the sample ID) 

Samples removed from QC file = 151180
Samples retained in QC file = 337199
NOTE: all samples retained are in the .bgen files

 * The ukb_sqc_v2.txt file has more samples than the .bgen files, but the same number of samples as the application specific .sample file

**Description of inclusion parameters:**

```
het.missing.outliers		      (0/1)	(no/yes) Indicates samples identified as outliers in heterozygosity and missing rates, which indicates poor-quality genotypes for these samples.
putative.sex.chromosome.aneuploidy    (0/1)	(no/yes) Indicates samples identified as putatively carrying sex chromosome configurations that are not either XX or XY. These were identified by looking at average log2Ratios for Y and X chromosomes. See genotype QC documentation for details.
in.kinship.table		      (0/1)	(no/yes) Indicates samples which have at least one relative among the set of genotyped individuals. These are exactly the set of samples that appear in the kinship table. See genotype QC documentation for details.
excluded.from.kinship.inference	      (0/1)	(no/yes) Indicates samples which were excluded from the kinship inference procedure. See genotype QC documentation for details.
excess.relatives		      (0/1)	(no/yes) Indicates samples which have more than 10 putative third-degree relatives in the kinship table.
in.white.British.ancestry.subset      (0/1)	(no/yes) Indicates samples who self-reported 'White British' and have very similar genetic ancestry based on a principal components analysis of the genotypes. See genotype QC documentation for details.
used.in.pca.calculation		      (0/1)	(no/yes) Indicates samples which were used to compute principal components. All samples with genotype data were subsequently projected on to each of the 40 computed components. See genotype QC documentation for details.
in.Phasing.Input.chr1_22	      (0/1)	(no/yes) Indicates sample was in the input for phasing of chr1-chr22.
in.Phasing.Input.chrX		      (0/1)	(no/yes) Indicates sample was in the input for phasing of chrX.
in.Phasing.Input.chrXY		      (0/1)	(no/yes) Indicates sample was in the input for phasing of chrXY.

Quick descriptives of categories:

table(het.missing.outliers)
     0 
487409 
table(putative.sex.chromosome.aneuploidy)
     0      1
486757    652
table(in.kinship.table)
     0      1 
339678 147731 
table(excluded.from.kinship.inference)
     0      1 
487400      9 
table(excess.relatives)
     0      1 
487221    188 
table(in.white.British.ancestry.subset)
     0      1 
 78437 408972 
table(used.in.pca.calculation)
     0      1 
 80190 407219 

Total samples in imputed BGEN files: 487409
Withdrawn samples (remove): 6
Redacted samples (remove): 3
In white, British ancestry subset (keep): 408972
Used in PCA calculation (keep): 407219
Marked as having excess relatives (remove): 188
Marked as sex chromosome aneuploidies (remove): 652
Samples remaining: 337199
```

**Genotype QC** 

**Primary genotype QC parameters for inclusion to GWAS from ukb_sqc_v2.txt file:**
  * SNPs present in HRC imputation file: `../imputed/resources/HRC/HRC.r1-1.GRCh37.wgs.mac5.sites.tab`
  * UKBB pHWE > 1e-10
  * UKBB callRate > 0.95
  * UKBB INFO score > 0.8
  * QC positive Alternate Allele Frequency  (0.001 > AF < 0.999 in 337199 QC positive samples)
  * NOTE: MAF and INFO score available in per chromosome files: ukb_mfi_chr*_v2.txt

```
Quick descriptive of genotype inclusion counts:

QC positive sample subset (n = 337199):
- Total Variants: 92,693,895
- in HRC imputation: 39,131,578
- w/ pHWE > 1e-10: 92,471,744
- w/ call rate > 95%: 92,693,895
- w/ 0.001 < MAF < 0.999: 16,047,295
- w/ INFO score > 0.8: 29,447,617
- w/ all of above: 10,894,597
```

**Association in Hail**

SNP association is performed on UKBB imputed dosage BGEN files using hail on a google cloud platform.

* [Hail](https://hail.is/) scalable genetic analyses
* [Google Cloud Platform](https://cloud.google.com/) cluster computation and data storage
* [cloudtools](https://github.com/Nealelab/cloudtools) script submission to compute clusters

This allows us to take advantage of large cluster computing and parallel processing via Apache Spark

**Association model**
 * **To expedite our initial GWAS, we implemented a linear regression model on all phenotypes**
 * Linear regression model covariates were sex and the first 10 PCs (taken directly from ukb_sqc_v2.txt) 
 * Hail's [linreg3] command used for all phenotypes
    * Can do up to 110 phneotypes simultaneously when NA structure is identical
    * Can do up to 37 phenotypes simultaneously when NA structure varies

 * STEPS for running regression in Hail:
```
    1) Read in .bgen and .sample files
    2) Read in sqc_v2 covariate file (matched to application-specific phenotypes)
    3) Read in PHESANT-curated application specific phenotype file
    4) Subset phenotype file to QC-positive samples:
       - Turn phenotypes to NA for all QC-negative samples
    5) Subset genotype file to QC-positive SNPs
    6) Run linreg3
    7) Export summary stats (detailed below)  	   
```
 * Examples scripts are listed here
  * [ukb1859_map_results.py](https://github.com/Nealelab/UK_Biobank_GWAS/ukb1859_map_results.py)
  * [ukb1859_build_pipelines.py](https://github.com/Nealelab/UK_Biobank_GWAS/ukb1859_build_pipelines.py)
  * [ukb1859_linreg3.py](https://github.com/Nealelab/UK_Biobank_GWAS/ukb1859_linreg3.py)


## Summary stat output

**QCed SNP information file**
    * variant (hg19) [CHROM:POS:REF:ALT]
    * rsid
    * info (UKBB INFO score)
    * AF (QC positive alternate allele frequency)
    * pHWE
    * callRate
  * Example SNP information: 
```	
variant	rsid	info	AF	pHWE	callRate
10:61334:G:A	rs183305313	8.46690e-01	4.82208e-03	4.77873e-03	1.00000e+00
10:69083:C:T	rs35418599	8.01306e-01	7.77212e-01	7.74927e-01	9.99961e-01
10:90127:C:T	rs185642176	9.84897e-01	8.62740e-02	2.77123e-01	1.00000e+00
10:94263:C:A	rs184120752	9.57328e-01	2.51928e-02	8.26055e-02	1.00000e+00
10:94426:C:T	rs10904045	9.90536e-01	3.96792e-01	3.54267e-01	1.00000e+00
10:94538:C:T	rs189409193	8.24138e-01	5.50862e-03	9.37271e-01	1.00000e+00
```

**SNP summary stat file**
    * variant (hg19) [CHROM:POS:REF:ALT]
    * rsid
    * nCompleteSamples (non-missing samples)
    * AC (non-missing sample allele count)
    * ytx (case/control = dosage weighted alternate allele count in cases; quantitative = dosage weighted mean trait value among alternate allele carriers)
    * beta
    * se
    * pval
  * Example SNP summary stat: 
```
variant	rsid	nCompleteSamples	AC	ytx	beta	se	tstat	pval
5:43888254:C:T	rs13184706	953	4.17176e+01	5.64980e+01	-1.11569e-01	8.01312e-02	-1.39233e+00	1.64152e-01
5:43888493:C:T	rs58824264	953	9.03529e+00	1.30706e+01	-3.42168e-02	1.68596e-01	-2.02951e-01	8.39217e-01
5:43888556:T:C	rs72762387	953	4.86235e+01	7.81804e+01	1.31571e-01	7.44976e-02	1.76611e+00	7.77023e-02
5:43888648:C:T	rs115032754	953	3.77647e+01	5.40039e+01	-5.98780e-02	8.59590e-02	-6.96588e-01	4.86233e-01
5:43888690:C:G	rs147555725	953	5.87843e+00	9.80000e+00	1.98330e-01	2.11226e-01	9.38946e-01	3.48000e-01
5:43888838:G:C	rs13185925	953	7.21765e+01	1.04306e+02	-2.46341e-02	6.00665e-02	-4.10113e-01	6.81816e-01
```
