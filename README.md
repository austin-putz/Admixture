# Admixture
-----------------------------------------------------------------------

Quick start quide for using Admixture. Admixture is software designed to estimate breed composition (fractions) in human or livestock species. The documentation is pretty good compared to other free, open-sourced software. Usually there is no documentation or it's terrible (see STRUCTURE's documentation, if you can find it...). 

Input format is the same as PLINK (6 initial columns followed by 1/2 formatted genotypes), which makes it nice if you already work with PLINK. I did not and have created a shell script to convert a dense (ID 012 string format) to PLINK format. It will allow you to add a pedigree, but other than that it imputes family ID to "1", sex to "0" = missing, and phenotype to "1" because the only interest is to use PLINK to clean (run quality control) the genotype file for further analyses. 

Please see the website for [Admixture](http://software.genetics.ucla.edu/admixture/). You can download Admixture from [here](http://software.genetics.ucla.edu/admixture/download.html). The manual is availabe as a pdf within the last link. 

-----------------------------------------------------------------------
## Friendly Advice

Just look through the online documentation for [PLINK](http://zzz.bwh.harvard.edu/plink/data.shtml#ped) and learn how to input files. This will provide a much fuller documenation than presented here. Here is only for getting data into Admixture and not everything PLINK can do, which is a lot. PLINK will be needed to process and clean genotypes if you haven't do so already so it's quite useful. Errors will occur in Admixture if not cleaned properly. 

-----------------------------------------------------------------------
## Input Formats

The input file format for genotypes is the same as PLINK. PLINK uses -9 as missing by default I believe. 

-----------------------------------------------------------------------
### Genotype (pedigree or .ped) file

Summary:
*) Space Separated
*) 6 Initial Columns
*) Genotypes also space separated and require 2 alleles for each locus. Therefore 2*n, where n is the number of SNPs. They are either allele coded (e.g. 'G G', 'C G', 'C C') for **12 coded** (i.e. '1 1', '1 2', '2 2', '0 0'. We will use the 12 coding for this tutorial. Animal breeders typically store genotypes in 012/5 format. 

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
*) 0 --> '1 1'
*) 1 --> '1 2'
*) 2 --> '2 2'
*) 5 --> '0 0'

Which is quite easy to do in a bash script because you can convert 2's first, then 1's, then 0's, then 5's and you shouldn't ever search and replace the same number. If you do this out of order, you will get major mistakes. You cannot convert 1 --> '1 2' then try to replace 2's, it will replace the wrong 2's with '2 2' as well! BE CAREFUL with this!

--------------------------------------------------------------------------
### Map (.map) file

The map file. 




