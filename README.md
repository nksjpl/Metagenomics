# Metagenomic Analysis Pipeline for Illumina and Nanopore Sequences
    1. Data Preprocessing

    Objective: Ensure high-quality input data by removing artifacts, adapters, and low-quality reads.

    Illumina-specific:

    #Tools: 
        FastQC (quality assessment).
        Trimmomatic or Cutadapt (adapter trimming, quality filtering).
    #Steps:
        1. Run FastQC to visualize raw read quality (e.g., per-base quality scores, adapter content).
        2. Trim adapters and low-quality bases (e.g., Q<20), remove reads shorter than 50 bp.
        3. Re-run FastQC post-trimming for QC validation.
    
    #Command Example for bash

    trimmomatic PE -phred33 input_R1.fastq input_R2.fastq output_R1_paired.fq output_R1_unpaired.fq output_R2_paired.fq output_R2_unpaired.fq ILLUMINACLIP:adapters.fa:2:30:10 SLIDINGWINDOW:4:20 MINLEN:50


    Nanopore-specific:

    #Tools: 

        NanoFilt (filtering and trimming).
        Porechop (adapter removal).

	#Steps:

        1. Filter reads by quality (e.g., Q>10) and length (e.g., >500 bp) using NanoFilt.
        2. Remove adapters with Porechop.
        3. Assess quality with NanoStat.

    #Command Example for bash

        porechop -i input.fastq -o trimmed.fastq 
        nanofilt -q 10 -l 500 trimmed.fastq > filtered.fastq


    2. Host/Contaminant Removal (Optional)

	Objective: Remove host or non-target sequences (e.g., human DNA).

	Tools: 

        1. Bowtie2 (Illumina) or Minimap2 (Nanopore) for alignment to a reference host genome.
        2. Samtools for filtering unmapped reads.

	Steps:

        1. Index the host genome (e.g., human GRCh38).
        2. Map reads to the host genome.
        3. Extract unmapped reads (metagenomic fraction).

	#Command Example for bash 
	
    Illumina:
        bowtie2 -x host_index -1 output_R1_paired.fq -2 output_R2_paired.fq | samtools view -b -f 4 -o unmapped.bam
        samtools fastq unmapped.bam > cleaned.fastq
        
	Nanopore:
        minimap2 -ax map-ont host_index filtered.fastq | samtools view -b -f 4 -o unmapped.bam
        samtools fastq unmapped.bam > cleaned.fastq

