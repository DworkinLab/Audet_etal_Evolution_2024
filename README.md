# README

This repository includes all scripts and phenotypic data to reproduce the analyses associated with the manuscript:
Audet T, Krol J, Pelletier K, Stewart AD, Dworkin I. Sexually discordant selection is associated with trait specific morphological changes and a complex genomic response. Evolution. 2024 May 9:qpae071. doi: 10.1093/evolut/qpae071. Epub ahead of print. PMID: 38720526.


A static copy of this repository (frozen at time of acceptance of the paper) is available on [DRYAD](https://datadryad.org/stash/dataset/doi:10.5061/dryad.6t1g1jx6k).

A [previous pre-print of the manuscript is available here](https://www.biorxiv.org/content/10.1101/2023.08.31.555745v2).

In addition to the phenotypic data, raw sequence data is available through [NCBI SRA, BioProject PRJNA1107500](https://www.ncbi.nlm.nih.gov/sra/PRJNA1107500). This includes the sequence generated for Audet et al. as well as for the raw sequences from:
Turner TL, Stewart AD, Fields AT, Rice WR, Tarone AM. Population-based resequencing of experimentally evolved populations reveals the genetic basis of body size variation in Drosophila melanogaster. PLoS Genet. 2011 Mar;7(3):e1001336. doi: 10.1371/journal.pgen.1001336. Epub 2011 Mar 17. PMID: 21437274; PMCID: PMC3060078.


## analysis

For the phenotypic analyses and experiments, raw data is in the data folder, consisting of two files,`JK_Feb2020_legsThorax_SSD.csv` for the morphological data and `sex_ratio_cross.csv` for adults sex ratio. The associated scripts are in the `Rscripts` folder, specifically the `Phenotype_analysis.Rmd` for the analysis of morphological changes and the `SexRatio_F1_F2.Rmd` for the analysis of changes in sex ratios. Please see the file `MetaData.csv` for meaning of variable names in the raw data files.

For the genomic analysis, below we summarize the major steps of the analysis in this readme, with the associated scripts we used to run the analyses (on the cluster we used) in `bash_scripts` and `Rscripts`. 

For the simulations using SLiM, the scripts can be found in the folder `simulationScripts`, with the `SeperateSexesBurnIn.slim` script used to produce the founding population that was sampled from (run only once), and the script `SeperateSexesOutputSamples_Run_OnePop_local.slim` to generate single populations (run 100 times) to simulate the LH_M and artificial selection undergoing the demographic changes for these populations, used to assess patterns of between sex FST.

# 1) Trimming was done with bbduk (bbmap v. 38.86)
```
bbduk.sh \
in1=R1.fastq \
in2=R2.fastq \
out1=R1_trimmed.fastq \
out2=R2_trimmed.fastq \
ref=AllAdapters.fa \
threads=32 ftr=149 ktrim=r k=23 mink=7 hdist=1 tpe tbo \
qtrim=rl trimq=20 minlength=36
```
# 2) Genome was mapped using bwa-mem v. 0.7.17 to Drosophila reference genome 6.23
```
bwa mem -t 32 \
-M ref.fa \
R1_trimmed.fastq \
R2_trimmed.fastq \
> mapped.sam
```

# 3) Sam files were converted to bam files using Samtools v. 1.12 at the same time the core genome was extracted to filter out other contigs including reads that mapped to commensal genomes
```
samtools view -h -@ 32 in.sam | \
awk '{if ($3 == "2L" || $3 == "2R" || $3 == "3L" || $3 == "3R" || $3 == "4" || $3 == "X" || \
$2 == "SN:2L" || $2 == "SN:2R" || $2 == "SN:3L" || $2 == "SN:3R" || $2 == "SN:4" || $2 == "SN:X") {print $0}}' | \
samtools view -b -@ 32 -o out.bam
```


# 4) Supplimentary reads from an additional run of sexuencing were merged together with samtoold v. 1.12

This was done by first seperating each run of sequencing in to two directories called run1 and run2, and finally merging these directories in to a final directory called merged. This method was used to merge suppletary runs of sequencing together as well as to merge sexes for analyses where sexes are pooled together.
```
samtools merge merged.bam \
run1/*.bam \
run2/*.bam
```
# 5) Mark and then remove read groups using Samtools v. 1.12

Done in four stages. Files are first sorted by name, then fixmate is used to add quality tags to reads. Then files are sorted by coordinate and mardup is used to mark the duplicates with the -r flags to remove those duplicate reads.

```
samtools sort -n -@ 32 -o out.bam in.bam

samtools fixmate -m -u -@ 32 in.bam out.bam

samtools sort -@ 32 -o out.bam in.bam

samtools markdup -l 150 -r -s -f stats.txt -d 2500 -@ 32 in.bam out.bam
```

# 6) Read-groups were added using picard v 2.26 and then indels were marked and realigned around using GATK v. 3.8

```
java -jar -Xmx10g /picard.jar AddOrReplaceReadGroups \
INPUT=in.bam \
OUTPUT=out_RG.bam \
SORT_ORDER=coordinate \
RGID=library \
RGLB=library \
RGPL=illumina \
RGSM=Stewart \
RGPU=library \
CREATE_INDEX=true \
VALIDATION_STRINGENCY=SILENT
```
```
java -Xmx32g -jar /GenomeAnalysisTK.jar -I in_RG.bam \
-R /ref/dmel-all-chromosome-r6.23.fasta \
-T RealignerTargetCreator \
-o out.intervals
```
```
java -Xmx10g -jar /GenomeAnalysisTK.jar -I in_RG.bam \
-R /ref/dmel-all-chromosome-r6.23.fasta \
-T IndelRealigner -targetIntervals in.intervals \
-o out.bam
```

# 7) Samtools v. 1.12 was used to create an mpileup with sequence and SNP quality thresholds set to 20 and maximum depth set to 1.5x expected depth to remove suspiciously high coverage areas.
```
samtools mpileup -Q 20 -q 20 -d 300 \
-f /ref/dmel-all-chromosome-r6.23.fasta \
in.bam \
-o out.mpileup
```
mpileup files were created for 1) treatments where sexes were combined 2) treatments where sexes were kept seperately 3) replicates and sexes combined

# 8) Repeteive regions were removed using popoolation v. 1.2.2.
These regions were the known transposable elements in the reference genome version 6.23, other "blacklisted" regions that have been shown to cause issues in SNP calling in the drosophila genome (Amemiya et al. 2019; https://github.com/Boyle-Lab/Blacklist/blob/master/lists/dm6-blacklist.v2.bed.gz), and a created bedfile to isolate regions that show suspiciously high inter-sex Fst and were verified to be transposable or repetative elements.
```
curl -O http://ftp.flybase.net/genomes/Drosophila_melanogaster/dmel_r6.23_FB2018_04/fasta/dmel-all-transposon-r6.23.fasta.gz
```

## First Identify repeats using RepeatMasker v. 4
```
/path/to/RepeatMasker \
-pa 20 \
--lib /path/to/transposons/dmel-all-transposon-r6.23.fasta \
--gff /path/to/reference/genome/dmel-all-chromosome-r6.23.fasta	
```

## Next remove repetetive regions using popoolation v. 1.2.2.
```
perl /popoolation_1.2.2/basic-pipeline/filter-pileup-by-gtf.pl \
--gtf /ReapeatMasker/output/Dmelgenome/dmel-all-chromosome-#r6.23.fasta.out.gff \
--input in.mpileup \
--output out.mpileup
```
# 9) Identify indels using Kapun scripts
```
Kapun_IDindels.sh

```
A sync file is created from this mpileup that contains allele frequencies at all positions using `grenedalf sync`

# 10) SNP calling was performed using poolSNP v. 1

This was done chromosome by chromosome to save memory. So `awk` was first used to break mpileup in to chromosomes.

This was done with: 1) all samples seperate (to look for sex specific SNPs), 2) sexes merged (For CMH testing)

```
bash /PoolSNP-master/PoolSNP.sh \
mpileup=in.mpileup \
reference=/ref/all_ref.fa \
names=C,E,L,S \
max-cov=0.98 \
min-cov=60 \
min-count=20 \
min-freq=0.01 \
miss-frac=0.2 \
jobs=32 \
BS=1 \
output=out.vcf
```

# 11) This VCF was filtered for the ENCODE blacklist using bedtools v. 2.3, and also had the previously identified Indels removed with customs scripts from Kapun et al.
```
bedtools intersect -v -a in.vcf \
-b /blacklist/dm6-blacklist.v2.bed \
> out.vcf
```

```
Kapun_filterindels.sh
```

## Using custom scripts we subset the all positions sync file with out SNP called clean vcf

```
subset_syncByVCF.sh
```

This is due to inconsistent vcf formatting output by `poolSNP` and not consistent with what `grenedalf fst` requires. This SNP called sync file is used for Fst, diversity measures, and models

## Calculate Fst from sync with all sexes and replicates merged (so just treatment)

```
/path/to/grenedalf/bin/grenedalf fst \
--window-type sliding \
--window-sliding-width 10000 \
--method unbiased-nei \
--pool-sizes 400 \
--threads 32 \
--sync-path /home/audett/scratch/SSD/Analysis/repsMerged/syncs/repsMerged.sync \
--sample-name-list C,E,L,S \
--omit-na-windows \
--out-dir /home/audett/scratch/SSD/Analysis/repsMerged/syncs/ \
--file-prefix repsMerged

```

# and for between sex Fst

```
/path/to/grenedalf/bin/grenedalf fst \
--window-type sliding \
--window-sliding-width 5000 \
--method unbiased-nei \
--pool-sizes 100 \
--threads 16 \
--sync-path /home/audett/scratch/SSD/Stewart/Stewart_withIndels.sync \
--sample-name-list C1F,C1M,C2F,C2M,E1F,E1M,E2F,E2M,L1F,L1M,L2F,L2M,S1F,S1M,S2F,S2M \
--omit-na-windows \
--out-dir /home/audett/scratch/SSD/Stewart/ \
--file-prefix sexFst

```
look for overlap in these bed files

````
bedtools intersect -header -u -a SNPs.vcf -b ./top5percent_fst.bed ./significant_CMH.bed low_pi.bed > ./sites_of_interest.bed

````

# Finally, model SNPs in each sites_of_interest file with R scripts
Please see the readme file in the `./Rscripts` folder for explanations of each R script.

