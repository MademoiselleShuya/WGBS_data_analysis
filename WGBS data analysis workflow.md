Here, we use Cabsau（Grape Data）as an example to perform WGBS data analysis.

# 1. Quality Control

Being able to do the quantification reliably depends on quality control before alignment, the alignment methods and post alignment quality control. Here we perform some simple quality control checks to ensure that the raw data looks good.  
```
/home/u1995087/miniconda3/envs/wgbs_tools/bin/fastqc cabsau_R1.fastq -o /mnt/data/shuya/WGBS_modern/1_quality_control/
```
==-o/--output_dir <dir>==  Write all output files into this directory. 

The input file, cabsau_R1.fastq 

Two output files, cabsau_R1_fastqc.html and cabsau_R1_fastqc.zip

If there are multiple samples, we can use multiqc to summarize our data. Multiqc will give us two output files multiqc_data and multiqc_report.html

```
/home/u1995087/miniconda3/envs/wgbs_tools/bin/multiqc .
```

# 2. Removing adapters
In our project, adapters were sequenced and if not properly removed, they will either lower the alignment rates or cause false C-T conversions. In this stage, Trim_galore will be used to remove low-quality bases on sequence ends and removing adapters to minimize issues with false C-T conversions and to increase alignment rates. 

```
/home/u1995087/miniconda3/envs/wgbs_tools/bin/trim_galore  --length 75 --paired cabsau_R1.fastq cabsau_R2.fastq -o /mnt/data/shuya/WGBS_modern/2_cutadapt/
```
==--paired== This   option   performs   length   trimming   of   quality/adapter/RRBS trimmed reads for paired-end files.

==--length <INT>== Discard reads that became shorter than length INT because of eitherquality  or  adapter  trimming. A  value  of  '0'  effectively  disablesthis behaviour. Default: 20 bp.

The input files are cabsau_R1.fastq, and cabsau_R2.fastq.

There are two output files, cabsau_R1.fastq_trimming_report.txt, and cabsau_R1_val_1.fq.
One is a rimming report which includes all trimming information, and another one is trimmed fastqc file.

# 3. Quality Control

Use fastqc to check the trimmed data

```
 /home/u1574607/miniconda3/bin/fastqc cabsau_R1_val_1.fq cabsau_R2_val_2.fq -o /mnt/data/shuya/WGBS_modern/3_qc_trimed/
```

Use multiqc to summarize our data

```
/home/u1995087/miniconda3/envs/wgbs_tools/bin/multiqc .
```

# 4. BS-seq Alignment

We perform Bismark to align reads to the reference genome. Before this, a bismark_genome_preparation script needs to be run to prepare the genome of interest for bisulfite alignments.

```
/home/u1995087/miniconda3/bin/Bismark-0.22.3/bismark_genome_preparation /mnt/data/shuya/WGBS_modern/ref/
```
==<path_to_genome_folder>== The path to the folder containing the genome to be bisulfite converted. Bismark Genome Preparation expects one or more FastA files in the folder (valid file extensions: .fa or .fasta). 

```
/home/u1995087/miniconda3/bin/Bismark-0.22.3/bismark /mnt/data/shuya/WGBS_modern/ref/ -1 cabsau_R1_val_1.fq -2 cabsau_R2_val_2.fq --multicore 8 -o /mnt/data/shuya/WGBS_modern/4_aln/
```

==<genome_folder>== The full path to the folder containing the unmodified reference genome as well as the subfolders created by the bismark_genome_preparation script (/Bisulfite_Genome/CT_conversion/ and Bisulfite_Genome/GA_conversion/). 

==-1 <mates1>== Comma-separated list of files containing the #1 mates 

==-2 <mates2>== Comma-separated list of files containing the #2 mates 

==--multicore <int>==  Sets the number of parallel instances of Bismark to be run concurrently

==-o/--output_dir <dir>== Write all output files into this directory. 

The input files are cabsau_R1_val_1.fq, and cabsau_R2_val_2.fq.

Bismark provides BAM files, as well as additional methylation call related metrics and files.


# 5. Removing PCR bias duplications
The last post-alignment quality procedure addresses PCR bias. A simple way could be to remove reads that align to the exact same genomic position on the same strand. This de-duplication can be performed using the samtools rmdup command or Bismark tools. Here, we use Bismark.

```
/home/u1995087/miniconda3/envs/wgbs_tools/bin/deduplicate_bismark --bam cabsau_R1_val_1_bismark_bt2_pe.bam
```
The input file is cabsau_R1_val_1_bismark_bt2_pe.bam.

cabsau_R1_val_1_bismark_bt2_pe.deduplicated.bam is the output file.


# 6. Methylation calling

We run a bismark_methylation_extractor script which operates on Bismark result files and extracts the methylation call for every single C analysed.

```
/home/u1995087/miniconda3/envs/wgbs_tools/bin/bismark_methylation_extractor --multicore 8 -p --no_overlap --report  --cytosine_report --bedGraph --scaffolds --genome_folder /mnt/data/shuya/WGBS_modern/ref/ cabsau_R1_val_1_bismark_bt2_pe.deduplicated.bam -o /mnt/data/shuya/WGBS_modern/5_methylation_calling/

```

==-p/--paired-end==  Input file(s) are Bismark result file(s) generated from paired-end read data. Specifying either --paired-end or --single-end is mandatory.

==--no_overlap==  For paired-end reads it is theoretically possible that Read 1 and Read 2 overlap. This option avoids scoring overlapping methylation calls twice.

==--report == Prints out a short methylation summary and the parameters used to run this script. Default: ON.

==--bedGraph==  After finishing the methylation extraction, the methylation output is written into a sorted bedGraph file that reports the position of a given cytosine and its methylation state (in %, seem details below) using 0-based genomic start and 1-based end coordinates. 

==--scaffolds/--gazillion==  Users working with unfinished genomes sporting tens or even hundreds of thousands of scaffolds/contigs/chromosomes frequently encountered errors with pre-sorting reads to individual chromosome files. These errors were caused by the operating system's limit of the number of filehandle that can be written to at any one time (typically 1024; to find out this limit on Linux, type: ulimit -a).

==--cytosine_report== After the conversion to bedGraph has completed, the option --cytosine_report produces a genome-wide methylation report for all cytosines in the genome.

==--genome_folder <path>==  Enter the genome folder you wish to use to extract sequences from (full path only). Accepted formats are FastA files ending with '.fa' or '.fasta'. Specifying a genome folder path is mandatory.

The input file is cabsau_R1_val_1_bismark_bt2_pe.deduplicated.bam.

This process will produce several files: cabsau_R1_val_1_bismark_bt2_pe.deduplicated.bedGraph
cabsau_R1_val_1_bismark_bt2_pe.deduplicated.bismark.cov
cabsau_R1_val_1_bismark_bt2_pe.deduplicated.CpG_report.txt
cabsau_R1_val_1_bismark_bt2_pe.deduplicated.M-bias_R1.png
cabsau_R1_val_1_bismark_bt2_pe.deduplicated.M-bias_R2.png
cabsau_R1_val_1_bismark_bt2_pe.deduplicated.M-bias.txt
cabsau_R1_val_1_bismark_bt2_pe.deduplicated_splitting_report.txt

# 7. Summary

Generate summary graphical HTML report from several Bismark text report files reports
```
/home/u1995087/miniconda3/envs/wgbs_tools/bin/bismark2summary chardonnay_R1_val_1_bismark_bt2_pe.deduplicated.bam chasselas_R1_val_1_bismark_bt2_pe.deduplicated.bam gamay_R1_val_1_bismark_bt2_pe.deduplicated.bam muscat.e_R1_val_1_bismark_bt2_pe.deduplicated.bam muscat.a_R1_val_1_bismark_bt2_pe.deduplicated.bam pinot.n.aus_R1_val_1_bismark_bt2_pe.deduplicated.bam pinot.n_R1_val_1_bismark_bt2_pe.deduplicated.bam riparia_R1_val_1_bismark_bt2_pe.deduplicated.bam
```

Use multiqc to summarize our data

```
/home/u1995087/miniconda3/envs/wgbs_tools/bin/multiqc .
```
