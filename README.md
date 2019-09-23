# Admixture
-----------------------------------------------------------------------

Quick start quide for using Admixture. Admixture is software designed to estimate breed composition (fractions) in human or livestock species. The documentation is pretty good compared to other free, open-sourced software. Usually there is no documentation or it's terrible (see STRUCTURE's documentation, if you can find it...). 

Input format is the same as PLINK (6 initial columns followed by 1/2 formatted genotypes), which makes it nice if you already work with PLINK. I did not and have created a shell script to convert a dense (ID 012 string format) to PLINK format. It will allow you to add a pedigree, but other than that it imputes family ID to "1", sex to "0" = missing, and phenotype to "1" because the only interest is to use PLINK to clean (run quality control) the genotype file for further analyses. 

Please see the website for [Admixture](http://software.genetics.ucla.edu/admixture/). You can download Admixture from [here](http://software.genetics.ucla.edu/admixture/download.html). The manual is availabe as a pdf within the last link. 

-----------------------------------------------------------------------
## Friendly Advice

Look through the online documentation for [PLINK](http://zzz.bwh.harvard.edu/plink/data.shtml#ped) if you have time and learn how to format input files. This will provide a much fuller documenation than presented here. This tutorial is only for getting data into Admixture and not everything PLINK can do, which is a lot. PLINK will be needed to process and clean genotypes if you haven't do so already so it's quite useful. Errors will occur in Admixture if not cleaned properly. 

-----------------------------------------------------------------------
## Input Formats

The input file format for genotypes is the same as PLINK. 

-----------------------------------------------------------------------
### Genotype (pedigree or .ped) file

Summary:
* Space Separated
* 6 Initial Columns (see below)
* Genotypes also space separated and require 2 alleles for each locus. Therefore 2p, where p is the number of SNPs. They are either allele coded (e.g. 'G G', 'C G', 'C C') for **12 coded** (i.e. '1 1', '1 2', '2 2', '0 0'. We will use the 12 coding for this tutorial. Animal breeders typically store genotypes in a dense (no space), "0125" format. 

Each individual is on one line with a few starter columns and then the genotypes 

Columns:
1. Family Identification - not quite sure what this, I just put "1" for all animals. 
2. Individual Identification - name or alphanumeric code for each animal
3. Sire (father) - sire identification of that individual
4. Dam (mother) - dam (mother) identification of that individual
5. Sex/Gender - 1 = male, 2 = female, 0 = unknown (or anything else)
6. Phenotype (just put '0' or '1' for all if you just want to process genotypes)
7. etc. Genotypes coded with 1's and 2's for each allele. Will have 2 columns for each locus. 

Example:
![](/Screenshots/plink_format.png)

-----------------------------------------------------------------------
#### How do I recode my genotypes?

We, in animal breeding, typically store genotypes in a dense format with 0 = homozygous, 1 = heterozygous, 2 = homozygous for alternative allele, 5 = missing and NO spaces. Only the ID, space, genotypes in one long string. This genotype coding represents the allele count at a locus (number of copies of the B allele usually). 

This is the DENSE format we typically are converting from:
![](/Screenshots/geno_dense_format.png)

PLINK wants it coded as follows:
* 0 --> '1 1'
* 1 --> '1 2'
* 2 --> '2 2'
* 5 --> '0 0'

Which is quite easy to do in a bash script because you can convert 2's first, then 1's, then 0's, then 5's and you shouldn't ever search and replace the same number. If you do this out of order, you will get major mistakes. You cannot convert 1 --> '1 2' then try to replace 2's, it will replace the wrong 2's with '2 2' as well! BE CAREFUL with this!

```bash
sed 's/2/ 2 2 /g' initial_file.txt > intermediate_file_1.txt
sed 's/1/ 1 2 /g' intermediate_file_1.txt > intermediate_file_2.txt
sed 's/0/ 1 1 /g' intermediate_file_2.txt > intermediate_file_3.txt
sed 's/5/ 0 0 /g' intermediate_file_3.txt | tr -s " " > intermediate_file_4.txt
```

You can use the `-i` command for search and replace 'in place' or just change the original file, but this is more portable across computers (for instance the OSX doesn't take this option by default without downloading the GNU version of sed I believe). 

The last command also uses the `tr` command to remove extra spaces (multiple in a row, now just 1). You can use the `time` command to see how long this is taking. It will be the slowest part of the entire script if you try to write one yourself. 

--------------------------------------------------------------------------
### Map (.map) file

This file is not required by Admixture, but here it is if you want to know. 

The map file is a little different than normal as well. We typically save the map file in a simple format ordered in the same order as the dense genotype string (e.g. "0102010050120"). 

Columns:
1. SNP name or number
2. Chromosome (1,2,...n, usually don't allow X/Y, replace with next number)
3. Position in base-pairs on the chromosome

PLINK needs the following:
1. Chromosome (1,2,...,n, X, Y, or 0 if unplaced)
2. rs# or SNP identification
3. Genetic distance (morgans)
4. Base-pair position on chromosome

Just put '0's for genetic distance if missing. We typically don't have this information. 


--------------------------------------------------------------------------
## Running Admixture

First, make sure your genotype file has been through quality control (QC). If not use PLINK to output a new .ped file with the updated genotypes. Subset based on missing rate (call rates) of SNPs and animals. Also minor allele frequency (MAF) and anything else that PLINK can do. 

Next, simply run:

```
admixture genotype_file.ped 3
```

Where genotype_file.ped is the formatted genotype file with 6 columns prior to the genotypes as shown above; '3' (K number) is the number of clusters if you have 3 different genetic lines mixing (common in animal breeding). K is used to indicate the number of clusters in K-means for example and is therefore used here to indicate the number of ancestrial populations. 

--------------------------------------------------------------------------
### Selecting the correct 'K' value

Selecting the best K value can be performed with CV and the `--cv=10` flag to do 10 fold CV. The default is 5-fold with just the `--cv` flag. You can do the following command to run the program for each value of K.

```
for K in 1 2 3 4 5; do
  admixture --cv genotype_file.ped $K | tee log${K}.out
done
```

The tee command allows the output to be printed to stdout while also saving the output to a file independently for each K value. Then remove the output you need with the following command. 

```
grep -h CV log*.out
```

Will print the CV errors for each. Pick the one that minimized the CV error. Sometimes there isn't much difference between K's and you'll need to use your judgement at times. 

--------------------------------------------------------------------------
### Bootstrapping to get SE's

```
admixture -B100 genotype_file.ped 3
```

Will do 100 bootstrap samples and place the SE estimates in a file called `genotype_file.Q_se` if the original file was called `genotype_file.ped`. Q stands for the breed composition fractions estimated. 

--------------------------------------------------------------------------
### Using parallel computing

```
admixture genotype_file.ped 3 -j16
```

Will use 16 processors if they are available. On the HPC you can ask for more cores/cpus. 



