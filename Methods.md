# Methods

## Preprocessing

### PRJNA698267

All SRA datasets in PRJNA698267 were trimmed using TrimGalore v0.6.7 using default adapter detection, and microbial profile detected using fastv.

```
#!/bin/bash
 
# Declare an array of string with type
declare -a StringArray=("SRR13615939.lite.1" "SRR13615940.1" etc...)

for val in ${StringArray[@]}; do
	fastq-dump $val
	~/apps/TrimGalore-0.6.7/trim_galore $val"".fastq  -j 4
done
```

SRR13616010 raw read quality was checked using FastQC and microbial profile generated using fastv

```
fastqc SRR13616010.lite.1_trimmed.fq -o fastqc -t 24

fastv -i SRR13616010.lite.1_trimmed.fq -c microbial.kc.fasta.gz -h fastv/SRR13616010_microbial_fastv.html -j fastv/SRR13616010_microbial_fastv.json -R "SRR13616010_microbial_fastv" -w 8
```

### PRJNA802993

SRR17868030 was trimmed and filtered using fastp with default settings and microbial profile generated using fastv.

```
fastp -i SRR17868030_1.fastq -o SRR17868030_1_fastp.fq -I SRR17868030_2.fastq -O SRR17868030_2_fastp.fq --detect_adapter_for_pe -w 16 -h SRR17868030_fastp.html -j SRR17868030_fastp.json

fastv -i SRR17868030_1.fastq -I SRR17868030_2.fastq -c microbial.kc.fasta.gz -h SRR17868030_microbial_fastv.html -j SRR17868030_microbial_fastv.json -R 'SRR17868030_microbial_fastv' -w 16

```

### PRJNA806767

SRR18012762 was trimmed and filtered using fastp with default settings and microbial profile generated using fastv.

```
fastp -i SRR18012762_1.fastq -o SRR18012762_1_fastp.fq -I SRR18012762_2.fastq -O SRR18012762_2_fastp.fq --detect_adapter_for_pe -w 16 -h SRR18012762_fastp.html -j SRR18012762_fastp.json

fastv -i SRR18012762_1.fastq -I SRR18012762_2.fastq -c microbial.kc.fasta.gz -h SRR18012762_microbial_fastv.html -j SRR18012762_microbial_fastv.json -R 'SRR18012762_microbial_fastv' -w 16
```

## Alignment
All SRA datasets were aligned to the SARS-CoV-2 Wuhan-Hu-1 (NC_045512.2) genome with poly(A) tail removed using the notebook
```
Alignment_minimap2_raw_reads.ipynb 
```
## Consensus sequences

### PRJNA806767

A consensus sequence for the SRR18012762 dataset after alignemnt to SARS-CoV-2 Wuhan-Hu-1 was generated using the following commands: 

```
~/apps/ngsutils/ngsutilsj bam-removeclipping -f input.bam  ngsutils_removeclipping.bam

~/apps/samtools-1.15.1/bin/samtools mpileup -B -A -aa -d 0 -Q 0 --reference NC_045512.2.fa ngsutils_removeclipping.bam | gzip -9 > mpileup_BAaad0Q0.txt.gz

zcat mpileup_BAaad0Q0.txt.gz | ivar consensus -m 5 -q 30 -n N -t 0.5 -p mpileup_BAaad0Q0_ivar_m5_q30.fa
```

### PRJNA802993

A consensus sequence for the SRR17868030 dataset (supporting EPI_ISL_462306) was generated using samclip (Seeman, 2018) to remove reads with a softclip length of more than 10, then using ngsutils to remove clipped ends of reads, followed by the following commands: 

```
samclip --ref NC_045512.2.fa in1.sam --max 10 > out.sam

ngsutilsj bam-removeclipping -f input.bam out_bam-removeclipping.bam

samtools sort -O BAM -o sorted.bam in.sam

samtools mpileup -B -A -aa -d 0 -Q 0 --reference  NC_045512.2.fa sorted.bam | gzip -9 > SRR17868030_mpileup_BAaad0Q0.txt.gz

zcat SRR17868030_mpileup_BAaad0Q0.txt.gz | ivar consensus -m 5 -q 30 -n N -t 0.5 -p SRR17868030_mpileup_BAaad0Q0_ivar_m5_q30.fa
```

### SRR13616010

A consensus genome for SRR13616010 generated using samtools and ivar v1.3.1 (Grubaugh
 Et al. 2018) using the following commands:

```
samtools mpileup -A -aa -d 0 -Q 30 --reference NC_045512.2.fa SRR13616010_aligned.bam | gzip -9 > SRR13616010_mpileup_AaadQ30.txt.gz

zcat SRR13616010_mpileup_AaadQ30.txt.gz | ivar consensus -m 10 -n N -t 0.5 -p SRR13616010_ivar_consensus_AaadQ30.fa
```

Variants were called with ivar using the following command:

```
zcat SRR13616010_mpileup_AaadQ30.txt.gz | ivar variants -m 10 -r NC_045512.2.fa -g NC_045512.2.gff3 -p SRR13616010_AaadQ30_ivar_var.tsv
```

## Mapping statistics

Read mapping statistics for SRR18012762 were calculated after alignment to NC_045512.2 with poly(A) tail removed. NGSutils (Breese and Liu, 2013) was used to remove soft clipped ends of mapped reads using the following command: 

```
ngsutilsj bam-removeclipping -f mapped.bam mapped_clipped.bam. 
```

Samtools (reference) was then used to calculate statistics using the following command: 

```
samtools stats mapped_clipped.bam --reference NC_045512.2.fa >SRR18012762_clipped_samtools_stats.txt
```

## Variant QC

Nucleotide mappings at positions 8782, 28144 and 29095 was calculated on all trimmed bams for all all SRA datasets in PRJNA698267 and PRJNA612766 using the following notebook:
```
Mutation_stats.ipynb
```