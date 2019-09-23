# Admixture

Admixture is software designed to estimate breed composition (fractions) in human or livestock species. The documentation is fairly good for Admixture compared to other free, open-sourced software. 

Input format is the same as PLINK (6 initial columns followed by 1/2 formatted genotypes), which makes it nice if you already work with PLINK. I did not and have created a shell script to convert a dense (ID 012 string format) to PLINK format. It will allow you to add a pedigree, but other than that it imputes family ID to "1", sex to "0" = missing, and phenotype to "1" because the only interest is to use PLINK to clean (run quality control) the genotype file for further analyses. 

Please see the website for [Admixture](http://software.genetics.ucla.edu/admixture/). You can download Admixture from [here](http://software.genetics.ucla.edu/admixture/download.html). 



