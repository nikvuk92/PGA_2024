# PGA Course Autumn 2024

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
- A short presentation (tbd);
- the deadline for the written report is the day of the presentation.

## Unmasking the mystery of your 25 samples

The setup of this project is very similar to what you would do as a researcher in the field of (ancient) Human Genetics.
Therefore, it is up to you to identify the populations you have received and aided by literature on the subject as well as some historical context, try to infer possible demographic events. 
In order to achieve this, you are provided with a two reference datasets that should come in handy as a way to compare the unknown samples to the known ones. 
```
/crex/proj/uppmax2024-2-19/ANCIENT_REFERENCE_DATASET
/crex/proj/uppmax2024-2-19/MODERN_REFERENCE_DATASET
```
Helpful scripts are there to aid you on your quest and can be found:
```
/crex/proj/uppmax2024-2-19/SCRIPTS
```
In the steps that will lead you towards unmasking the unknown populations you will need to: 

1. Take a look at both Reference populations to get a feel of what you are working with. Check the genotyping rate of both; 
2. Check for related individuals in your **modern reference dataset**;
3. Filter the related individuals (if there are any) from the **reference dataset** - *we have nothing against family, but individuals that are in close kinship will affect our downstream analysis and we don't want that*;
4. Filter the **modern reference** individuals & SNPs
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

If working on Rackham load the following modules by typing in:
```
module load bioinfo-tools
module load plink/1.90b4.9
module load python/2.7.11
module load pong
module load KING

```
The software is already pre-installed on Rackham, you just have to load it. 
Heads up - every time you close your terminal, you will need to re-load the modules. If something is not working, a good sanity check is to see which modules are loaded :
```
module list
```

Okay, let's try running plink:
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
Also make sure to copy **all the scripts** over from the SCRIPTS folder into your directory.
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

*The filtering values below are just stand in values. They might be good picks or there might be better ones.*

Whatever you choose in the end try to rationalize your choices and report after each step, how many SNPs you had before and after each step, what the genotyping rates looked like, if individuals were removed or not etc.

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
## Step 4 Filtering out related individuals (just the Modern Reference dataset)

In the `SCRIPTS` folder, there is a script called `sbatch_KING.sh` that can be used to run [KING](http://people.virginia.edu/~wc9c/KING/manual.html) You can have a look inside for instructions on how to run the script. Check out the manual and try and figure out how the software works.
After you have run the script have a look at the produced output files and figure out how to remove the related individuals, if there are any

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
module load bioinfo-tools KING
king -b $1  --related
```
You can submit it as a job by doing:
```
sbatch -M snowy THIS_SCRIPT.sh YOUR_DATASET4.bed
```
Look at the output from KING & **keep** the unrelated individuals.

Checkpoint *At this point, if you exclude any related individuals you should be at YOUR_DATASET_5, otherwise go on with YOUR_DATASET_4*

## Step 5 Make the modern dataset haploid 
"Pseudohaploidizing" a TPED file typically involves converting a diploid genotype data file into a pseudohaploid format, where each variant or SNP (Single Nucleotide Polymorphism) is represented only once, as if the genotype data came from a haploid individual. This is mainly employed when working with ancient DNA. The script that we are using selects one allele for each SNP marker for each individual. There are multiple ways for the allele selection process but it must be consistent across the dataset. Can you think about why this is a good idea to do when dealing with ancient DNA? 

you can run the script on the modern dataset (in an interactive node) as shown below. You need to load ```python/2.7.11``` as module first:
```
haploidize_tped.py < YOUR_DATASET_4.tped > YOUR_DATASET_4_haploid.tped
```
As you can see, your file needs to be in .tped format. It will take some time to run because it goes through each SNP position in your file. After it is done, the file YOUR_DATASET_4_haploid.tped is the haploid version of YOUR_DATASET_4.tped. Therefore you can either rename YOUR_DATASET_4.tped into something different (or if you are certain, just delete it) and rename the haploid version to just YOUR_DATASET_4.tped. This way, all the files that accompanied the original tped will work for your "new" version.

Now that you have made sure you have a haploid dataset, if you are curious you can obtain the observed and expected heterozygosities and look at the HWE stats by using this command 

```
plink --bfile YOUR_DATASET_4 --hardy --out hardy_YOUR_DATASET_4
```
Look at file hardy_YOUR_DATASET_4.hwe !

P.S. There is no need to make the ancient reference dataset pseudohaploid because we have already made sure that it is for you.

## Step 6 Plot a PCA using the Modern reference individuals
Plot a PCA using plink on the Modern reference individuals and describe what you see.

```plink --bfile YOUR_DATASET_4 --pca```

The PCA analysis should give you an eigenvec and an eigenval file as outputs. Plot them in R and include this PCA in your report.
Hmmm there is an outlier in the PCA that definitely doesn't seem to be making sense. Perhaps a mislabelled sample? Remove that one and re-run the PCA. (while it is not the best practice to just remove outliers you don't like, this is a controlled environment and removing the misplaced individual is part of the exercize :)) 
Now re-plot the and hopefully it should look better! Also plot the different PCs (as barplots) to show how much of the variation each one explains. Try to describe what each PC represents (up to some reasonable PC ex.PC5).

## Step 7 Before merging let's first unify SNP names 
When merging datasets from different chips, the same position can have different names on different chips. The rename_SNP.py script can be used on the .bim files to change the SNP name into the names based on position. Do this for both reference datasets and your unknown dataset. Check the bim files prior to making the change and see the SNP names before they get renamed.

In order to run rename_SNP.py first load ```module load python/2.7.11``` (Issues might occur if not using the right version of Python).

```python rename_SNP.py FILE.bim```

After which you should get that the output with replaced SNP names has been written to FILE.bim_pos_names. You can delete the original bim (or rename it if you're feeling unsure) & edit the 

```FILE.bim_pos_names``` to become ```FILE.bim```.

Now check what the bim files look like after the changes.

### Step 8 Merging the reference datasets to eachother

*P.S. As long as you have all the files/datasets in the same directory you don't need to specify the full paths. If they are in different datasets, remember to add the full path to the file, such as:
```/proj/uppmaxproject/full_path_to_YOUR_OWN_directory/Reference_datasets/datasetA/file```

*P.P.S. Do this and other heavy lifting in an **interactive node!**

Merging is done using plink's bmerge command:
```plink --bmerge FILE_A --bfile FILE_B --out FILEA+B```

### Step 9 Merge the Unknown dataset to the reference-combined-dataset
Finally, after you've merged the two reference datasets together, we need to merge in the Unknowns as well. To do this, as previously, 
Again did the missigness change in any way? Let's call this one ```COMPLETE_DATASET```.

### Filter missing genotypes 
``` --geno ``` filters out all variants with missing call rates exceeding some provided value. Using a value like the one below is a quick way of making our dataset lighter.
```
plink --bfile COMPLETE_DATASET --geno 0.99 --make-bed --out COMPLETE_DATASET_GENO 
```
Think about why the bulk of the filtering was done on the modern dataset and why we are very cautious now. 

## Step 10 Remove all unnecessary iterations of the same data except .log files from the merging/filtering steps and of course the final dataset
As our storage on uppmax is quite limited, this is a good point to free up your space from all the data that was produced from each of the previous steps and just keep the final dataset.  
Feels stupid to say, but please do not delete the files you need downstream.

## Step 11 Reflect and review
How many SNPs are you working with? Make sure you report how the genotype rates, SNP counts and other important parameteres changed based on your filter strategy.
Now you have generated your input files for the next exercise which will deal with population structure analysis on the combined dataset. You will look at the population structure of your unknown samples in comparison to known known reference populations (such as Human Origins).

=============================================================================

# PART 2: Population structure inference 

**Word of advice - the following analyses might take quite some time to run. Therefore, plan accordingly and leave yourself enough room for error.**

Furthermore, make sure to doublecheck your script before running it. 90% of issues you will encounter when running scripts are going to be typos (*Sanchez,R., Smith,M., 2022*). Check if you have written the correct file names, the correct extensions, whether you have the right path to your file or whether you have typed in your correct project name. Since we have a limited CPU hours allocated for this course it is really important to be efficient!  

## Step 1 LD Pruning 

Before running PCA & ADMIXTURE it is advisable to prune the data to thin the marker set for linkage disequilibrium.

Why LD Pruning is Important:
- Reducing Redundancy:
Linked SNPs: SNPs that are in linkage disequilibrium (LD) are not independent of each other, meaning that their allele frequencies are correlated due to physical proximity on the chromosome.
Pruning: By removing SNPs in high LD, you reduce redundancy in your dataset, which helps ensure that the remaining SNPs are more independent.
- Improving Statistical Power:
PCA and ADMIXTURE: Analyses like Principal Component Analysis (PCA) and ADMIXTURE can be skewed by linked SNPs because these methods assume independence between markers. LD pruning helps avoid bias and overestimation of population structure or admixture components.
Avoiding Overfitting: Pruning helps prevent overfitting in models by removing highly correlated variables (SNPs).
- Computational Efficiency:
Smaller Dataset: Pruning reduces the number of SNPs, making the dataset smaller and more manageable. This can lead to faster computation times and reduced memory usage, especially in computationally intensive analyses.

You can read up on how to prune for LD(https://dalexander.github.io/admixture/admixture-manual.pdf).

For example, ```--indep-pairwise 50 10 0.5``` :
- Window Size (50): This means PLINK will examine LD between SNPs within a sliding window of 50 SNPs. A smaller window size would prune less aggressively, but a window of 50 SNPs is generally considered a good balance between capturing local LD and not being too strict.
- Step Size (10): The window will slide across the genome in steps of 10 SNPs. This step size allows for a reasonable balance between computational efficiency and thoroughness in detecting SNP pairs in high LD.
- r² Threshold (0.5): SNP pairs with an r² greater than 0.5 within the window will have one SNP removed. An r² of 0.5 is a moderate threshold that ensures SNPs in very high LD (strongly correlated) are pruned, while keeping more SNPs that are only moderately correlated.

You can try the following:
```
plink --bfile COMPLETE_DATASET_GENO --indep-pairwise 10 10 0.5
```


this creates some output files. Check what they contain.


```
plink --bfile COMPLETE_DATASET_GENO --extract plink.prune.in --make-bed --out FINAL_DATASET
```
Check how much you're pruning out. Perhaps you can tweak the parameters if you are pruning out too much! 

## Step 2 Plink PCA

Quickly run a conventional PCA on your FINAL_DATASET using plink and try to visualize it. This is just for comparison purposes, so don't dwell on it for too long.
(you need the bed version of your FINAL_DATASET to run this)

```

plink --bfile FINAL_DATASET --pca --out FINAL_DATASET_PCA

```

## Step 3 Projected PCA

There is a reason why we are favouring a projected PCA instead of a "conventional" PCA. It would be good to think about why we have decided to do this step. Maybe read up a bit on what it is and what it does.
But it is surely something to do with the specific nature of the samples we are dealing with.

For running a projected PCA, you first need to produce some files:

  - a tped file (your final merged dataset); 
  - a list of the modern populations in the dataset;
  - a list of the unknown populations in the dataset;

To produce the tped, you can use plink as you've done so far. 

And to produce the two population lists you can use a combination of awk, grep, sed, sort, uniq or whatever combination of those you choose.
Ex.
```
awk {'print $2'} FINAL_DATASET.tfam |sort|uniq > Sorted_list_of_the_populations.txt
```
Then, this sorted list of populations, you can subset to an ```unknown.txt``` and a ```modern.txt``` using any approach you like.  But I suggest looking at the fam/tfam file to think of a smart way to do this in one quick step. (Look at the patterns you might grep on!)

Whatever you end up choosing, the important thing is that:
- All the "UNK" samples go in the UNKNOWN_SAMPLES.txt
- All the ancient samples that came from the ANCIENT_REFERENCE_DATASET.fam also go into the UNKNOWN_SAMPLES.txt
- All the modern samples from MODERN_REFERENCE_DATASET.fam go into the MODERN_SAMPLES.txt

**It is crucial that there are no duplicate entries in the both of these population files!**
*Note Populations IDs in "MODERN_REFERENCE_DATASET" and "UNKNOWN_SAMPLES" must exactly match the IDs in the tfam.*

Hint: Check the ```pca_lsqproj.sh``` file if it is calling the following modules:

```
module load bioinfo-tools
module load plink/1.90b4.9
module load python/2.7.11
module load eigensoft/7.2.0
```

Now make a ```tped``` version of the FINAL_DATASET and you can go ahead & submit a sbatch job:

```
sbatch -A correctUPPMAXproject -M snowy -p core -n 2 -t 07:00:00 -J PCA_lsq -e LSQ.er pca_lsqproj.sh FINAL_DATASET MODERN_SAMPLES.txt UNKNOWN_SAMPLES.txt
```
This may take a few hours. You can check whether the job is still running with ```jobinfo```.

```jobinfo -M snowy -u YOUR_USERNAME```

When it's done, assuming it has ended sucessfuly, copy the ```.evec``` and ```.eval``` files to your computer's R directory and plot the PCA.

You will surely think of a better way to do this but in case you are stuck, below is an example that works. Make sure to edit the correct numbers and names in order to fit your data.

```
library(ggplot2)

####### LSQ PROJECTION PCA PLOT 
pca<-as.data.frame(EXAMPLE.popsubset.evec)

####### DIVIDE THE TABLE INTO MODERN & ANCIENT REFERENCE POPULATIONS - make sure to edit the right numbers! ########

modern<-pca[c(1:100),]
modern$V12<-factor(modern$V12)
modern$V2<-as.numeric(modern$V2)
modern$V3<-as.numeric(modern$V3)
ancient<-pca[c(100:200),]

###### PLOTTING THE MODERN REFERENCE POPULATIONS
plot(modern$V2,modern$V3,pch=20,cex=2.4,col = rgb(0, 0, 0, 0.15),xlab="PCA1",ylab="PCA2")

###### LABELING THE PLOT CREATED WITH THE MODERN REFERENCE POPULATIONS

for(k in 1:length(unique(as.character(modern$V12)))){
  text(y=mean(modern$V3[as.character(modern$V12)==unique(as.character(modern$V12))[k]]),x=mean(modern$V2[as.character(modern$V12)==unique(as.character(modern$V12))[k]]),
       labels=unique(as.character(modern$V12))[k],cex=0.6,col="gray66")   
}

###### SPECIFYING THE ANCIENT REFERENCE POPULATIONS - change the numbers accordingly based on the position in the final file!

REF_POP_1<-pca[c(797:812),]
REF_POP_2 <-pca[c(653:657),]

###### SPECIFYING YOUR SAMPLES OF INTEREST

UNK_SAMPLES<-pca[(900:925),]

###### PLOTTING THE ANCIENT REFERENCE POPULATIONS

REF_POP_1$V12<-factor(REF_POP_1$V12)
points(REF_POP_1$V2,REF_POP_1$V3,pch=15,cex=1,col="red",bg="red")

REF_POP_2$V12<-factor(REF_POP_2$V12)
points(REF_POP_2$V2,REF_POP_2$V3,pch=4,cex=1,col="orange",bg="orange")

###### PLOTTING YOUR SAMPLES OF INTEREST ######

UNK_SAMPLES$V12<-factor(UNK_SAMPLES$V12)
points(UNK_SAMPLES$V2,UNK_SAMPLES$V3,pch=17,cex=1.4,col="skyblue",bg="skyblue")
for(k in 1:25) 
  
{
  text(UNK_SAMPLES$V2[k],UNK_SAMPLES$V3[k],labels=UNK_SAMPLES$V12[k],pos=c(4),cex=0.6)   
}

###### MAKE SURE TO CREATE A NICE LEGEND AS WELL!

```
The example was for two reference populations but do plot all the reference populations you think are relevant to your dataset. The more the merrier!
After running this in R, you should get your nice projected PCA (what are we projecting anyways?). What clues does this PCA give you?

## Step 5 ADMIXTURE 

ADMIXTURE is a similar tool to STRUCTURE but runs much quicker, especially on large datasets.
ADMIXTURE runs directly from .bed or .ped files and needs no extra parameters for file preparation. You do not specify burin and repeats, ADMIXTURE exits when it converged on a solution (Delta< minimum value)

First, you have to load the module:

```
module load bioinfo-tools
module load ADMIXTURE/1.3.0
```
A basic ADMIXTURE run would look like this (do not run this):

```
admixture -s time EXAMPLE.bed 2
```

This command executes the program with a seed set from system clock time, it gives the input file (remember the extension) and the K value at which to run ADMIXTURE (2 in the previous command).

For ADMIXTURE you also need to run many iterations at each K value, thus a compute cluster and some scripting is useful.

Make a script from the code below to run Admixture for K = 2-6 with 4 iterations at each K value. Note that you are working with the ```FINAL_DATASET.bed`` file here (not the Tped)

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
admixture -j3 -s $RANDOM /the/path/to/your/FINAL_DATASET.bed ${kval}
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
sbatch -A YOURPROJECTNAME -M snowy -p core -n 4 -t 1-00:00:00 the_admixture_script_you_just_created.sh
```
After a successful Admixture run, you should be seeing a bunch of new folders being created. 
In each one of them you will find the output ```P``` and ```Q``` for each iteration. 

*If you decide to work on PONG locally, you need to edit each ```Q``` file by adding the right k & iteration number to each one. After doing this for all folders, you can move all the ```Q``` files to a directory of your liking.*

## Step 6 PONG (Visualizing step 5)

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
    echo -e "k${i}_r${j}\t${i}\t${i}.${j}/FINAL_DATASET.${i}.Q" >> unknown_FILEMAP.txt
    done
done

```
Example of what this file looks like, depending on how many K and how many iterations you run:

```
k2_r1	2	2.1/FINAL_DATASET.2.Q
k2_r2	2	2.2/FINAL_DATASET.2.Q
k2_r3	2	2.3/FINAL_DATASET.2.Q
k3_r1	3	3.1/FINAL_DATASET.3.Q
k3_r2	3	3.2/FINAL_DATASET.3.Q
k3_r3	3	3.3/FINAL_DATASET.3.Q
k4_r1	4	4.1/FINAL_DATASET.4.Q
k4_r2	4	4.2/FINAL_DATASET.4.Q
k4_r3	4	4.3/FINAL_DATASET.4.Q
k5_r1	5	5.1/FINAL_DATASET.5.Q
k5_r2	5	5.2/FINAL_DATASET.5.Q
k5_r3	5	5.3/FINAL_DATASET.5.Q
k6_r1	6	6.1/FINAL_DATASET.6.Q
k6_r2	6	6.2/FINAL_DATASET.6.Q
k6_r3	6	6.3/FINAL_DATASET.6.Q

```

The next file we need to create is the ***ind2pop*** file. It is just a list of which population each individual belongs to.
We have this information in the `.fam` file so we can just cut out the field we need:

```
cut -f 1 -d " " FINAL_DATASET_PRUNED.fam > unknown_IND2POP.txt

``` 
The ind2pop file should look exactly like the first column of the fam.

The ***poporder*** file is a key between what your populations are called and what "Proper" name you want to show up in your final plot. 
Also as the name suggests, **gives the order in which you want the populations to be in**, so order them as you think it makes most sense. For example if you want to show the transition between the Neolithic and the Mesolithic, you could plot all the Neolithic populations next to each other, followed by all the Mesolithic ones, or you could choose geography, or whatever you like.

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
cut -f 1 -d " " FINAL_DATASET.fam | uniq > unknown_POPORDER_prep.txt
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
 
