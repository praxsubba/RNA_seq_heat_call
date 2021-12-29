# RNA_seq_heat_call
## Heart
### First step is to download and arrange all files availabe in the Box folder "GC-ME-9475-294065772"
cat FASTQ*/*/*E24HR*fastq.gz >> /Users/subbaprakrit/Box/Prakrit\ Subba\ research/PS_Heat_call_RNA_seq_analysis/Heart/E24HR.fastq.gz
#### Note: Change E24HR from E8HR to E34HR 

### Remove the adapter contamination, polyA read through, and low quality tails

#PBS -N bbduk_heart 
#PBS -l select=1:ncpus=12:mem=62gb:interconnect=fdr,walltime=24:00:00 
#PBS -m abe 
#PBS -j oe

cd /home/psubba/Heart
for sample in E*.fastq; do cat $sample | /home/psubba/miniconda2/pkgs/bbmap-38.18-0/bin/bbduk.sh -Xmx10g in=stdin.fq out=${sample}_trimmed_clean.fastq int=f ref=/home/psubba/miniconda2/pkgs/bbmap-38.18-0/opt/bbmap-38.18/resources/polyA.fa.gz,/home/psubba/miniconda2/pkgs/bbmap-38.18-0/opt/bbmap-38.18/resources/truseq.fa.gz k=13 ktrim=r useshortkmers=t mink=5 qtrim=t trimq=10 minlength=20 2>> output.txt; done

### Create Genome Index for Reference genome using STAR
#PBS -N STAR_index 
#PBS -l select=1:ncpus=12:mem=62gb:interconnect=fdr,walltime=24:00:00 
#PBS -m abe 
#PBS -j oe

cd /zfs/jmgeorge/Prakrit/STAR_RNA_seq_heatcall
STAR --runThreadN 8 \
--runMode genomeGenerate \
--genomeDir /zfs/jmgeorge/Prakrit/STAR_RNA_seq_heatcall \
--genomeFastaFiles GCF_003957565.2_bTaeGut1.4.pri_genomic.fna \
--sjdbGTFfile GCF_003957565.2_bTaeGut1.4.pri_genomic.gtf \
--sjdbOverhang 99

### Mapping using array job for all the samples stored in sample.txt
#!/bin/bash

#PBS -N STAR_mapping_array
#PBS -l select=1:ncpus=12:mem=62gb:interconnect=fdr,walltime=24:00:00 
#PBS -j oe
#PBS -m abe
#PBS -M psubba@clemson.edu
#PBS -J 1-25

cd /zfs/jmgeorge/Prakrit/STAR_RNA_seq_heatcall
f=( $(sed -n ${PBS_ARRAY_INDEX}p samples.txt) )

STAR --runThreadN 8 --genomeDir /zfs/jmgeorge/Prakrit/STAR_RNA_seq_heatcall --readFilesIn /home/psubba/Heart/${f}HR.fastq_trimmed_clean.fastq \ 
--outFilterType BySJout --outFilterMultimapNmax 5 --alignSJoverhangMin 5 --alignSJDBoverhangMin 3 \
--outFilterMismatchNmax 10 --outFilterMismatchNoverLmax 0.1 --alignIntronMin 21 \ --alignIntronMax 0 --alignMatesGapMax 0 --outSAMattributes NH HI NM MD \
--outSAMtype BAM SortedByCoordinate --outFileNamePrefix ${f}_star_result --quantMode TranscriptomeSAM GeneCounts


echo 'JOB ENDED'        # prints to your output file

### Count reads mapped using featurecounts
#!/bin/bash

#PBS -N featurecounts_array
#PBS -l select=1:ncpus=12:mem=62gb:interconnect=fdr,walltime=24:00:00 
#PBS -j oe
#PBS -m abe
#PBS -M psubba@clemson.edu
#PBS -J 1-25

cd /zfs/jmgeorge/Prakrit/STAR_RNA_seq_heatcall
f=( $(sed -n ${PBS_ARRAY_INDEX}p samples.txt) )
featureCounts -a GCF_003957565.2_bTaeGut1.4.pri_genomic.gtf -F GTF -o ${f}_countMatrix.txt ${f}_star_resultAligned.sortedByCoord.out.bam
