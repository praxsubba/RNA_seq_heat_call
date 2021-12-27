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

#### Need to create one common fasta file for genome mapping
#PBS -N STAR_index 
#PBS -l select=1:ncpus=12:mem=62gb:interconnect=fdr,walltime=24:00:00 
#PBS -m abe 
#PBS -j oe

cd /zfs/jmgeorge/Prakrit/STAR_RNA_seq_heatcall
for sample in *clean.fastq ; do \
STAR --runThreadN 8 --genomeDir /zfs/jmgeorge/Prakrit/STAR_RNA_seq_heatcall --readFilesIn ${sample} \
--outFilterType BySJout --outFilterMultimapNmax 20 --alignSJoverhangMin 8 --alignSJDBoverhangMin 1 \
--outFilterMismatchNmax 999 --outFilterMismatchNoverLmax 0.1 --alignIntronMin 20 \
--alignIntronMax 1000000 --alignMatesGapMax 1000000 --outSAMattributes NH HI NM MD
--outSAMtype BAM SortedByCoordinate --outFileNamePrefix ${sample}_star_result ;\
done
