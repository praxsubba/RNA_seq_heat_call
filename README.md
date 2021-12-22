# RNA_seq_heat_call
## Heart
### First step is to download and arrange all files availabe in the Box folder "GC-ME-9475-294065772"
cat FASTQ*/*/*E24HR*fastq.gz >> /Users/subbaprakrit/Box/Prakrit\ Subba\ research/PS_Heat_call_RNA_seq_analysis/Heart/E24HR.fastq.gz
#### Change E24HR from E8HR to E34HR 

### Remove the adapter contamination, polyA read through, and low quality tails

#PBS -N bbduk_heart 
#PBS -l select=1:ncpus=12:mem=62gb:interconnect=fdr,walltime=24:00:00 
#PBS -m abe 
#PBS -j oe

cd /home/psubba/Heart
for sample in E*.fastq; do cat $sample | /home/psubba/miniconda2/pkgs/bbmap-38.18-0/bin/bbduk.sh -Xmx10g in=stdin.fq out=${sample}_trimmed_clean.fastq int=f ref=/home/psubba/miniconda2/pkgs/bbmap-38.18-0/opt/bbmap-38.18/resources/polyA.fa.gz,/home/psubba/miniconda2/pkgs/bbmap-38.18-0/opt/bbmap-38.18/resources/truseq.fa.gz k=13 ktrim=r useshortkmers=t mink=5 qtrim=t trimq=10 minlength=20 2>> output.txt; done
