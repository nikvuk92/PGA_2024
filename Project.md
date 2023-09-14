# PGA Course Autumn 2023

## Instructions for project work

The goal of this population genomics project is to familiarize you with the workflows, different types of commonly used analyses as well as a way of thinking about how one approaches questions such as trying to explain the underlying forces that affect populations in general. We know that people tend to migrate and throughout history they have done that a lot. Similar populations get isolated and with time may become more distant, distant peoples admix with time may become more closely related - and trying to decipher human population history is usually not an easy task.

The structure of the project work will be organized as follows: 

- each group consists of two students;
- you need an username & password to log into Uppmax;
- up to you to decide if you want to work on your own laptops or in Hubben;
- we have an introductory meeting where we discuss the framework of the exercize in depth;
- we will create a slack group in which you will be able to ask specific questions during the project duration;
- we can also set up group meeting times over zoom;
- the end product should be a Scientific report of ~10 pages, including Abstract, Introduction, Materials & Methods, Results, Discussion, References, Appendices (length is not crucial here, substance is);
- A short presentation (16-17 October);
- the deadline for the written report is the day of the presentation.

## Unmasking the mystery of your 25 samples

The setup of this project is very similar to what you would do as a researcher in the field of (ancient) Human Genetics.
Therefore, it is up to you to identify the populations you have received and aided by literature on the subject as well as some historical context, try to infer possible demographic events. 
In order to achieve this, you are provided with a two reference datasets that should come in handy as a way to compare the unknown samples to the known ones. 
```
/crex/proj/uppmax2023-2-38/ANCIENT_REFERENCE_DATASET
/crex/proj/uppmax2023-2-38/MODERN_REFERENCE_DATASET
```
Helpful scripts are there to aid you on your quest can be found:
```
/crex/proj/uppmax2023-2-38/SCRIPTS
```
The setup of this The unknown samples are a mysterious bunch. They look different from the rest don't they? (*hint - check the genotyping rate - i.e. missingness*).

In the steps that will lead you towards unmasking the unknown populations you will need to: 

1. Take a look at both Reference populations to get a feel of what you are working with. Check the genotyping rate of both; 
2. Check for related individuals in your **modern reference dataset**;
3. Filter the related individuals (if there are any) from the **reference dataset** - *we have nothing against family, but individuals that are in close kinship will affect our downstream analysis and we don't want that*;
4. Filter the **modern reference** individuals & SNPs (missingness, HWE, MAF) - *word of caution: do not apply the 'maf' filtering until you have your final references dataset!*;
   *Remember: The **unknown** samples **do not** go through any filtering!*
5. Run a PCA on the **modern reference** dataset only to get an understanding of the populations that will build the principle components;
6. In case there is something odd with the way your PCA looks like, think about how you can solve it.
7. After bringing the **modern reference** panel into shape merge the two reference datasets to each other;
6. Unify SNP names across your dataset - change the SNP name into the names based on position. This is needed when merging datasets from different SNP chips (the same position can have different names on different chips);
7. After merging together your reference dataset - merge it together with your **unknown** data;
8. Before running PCA/Admixture you need to prune your data for SNPs in LD (pruning is explained in the admixture manual https://dalexander.github.io/admixture/admixture-manual.pdf);
9. Try to run a normal PCA on this dataset - can you explain why you think this is/isn't a good idea?;
10. Run a projected PCA;
11. Run Admixture (remember - Admixture takes a lot of time! Organize your time wisely);
12. Read up on some literature - it will help for writing the report! (Pay special attention to the key figures in them):
TAPAS [Olalde et al 2019](https://www.science.org/doi/10.1126/science.aav4040)
FIKABROD [Malmström 2009, 2019](https://royalsocietypublishing.org/doi/full/10.1098/rspb.2019.1528; https://www.sciencedirect.com/science/article/pii/S0960982209016947?via%3Dihub) 
FISHNCHIPS [Olalde 2018, Patterson 2021](https://www.nature.com/articles/nature25738; https://www.nature.com/articles/s41586-021-04287-4)
CARBONARA [Pritchard 2019](https://www.science.org/doi/10.1126/science.aay6826)
13. Write your report. Make sure to explain the following questions:
- First describe what you see when you plot the modern populations. Can you say anything about this initial PCA?
- Using the techniques explained below, make your best guess on the makeup of each unknown sample. Some will be easy and for others it will be more difficult.
- After reading the papers, together with your findings and discussion with the TA you should be able to give some context to your populations and present your findings.
14. Don't panic - there's plenty of time. Or is there?

## Some technicalities

* You should have R studio installed on your computer

Before we start here are some basic Unix/Linux commands if you are not used to working in a Unix-style terminal:

### Logging into SNOWY & working on the server:

```
ssh -AX user@rackham.uppmax.uu.se
```
After which you type in your password. *And yes, it'll not show up when typing it, but if you type it in correctly & press enter, you're in.*

Note: When acessing SNOWY from Rackhams login nodes you must always use the flag -M for all SLURM commands.
Examples:

```
- squeue -M snowy
- jobinfo -M snowy
- sbatch -M snowy slurm_script_file
- scancel -u username -M snowy
- interactive -A projectname -M snowy -p node -n 32 -t 01:00:00
```

Note: It is recommended to load all your modules in your job script file. This is even more important when running on Snowy since the module environment is not the same on the Rackham login nodes as on Snowy compute nodes.
You can read up more on how to use SNOWY : [Snowy_User_guide](https://www.uppmax.uu.se/support/user-guides/snowy-user-guide/)

Generally, for any bigger job (bigger Plink jobs, PCA, Admixture) **always work in interactive mode**.

### How to run interactively on a compute node:

Ex.:
```
   interactive -A projectname -M snowy -p node -n 32 -t 02:00:00
```
### Moving about:
```
    cd – change directory
    pwd – display the name of your current directory
    ls – list names of files in a directory
```
### File/Directory manipulations:
```
    cp – copy a file
    mv – move or rename files or directories
    mkdir – make a directory
    rm – remove files or directories
    rmdir – remove a directory
```
### Display file content:
```
    cat - concatenate file contents to the screen or to a file
    less - open file for viewing
    more 
    tail
    head
    realpath - checking the path to a file 
```
### You can check your jobs with:

```
jobinfo -u YOUR_USERNAME -M snowy
```
When creating and editing a file in text editor you can use ```nano``` or ```vim``` or something else.

# PART 0 Knowing your way around & using Plink   	    

**Beware** - it is vital to not apply any filters to your unknown samples - we don't know anything about them at this point! 

FYI: Link to PLINK site:[https://www.cog-genomics.org/plink2](https://www.cog-genomics.org/plink2)

PLINK is a software for fast and efficient filtering, merging, editing of large SNP datasets, and exports to different outputs directly usable in other programs. Stores huge datasets in compact binary format from which it reads-in data very quickly.

### Running the program:

If working on Rackham type in the following:
```
module load bioinfo-tools
module load plink/1.90b4.9 

```
The software is already pre-installed on Rackham, you just have to load it

Try it:
```
plink
```

### This is what a basic command structure looks like in Plink: 
``` 
plink --filetypeflag filename --commandflag commandspecification --outputfilecommand --out outputfilename
```
For example to do filtering of missing markers at 10% frequency cutoff (reading-in a bed format file called file1, doing the filtering, writing to a ped format file called file2):
```
plink --bfile file1 --geno 0.1 --recode --out file2
```

### Input formats:
File format summary:

ped format: usual format (.ped and .fam)
.ped contains familyID Paternal ID Maternal ID Sex Phenotype
.fam files contain sample info

bed format: binary/compact ped format (.fam .bim and .bed)
(.fam - sample info  .bim - marker info  and .bed - genotype info in binary format

tped format: transposed ped format (.tfam and .tped files) a text-based file format commonly used to store genotype data. It contains information about SNP markers and the genotypes for each individual. Each SNP marker can have two alleles, one from each chromosome (paternal and maternal).

tfam sample info, .tped marker and genotype info in transposed format

lgen format: long format (see manual, not used that often)

### Setup
Navigate to the directory you want to work in.
Copy over the reference datasets from their respective directories to your working folder by **changing** the command below while you are in your working folder. The **dot** means copy the contents "here". Do this for all your datasets. 
It is crucial to do this correctly, otherwise you might jeopardize the files for your colleagues.
*Advice - it might be easier for you if all your datasets (.bed +.bim +.fam files for all four, instead of having to go in and out of the different folders) are in the same directory.

```
cp /full_path_to_course_material/unk1.bed .
cp /full_path_to_course_material/unk1.bim .
cp /full_path_to_course_material/unk1.fam .
```

**The files above contain SNPs from a number of unknown individuals in bed file format**
 
Look at the `.bim` (markers) and `.fam` (sample info) files by typing:

```
less Unknown.bim
``` 
do the same for the `.fam` file

```
less Unknown.fam 
```
As mentioned before the `.bim` file store the variant markers present in the data and `.fam` lists the samples. (you can try to look at the .bed as well, but this file is in binary format and cannot be visualized as text if you want to see the specific genotype info you must export the bed format to a ped format)

Read in a  bed file dataset and convert to ped format by typing/pasting in:

```
plink --bfile Unknown --recode --out Unknown_ped 
```

### Look at the info that Plink prints to the screen. 
How many SNPs are there in the data? How many individuals? How much missing data?

Look at the missingness information of each individual and SNP by typing:

```
plink  --bfile Unknown --missing --out test1miss

```
Look at the two generated files by using the `less` command (q to quit)

```
less test1miss.imiss
less test1miss.lmiss
```

The `.imiss` contains the individual missingness and the `.lmiss` the marker missingness
Do this for both the reference datasets and your unknown dataset, but be careful not to overwrite the same file. (Note all of these numbers in your report)
Do you understand the columns of the files? The last three columns are the number of missing, the total, and the fraction of missing markers and individuals for the two files respectively

Look at the first few lines of the newly generated .map (fileset variant information file) and .ped (Paternal ID Maternal ID Sex Phenotype) files using the more command as demonstrated above.

Read in bed/ped file and convert to tped:

```
plink --bfile Unknown --recode transpose --out Unknown_tped 
plink --file Unknown_ped --recode transpose --out Unknown_tped 
```
**Do you see the difference in the two commands above for reading from a bed (--bfile) and reading from a ped (--file) file. 
Which one takes longer to read-in?**

Look at the first few lines of the  `.tfam` and `.tped` files by using the `less` command

**Can you see what symbol is used to encode missing data?**

Note- try to always work with bed files, they are much smaller and takes less time to read in. 
See this for yourself by inspecting the difference in file sizes:

```
ls -lh * 
```

Plink can convert to other file formats as well, you can have a look in the manual for the different types of conversions

# PART 1: Merging, Filtering, QC steps
*Remember, you are only filtering the modern reference dataset*

## Step 1 Filtering for missing data
First, we filter for marker missingness:

*YOUR_DATASET refers (below) to the MODERN_REFERENCE_DATASET*

Paste in the command below to filter out markers with more than 10% missing data

```
plink --bfile YOUR_DATASET --geno 0.1 --make-bed --out YOUR_DATASET2 
```

Look at the screen output, how many SNPs were excluded?

## Step 2 Filter for individual missingness

Paste in the command below to filter out individual missingness

```
plink --bfile YOUR_DATASET2 --mind 0.15 --make-bed --out YOUR_DATASET3 
```

Look at the screen output, how many individuals were excluded?

## Step 3 MAF filtering

To filter for minimum allele frequency is not always optimal, especially if you are going to merge your data with other datasets in which the alleles might be present. But we will apply a MAF filter in this case

Filter data for a minimum allele frequency of 1% by pasting in:

```
plink --bfile YOUR_DATASET3 --maf 0.01 --make-bed --out YOUR_DATASET4 
```

How many SNPs are left at this point?

## Step 4 Filtering for SNPs out of Hardy-Weinberg equilibrium

Most likely, SNPs out of HWE usually indicates problems with the genotyping. However, to avoid filtering out SNPs that are selected for/against in certain groups (especially when working with case/control data) filtering HWE per group is recommended. After, only exclude the common SNPs that fall out of the HWE in the different groups - (OPTIONAL). But for reasons of time, we will now just filter the entire dataset for SNPs that aren’t in HWE with a significance value of 0.001

```
plink --bfile YOUR_DATASET4 --hwe 0.001 --make-bed --out YOUR_DATASET5
```

Look at the screen. How many SNPs were excluded?

If you only what to look at the HWE stats you can do as follows. By doing this command you can also obtain the observed and expected heterozygosities. 

```
plink --bfile YOUR_DATASET5 --hardy --out hardy_YOUR_DATASET5
```
Look at file hardy_YOUR_DATASET5.hwe, see if you understand the output?

There are additional filtering steps that you can go further. PLINK site on the side lists all the cool commands that you can use to treat your data. Usually, we also filter for related individuals and do a sex-check on the X-chromosome to check for sample mix-ups. 

## Step 5 Filtering out related individuals (just the Modern Reference dataset)

In the `SCRIPTS` folder, there is a script called `sbatch_KING.sh` that can be used to run [KING](http://people.virginia.edu/~wc9c/KING/manual.html) You can have a look inside for instructions on how to run the script. Check out the manual and try and figure out how the software works.
After you have run the script have a look at the produced output files and figure out how to remove the related individuals. (*Hint - keep via Plink might be a good option) 
*P.S. You don't need to do run KING for the Unknown dataset.*

First load ```bioinfo-tools``` and then ```module load KING```.
The script for KING is as follows:

```
#!/bin/bash -l
#
#
#SBATCH -J king
#SBATCH -t 12:00:00
#SBATCH -A uppmax20XX-XX-XX
#SBATCH -n 8
king -b $1  --unrelated
```
You can submit it as a job by doing:
```
sbatch -M snowy THIS_SCRIPT.sh YOUR_DATASET5.bed
```
Look at the output from KING & **keep** the unrelated individuals.

Checkpoint *At this point, when you exclude the related individuals you should be at YOUR_DATASET_6*

## Step 6 Make the modern dataset haploid 
"Pseudohaploidizing" a TPED file typically involves converting a diploid genotype data file into a pseudohaploid format, where each variant or SNP (Single Nucleotide Polymorphism) is represented only once, as if the genotype data came from a haploid individual. This is mainly employed when working with ancient DNA. The script that we are using selects one allele for each SNP marker for each individual. There are multiple ways for the allele selection process but it must be consistent across the dataset. Can you think about why this is a good idea to do when dealing with ancient DNA? 

you can run the script on the modern dataset (in an interactive node) like this:
```
haploidize_tped.py < YOURDATASET.tped > YOURDATASET_haploid.tped
```
It will take some time to run because it goes through each SNP position in your file. After it is done, the file YOURDATASET_haploid.tped is the haploid version of YOURDATASET.tped. Therefore you can either rename YOURDATASET.tped into something different (or if you are certain, just delete it) and rename the haploid version to just YOURDATASET.tped. This way, all the files that accompanied the original tped will work for your "new" version.

P.S. There is no need to make the ancient reference dataset pseudohaploid because we have already made sure that it is for you.

## Step 7 Plot a PCA using the Modern reference individuals
Plot a PCA using plink on the Modern reference individuals and describe what you see.

```plink --bfile YOUR_DATASET5 --pca```

This should give you an eigenvec and an eigenval file as outputs. Plot them in R and include in your report. Also plot the different PCs (as barplots) to show how much of the variation each one explains.
Try to describe what each PC represents.

## Step 8 Before merging let's first unify SNP names 
When merging datasets from different chips, the same position can have different names on different chips. The rename_SNP.py script can be used on the .bim files to change the SNP name into the names based on position. Luckily for you, this has already been done for the three reference datasets. Otherwise there would have been just the additional step of renaming the SNPs for each dataset prior to merging. But since we don't know where the Unknown dataset came from, we want to conduct this step for it. Check the bim file for the Unknown dataset and see the SNP names before you rename them.

In order to run rename_SNP.py first load ```module load python/2.7.11``` (Issues might occur if not using the right version of Python).

```python rename_SNP.py FILE.bim```

After which you should get that the output with replaced SNP names has been written to FILE.bim_pos_names. You can delete the original bim (or rename it if you're feeling unsure) & edit the 

```FILE.bim_pos_names``` to be ```FILE.bim```.

Check to see what happened after the script did its job.

### Step 9 Merging the reference datasets to eachother

*P.S. As long as you have all the files/datasets in the same directory you don't need to specify the full paths. If they are in different datasets, remember to add the full path to the file, such as:
```/proj/uppmaxproject/full_path_to_YOUR_OWN_directory/Reference_datasets/datasetA/file```

*P.P.S. Do this and other heavy lifting in an **interactive node!**

Merging is done using plink's bmerge command:
```plink --bmerge FILE_A --bfile FILE_B --out FILEA+B```

Check the missingness again at this point and report it in your findings. How did it change ? Why do you think this is?

*Checkpoint* - You can name this dataset ```ALL_REFERENCE_DATASETS``` so that it's easier to follow.

### Step 10 Merge the Unknown dataset to the reference-combined-dataset
Finally, after you've merged the two reference datasets together, we need to merge in the Unknowns as well. To do this, as previously, 
Again did the missigness change in any way? 
*Checkpoint* - You can name this dataset ```ALL_COMBINED_DATASETS``` so that it's easier to follow.

These are the final files for the next exercise. Rename them:
```
mv FINAL_DATASET.bed PopStrucIn1.bed; mv FINAL_DATASET.bim PopStrucIn1.bim; mv FINAL_DATASET.fam PopStrucIn1.fam 
```
## Step 11 Remove all unnecessary iterations of the same data except .log files from the merging/filtering steps and of course the final dataset
As our storage on uppmax is quite limited, this is a good point to free up your space from all the data that was produced from each of the previous steps and just keep the final dataset.  
Feels stupid to say, but please do not delete the files you need downstream.

## Step 12 Reflect and review
How many SNPs are you working with?  
Now you have generated your input files for the next exercise which will deal with population structure analysis on the combined dataset. You will look at the population structure of your unknown samples in comparison to known known reference populations (such as Human Origins).

=============================================================================

# PART 2: Population structure inference 

**Word of advice - the following analyses might take quite some time to run. Therefore, plan accordingly and leave yourself enough room for error.**

Furthermore, make sure to doublecheck your script before running it. 90% of issues you will encounter when running scripts are going to be typos (*Sanchez,R., Smith,M., 2022*). Check if you have written the correct file names, the correct extensions, whether you have the right path to your file or whether you have typed in your correct project name. Since we have a limited CPU hours allocated for this course it is really important to be efficient!  

## Step 1 LD Pruning 

Before running PCA & ADMIXTURE it is advisable to prune the data to thin the marker set for linkage disequilibrium.
You can read up on how to prune for LD(https://dalexander.github.io/admixture/admixture-manual.pdf).

```
plink --bfile FINAL_DATASET --indep-pairwise 10 10 0.1
```


this creates some output files. Check what they contain.


```
plink --bfile FINAL_DATASET --extract plink.prune.in --make-bed --out FINAL_DATASET_PRUNED
```
Check how much you're pruning out. Perhaps you can tweak the parameters if you are pruning out too much! 

## Step 2 Projected PCA

There is a reason why we are doing a projected PCA instead of a "conventional" PCA. It would be good to think about why we have decided to do this step. Maybe read up a bit on what it is and what it does.
But it is surely something to do with the specific nature of the samples we are dealing with.

For running a projected PCA, you first need to produce some files:

  - a tped file (your final merged dataset); 
  - a list of the modern populations in the dataset;
  - a list of the unknown populations in the dataset;

To produce the tped, you can use plink as you've done so far. 

And to produce the two population lists:

```
awk {'print $1'} FINAL_DATASET.tfam |sort|uniq > Sorted_list_of_the_populations.txt
```
Then, this sorted list of populations, you can subset to an ```unknown.txt``` and a ```modern.txt``` using any approach you like.
*Note Populations IDs in "modern reference pops" must exactly match the IDs in the tfam, while population IDs in "unkown pops to include" can be more general (e.g. HG_SE instead of HG_SE_SF12) - No error messages when this is violated but weird results may occur!

Hint: Check the ```pca_lsqpeoj.sh``` file if it is calling the following modules:

```
module load bioinfo-tools
module load plink/1.90b4.9
module load python/2.7.11
module load eigensoft/7.2.0
```

Now make a ```tped``` version of the dataset and you can go ahead & submit a sbatch job:

```
sbatch -A correctUPPMAXproject -M snowy -p core -n 2 -t 07:00:00 -J PCA_lsq -e LSQ.er pca_lsqproj.sh FINAL_DATASET_PRUNED_TPED MODERN_SAMPLES.txt UNKNOWN_SAMPLES.txt
```
This should take a few hours. You can check whether the job is still running with ```jobinfo```.

```jobinfo -M snowy -u YOUR_USERNAME```

When it's done, assuming it has ended sucessfuly, copy the ```.evec``` and ```.eval``` files to your computer's R directory and plot the PCA.

There are multiple ways of doing this, but if you can't think of a better one here's a suggestion:

```
library(ggplot2)

pca<-read.table("FINAL_DATASET_PRUNED_TPED.popsubset.evec")
####### DIVIDE THE TABLE INTO MODERN & UNKNOWN - make sure to edit the right numbers! 

modern<-pca[c(1:1610),]
modern$V12<-factor(modern$V12)
ancient<-pca[c(1611:1626),]

###### PLOTTING THE MODERN SAMPLES
par(mfrow=c(1,1))
layout(matrix(c(1,1), ncol=1), heights=c(1,2))
par(mar=c(4,4,1,1))
with(pca,plot(pca$V2~pca$V3,pch=20,cex=0.5,col = rgb(0, 0, 0, 0.15),xlab="PCA2",ylab="PCA1"))

###### LABELING THE PLOT CREATED WITH THE MODERN SAMPLES 

mod_lab2<-aggregate(modern[,2:3],list(modern$V12),mean)
for(j in 1:100) 
{
  text(mod_lab2$V3[j],mod_lab2$V2[j],labels=mod_lab2$Group.1[j],cex=0.7)   
}

###### SPECIFYING THE UNKOWN SAMPLES - change the numbers accordingly based on the position in the final file! 

UNK1<-pca[c(1611),]
UNK2<-pca[c(1612),]
UNK3<-pca[c(1613),]
UNK4<-pca[c(1614),]
UNK5<-pca[c(1615),]
UNK6<-pca[c(1616),]
UNK7<-pca[c(1617),]
UNK8<-pca[c(1618),]
UNK9<-pca[c(1619),]
UNK10<-pca[c(1620),]
UNK11<-pca[c(1621),]
UNK12<-pca[c(1622),]
UNK13<-pca[c(1623),]
UNK14<-pca[c(1624),]
UNK15<-pca[c(1625),]
UNK16<-pca[c(1626),]


###### PLOTTING THE UNKNOWN SAMPLES 

UNK1$V12<-factor(UNK1$V12)
points(UNK1$V3,UNK1$V2,pch=17,cex=1.4,col="burlywood",bg="burlywood")
for(k in 1:35) 
  
{
  text(UNK1$V3[k],UNK1$V2[k],labels=UNK1$V12[k],pos=c(4),cex=0.9)   
}

####################################2
UNK2$V12<-factor(UNK2$V12)
points(UNK2$V3,UNK2$V2,pch=3,cex=1.4,col="yellow3",bg="yellow3")
for(k in 1:13) 
  
{
  text(UNK2$V3[k],UNK2$V2[k],labels=UNK2$V12[k],pos=c(3),cex=0.9)   
}

####################################3
UNK3$V12<-factor(UNK3$V12)
points(UNK3$V3,UNK3$V2,pch=4,cex=1.4,col="yellow2",bg="yellow2")
for(k in 1:13) 
  
{
  text(UNK3$V3[k],UNK3$V2[k],labels=UNK3$V12[k],pos=c(3),cex=0.9)   
}
####################################4
UNK4$V12<-factor(UNK4$V12)
points(UNK4$V3,UNK4$V2,pch=5,cex=1.4,col="palevioletred3",bg="palevioletred3")
for(k in 1:13) 
  
{
  text(UNK4$V3[k],UNK4$V2[k],labels=UNK4$V12[k],pos=c(1),cex=0.9)   
}

SOME_REFERENCE_POPULATION<-pca[c(97:102),]
#overlaying the ANCIENT REFERENCE samples
SOME_REFERENCE_POPULATION$V12<-factor(SOME_REFERENCE_POPULATION$V12)
points(SOME_REFERENCE_POPULATION$V3,SOME_REFERENCE_POPULATION$V2,pch=6,cex=1,col="green4")

```
Most likely you can guess where this is going, so just use this as a reference and write the R script for the rest of your unknown individuals.
After running it, you should get your nice projected PCA. Look at the output PDF, where are the unknown samples being projected? What clues does the PCA give you?

## Step 4 ADMIXTURE 

ADMIXTURE is a similar tool to STRUCTURE but runs much quicker, especially on large datasets.
ADMIXTURE runs directly from .bed or .ped files and needs no extra parameters for file preparation. You do not specify burin and repeats, ADMIXTURE exits when it converged on a solution (Delta< minimum value)

First, you have to load the module:

```
module load bioinfo-tools
module load ADMIXTURE/1.3.0
```
A basic ADMIXTURE run looks like this:

```
admixture -s time FINAL_DATASET_PRUNED.bed 2
```

This command executes the program with a seed set from system clock time, it gives the input file (remember the extension) and the K value at which to run ADMIXTURE (2 in the previous command).

For ADMIXTURE you also need to run many iterations at each K value, thus a compute cluster and some scripting is useful.

Make a script from the code below to run Admixture for K = 2-6 with 4 iterations at each K value.

```
#!/bin/bash
for kval in {2..6}; do
   for itir in {1..4}; do
           (echo '#!/bin/bash -l'
           echo "
           mkdir ${kval}.${itir}
           cd ${kval}.${itir}
#working folder where admixture and the bed files are
module load bioinfo-tools
module load ADMIXTURE/1.3.0
admixture -j3 -s $RANDOM path/to/your/FINAL_DATASET_PRUNED.bed ${kval}
cd ../
rm -r ${kval}.${int}
#moves it to a different folder and renames it so you end up with multiple iterations
exit 0") |
               sbatch -M snowy -p core -n 3 -t 72:0:0 -A YOUR_UPPMAX_PROJECT_HERE -J ADMX.${kval}.${itir} -o ADMX.${kval}.${itir}.output -e ADMX${kval}.${itir}.output
       done
done
```

You will need to **replace the project name** and the **path** to your pruned .bed-file in the script then just run it:

```
bash NAME_OF_THE_SCRIPT.sh
```
After a successful Admixture run, you should be seeing a bunch of new folders being created. 
In each one of them you will find the output ```P``` and ```Q``` for each iteration. 

*If you decide to work on PONG locally, you need to edit each ```Q``` file by adding the right k & iteration number to each one. After doing this for all folders, you can move all the ```Q``` files to a directory of your liking.*

## Step 5 PONG 

When the admixture analysis is finally done we use PONG to combine the different iterations and visualize our results. 

To be able to run PONG we thus need to generate **three different files**.

The first being the ***filemap***. This is the only input that is strictly required to run PONG. It consists of three columns.
From the PONG manual: 


```
Column 1. The runID, a unique label for the Q matrix (e.g. the string “run5_K7”).

Column 2. The K value for the Q matrix. Each value of K between Kmin and Kmax must
be represented by at least one Q matrix in the filemap; if not, pong will abort.

Column 3. The path to the Q matrix, relative to the location of the filemap. 
```

In order to create what we need we can edit and run the loop below. Make sure to edit the right number of **k** and **i**.
Also check whether the prefix of the admixture output Q files is the same in both the loop and the Q files. As long as you are starting PONG in the same directory of all the Admixture output files you should be ok.

```
for i in {2..7};
do
    for j in {1..3};
    do
    echo -e "k${i}_r${j}\t${i}\t${i}.${j}/FINAL_DATASET_PRUNED.${i}.Q" >> unknown_FILEMAP.txt
    done
done

```
Example of what this file looks like, depending on how many K and how many iterations you run:

```
k2_r1	2	2.1/FINAL_DATASET_PRUNED.2.Q
k2_r2	2	2.2/FINAL_DATASET_PRUNED.2.Q
k2_r3	2	2.3/FINAL_DATASET_PRUNED.2.Q
k3_r1	3	3.1/FINAL_DATASET_PRUNED.3.Q
k3_r2	3	3.2/FINAL_DATASET_PRUNED.3.Q
k3_r3	3	3.3/FINAL_DATASET_PRUNED.3.Q
k4_r1	4	4.1/FINAL_DATASET_PRUNED.4.Q
k4_r2	4	4.2/FINAL_DATASET_PRUNED.4.Q
k4_r3	4	4.3/FINAL_DATASET_PRUNED.4.Q
k5_r1	5	5.1/FINAL_DATASET_PRUNED.5.Q
k5_r2	5	5.2/FINAL_DATASET_PRUNED.5.Q
k5_r3	5	5.3/FINAL_DATASET_PRUNED.5.Q
k6_r1	6	6.1/FINAL_DATASET_PRUNED.6.Q
k6_r2	6	6.2/FINAL_DATASET_PRUNED.6.Q
k6_r3	6	6.3/FINAL_DATASET_PRUNED.6.Q
k7_r1	7	7.1/FINAL_DATASET_PRUNED.7.Q
k7_r2	7	7.2/FINAL_DATASET_PRUNED.7.Q
k7_r3	7	7.3/FINAL_DATASET_PRUNED.7.Q
```

The next file we need to create is the ***ind2pop*** file. It is just a list of which population each individual belongs to.
We have this information in the `.fam` file so we can just cut out the field we need:

```
cut -f 1 -d " " FINAL_DATASET_PRUNED.fam > unknown_IND2POP.txt

``` 
The ind2pop file should look exactly like the first column of the fam.

The ***poporder*** file is a key between what your populations are called and what "Proper" name you want to show up in your final plot. 
Also as the name suggests, **gives the order in which you want the populations to be in**, so order them as you think it makes most sense.

*Note that the file needs to be **tab-delimited**, i.e separated by tabs (If you have spaces issues may occure). 
Also, make sure within the names in this file to not have any spaces. If you do, try replacing them with underscores (_).*
*Again, be careful when editing this file, typos, extra spaces etc might cause problems.*

Below is just an example of what a poporder file should look like:

```
aDNA_Rick	pickle
aDNA_Morty	m0rty
aDNA_Ragnar	rgnr
aDNA_martians	mrt99
Napoleon_Bonaparte	nb001
Netherlands_BA	Dutch
```
An easy way of doing this is by :
```
cut -f 1 -d " " FINAL_DATASET_PRUNED.fam | uniq > unknown_POPORDER_prep.txt
```
and then just use this simple script:
```
#!usr/bin/env bash

file=$(cat unknown_POPORDER_prep.txt)

for line in $file
do
    echo -e "$line\t$line" >> unknown_POPORDER.txt
done
```
This way you will get a file with two columns. You can edit the the names of the populations you are interested in by changing the names of the second column.

### Now we have all the files we need. Time to run PONG.

*It is by far best to run PONG on your local computer. All you need to do is install PONG and copy the ```.Q``` files to your computer together with the three files (filemap,ind2pop,poporder) and run it. If you don't know how to install PONG(https://github.com/ramachandran-lab/pong), it is available through the module system on Uppmax, although very often it is quite laggy. 

* Mac Users - you will need to install XQuartz (an open-source version of the X.Org X server, a component of the X Window System that runs on macOS)https://www.xquartz.org
* Windows Users - MobaXterm https://mobaxterm.mobatek.net
* Ubuntu Users - I'm sure you can figure it out yourselves

```
module load pong 
```
**Super important:**
Since we are several people who are going to run PONG at the same time we need to use a different port, otherwise, we will collide with each other. The default port for PONG is 4000. Any other free port will work, like 4001, 2, etc. Make sure you are using a **unique port** before proceeding. If multiple people are trying to run PONG on the same port you have problems, so talk to your classmates and find a unique port in the 4000s that people aren't using.

```
pong -m unknown_FILEMAP.txt -i unknown_IND2POP.txt -n unknown_POPORDER.txt -g --port YOUR_PORT_NUMBER_HERE
```

When PONG is done it will start hosting a webserver that displays the results at port 4000 by default:  http://localhost:4000. Pong needs to be running for you to look at the results, i.e. **if you close it it will not work.**
It will look like this:

```
-------------------------------------------------------------------
                            p o n g
      by A. Behr, K. Liu, T. Devlin, G. Liu-Fang, and S. Ramachandran
                       Version 1.5 (2021)
-------------------------------------------------------------------
-------------------------------------------------------------------

Parsing input and generating cluster network graph
Matching clusters within each K and finding representative runs
For K=2, there are 3 modes across 3 runs.
For K=3, there are 3 modes across 3 runs.
For K=4, there are 3 modes across 3 runs.
For K=5, there are 3 modes across 3 runs.
For K=6, there are 3 modes across 3 runs.
For K=7, there are 3 modes across 3 runs.
Matching clusters across K
Finding best alignment for all runs within and across K
match time: 0.64s
align time: 0.01s
total time: 1.21s
-----------------------------------------------------------
pong server is now running locally & listening on port 4005
Open your web browser and navigate to http://localhost:4005 to see the visualization
```

The web server is hosted on the **same login node as you were running pong**. In case you are unsure of which one that is you can use the command `hostname` to figure it out:

```
hostname
```

To view files interactively you need to have an X11 connection. So when you connect to rackham from a new tab do:

```
ssh -AY YOUR_USERNAME_HERE@rackham.uppmax.uu.se
```

Make sure that you connect to **the same Rackham (i.e 1,2,3 etc) as you got from hostname**.  
In a **new tab** (if you didn't put PONG in the background) type:

```
firefox http://localhost:YOUR_PORT_NUMBER
```


Once you are finished with looking at the PONG output you can click and save some output to file and then close the program by hitting `ctrl - c` in the terminal tab running it. 

### What do the results from Admixture & PCA tell you? 
What do you see? What information does this give you about your misterious unknown populations? 
Do the results of your population PCA correspond to the population structure results you got from the ADMIXTURE plots? Can you figure out who they are?

**After you have finished with your analyses, delete all unnecessary files from your folder.**

# Part 3: Review & reflect

Okay, hopefully this project wasn't too difficult or too boring. Maybe it was even fun? And aside from that maybe you even learnt something. Regardless of how you felt about it, this project was constructed to provide you with a **real life** scenario of what most exploratory analysis our field of study look like, warts & all. More times than not you are working with something you don't know enough about (duh! - that's why it's called research) using techniques you're trying for the first time and failing on many attempts. Most of the manuals & how-toos you need to look for yourself, and references come in the shape of many scattered papers on the subject. And as you would do when you want to publish your findings in real life, the final step of the project is to write a 10-15 page report on your findings. Make sure to read up on the papers recommended above. They should provide you with enough of a backstory & will surely help you a lot in constructing the grand picture of things. You are free to also look into similar papers on the subject. Good luck & happy writing!
 
