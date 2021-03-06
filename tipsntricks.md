# Consensify tipsntricks

Axel Barlow and Johanna L. A. Paijmans, December 2018

This document assumes you have read the README and have successfully run the example dataset on your system. Here you will find additional info on further analyses, and considerations you may have when you are analysing your own data

## What filters should I use in angsd?

This is really up to you, based on the properties of your data and specific research question. As a starting point, you may wish to use map and base quality filters of 30. It is also possible to exclude transitions and/or terminal nucleotides which may be affected by cytosine deamination. However, consensify also reduces such errors and so we typically don't do this. For fragmentary genome assemblies, very small scaffolds may be problematic, so we generally exclude these (e.g. < 100kb or < 1MB)

## How do I decide the maximum acceptable depth?

This also also ultimately your decision, but we generally do use a max depth filter since regions with extremely high coverage may represent repetitive elements that the map quality filter failed to mitigate. Typically we use the upper 95th percentile of coverage as a cut off. You can calculate depth in angsd like this:

    angsd -bam bamlist.txt -doCounts 1 -doDepth 1 -maxDepth 200 -minQ 30 -minMapQ 30 -rf scaffolds_over_1MB.txt -out out

Note this should be run using the same filters applied for the base calls, so the coverage calculation is representative. bamlist.txt should be a list of the bam files you want to analyse. The -rf option allows you to exclude small scaffolds. See angsd documentation for full details. Several files are produced as output. One will be the individual coverage for each bam, from which you can determine the 95th percentile

## How do I run D statistics on the Consensify files?

There are many options, but we use Dr. James Cahill's excellent scripts written in C++ (https://github.com/jacahill/Admixture). Download and install the scripts, and you can run the Dstat like this:

    Dstat P1.fa P2.fa P3.fa P4.fa 5000000 > P1_P2_P3_P4.dstat

Where the first 4 arguments are the Consensify files to be used for the test. The 5th argument is the blocksize for weighted block jackknife. You can direct the output to a file with >. You can process the output with James' python scripts:

    python D-stat_parser.py P1_P2_P3_out.dstat

gives you D and the ABBA BABA counts

    python weighted_block_jackknife.py P1_P2_P3_out.dstat 5000000

gives you the standard error. Enter the block size after the input filename. D/SE give you the Z-score

See James' Github page for more details, and if you use his scripts then please cite the relevant papers. The f-hat script is also suited to the consensify files.

## How do I generate an input file for phylogenetic and other analysis?

Again there are many methods. We have written ReDuCToR; a script to combine the separate fasta output files from Consensify into a single alignment and remove all columns with invariant positions and missing data. The output is a fasta file which may be input to a variety of programs, such as R or RAxML. ReDuCToR is included with the Consensify distribution. It requires a basix UNIX toolkit (bash, awk, paste), samtools (Li et al. 2009) and snp-sites (Page et al. 2016). If you use ReDuCToR in your publication, then you must also cite these papers. 

USAGE (assuming the script is made executable first with `chmod +x ReDuCToR_v0.1.sh`:

    ./ReDuCToR_v0.1.sh $1 $2 $3

$1 is a file with the scaffold names of interest (generated by e.g. `awk '{print $1}' sample.fasta.fai > scaffolds.txt`)
$2 is a file with the fasta basenames (generated by e.g. `ls fasta.fa | cut -d'.' -f1 > fastanames.txt`).
$3 is the output filename.

NOTES: the variables $samtools and $snpsites should be changed to reflect your installation. Also, if you cancel the script halfway, remember to remove the `reductor_scaffolds/` folder before re-running it. If you do not do so, you will get a warning stating `contains sequences of unequal length.` but will still continue.

We often use the ape package in R to generate a distance matrix from the alignment under some model of nucleotide substitution, which can be used for principal coordinates analysis or neighbour-joining phylogenetic analysis. This can be done like this:

## How do I make figures in R?

Load ape library. Do `install.packages("ape")` if not already installed

    library(ape)

read in the alignment

    fa <- read.FASTA("alignment.fa")

generate the distance matrix, there are several models available

    mx <- dist.dna(fa, model="JC69")

Principal coordinates analysis

    mx.pcoa <- pcoa(mx)

plot PCos 1 and 2

    plot(mx.pcoa$vectors[,1:2], xlab="PCo1", ylab="PCo2", main="PCo using Consensify")

calculate NJ tree

    tree <- nj(mx)

root the tree, this assumes the outgroup is the first sequence in the alignment

    rtree <- root(tree, outgroup=1)

plot the tree

    plot(rtree, main="NJ tree using Consensify")

## References

* Korneliussen TS, Albrechtsen A, Nielsen R (2014) ANGSD : Analysis of Next Generation Sequencing Data. BMC Bioinformatics 15: 356.

* Li H, Handsaker B, Wysoker A, Fennell T, Ruan J, Homer N, Marth G, Abecasis G, Durbin R (2009) The Sequence Alignment/Map format and SAMtools. Bioinformatics 25:2078–2079 . doi: 10.1093/bioinformatics/btp352

* Page AJ, Taylor B, Delaney AJ, Soares J, Seemann T, Keane JA, Harris SR (2016) SNP-sites: rapid efficient extraction of SNPs from multi-FASTA alignments. Microb Genom 2: . doi: 10.1099/mgen.0.000056
