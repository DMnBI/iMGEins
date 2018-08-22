iMGEins
=======

This is an explanation of how to use iMGEins.

Pre-installed programs for iMGEins
----------------------------------

iMGEins needs the following programs and the MGE sequence database. You should first confirm that the following programs are correctly installed in your computer.

- SAMTOOLS version 0.1.18 or higher
- BLAST version 2.2.29+ or higher
- Bowtie2 version 2.1.0 or higher
- Java version 1.7 or higher
- MGE database (.fasta or .fa)

In order to check the availability of SAMTOOLS, BLAST and Bowtie2, you can type “samtools -version”, “blastn –version” and “bowtie2-align --version”, respectively. 

Quick start guide
-----------------
In the folder "sample_data" there are simulated data to run iMGEins.
#### sample_insertion_pos_seq.txt
In this text, there are inserted sequences and its position of chromosome 11, hg19.

#### iMGEins_sample.sam
This sam file is generated from a modified human chromosome 11 that has insertions.

#### sample_MGE_DB.fasta*
This file is a database for sample data. It includes inserted MGEs except random sequence insertions.

#### default_param_for_sample.txt
This parameter file is set for sample data.

Now you are ready to run iMGEins. The following is the basic iMGEins command line for sample data running.
	
	java -jar /data/jwbae/iMGEins_ReleaseVer1.1.0.jar -i ./sample_data/iMGEins_sample.sam -o /your/own/working/dir/ -p ./sample_data/default_param_for_sample.txt



Running iMGEins using your own data
-----------------------------------

### Step 1: Read mapping (optional)

If you have sequencing reads in fastq format but do not have the mapping results in SAM format, you need to map the reads to the reference genome. We recommend Bowtie2 with local alignment mode. 

If you do not have bowtie index files for your reference genome, you should build index file prior to read mapping.

Usage: 
>bowtie2-build your_genome.fa your_genome

If you have bowtie index files, you can directly map reads on the reference genome.

Usage: 
>bowtie2-align --sensitive-local –x /path/to/ bowtie2/index/genome -1 pair_read1.fq   -2 pair_read2.fq –S ouput_file_name.sam

For detailed instructions, please consult with the bowtie manual (http://bowtiebio.sourceforge.net/bowtie2/manual.shtml)

### Step 2: Identify individual MGEs 
This step is for predicting breakpoints and identifying MGEs. iMGEins needs a configuration file name as a parameter to set other required parameters.

	Usage:
	>java -jar -Xmx??g iMGEins_ReleaseVerX.X.X.jar -i [input_sam_file] -o [output_foler] -p [configuration_file]


Configuration file:

	# minimum length of the soft-clipped region
	clipping_size = 5

	# threshold of average base quality of soft clipped region
	baseQthreshold = 60

	# minimum number of soft-clipped reads required
	soft_clip_map_sum = 3

	# minimum number of soft-clipped reads required at each direction of the breakpoint
	support_soft_clip_map = 2

	# coverage of input data
	coverage = 30

	# profile threshold
	profile_cutoff = 0.90

	# overlap reads percentage of the breakpoint
	overlapFilterRatio = 0.1

	# flag for identifying MGEs using BLAST search
	# 1 for searching; 0 for not searching
	blast_flag = 1

	# maximum length of upstream/downstream regions for finding singly-mapped reads
	sm_range = 500

	# path to the MGE database file for BLAST search
	#MGE_DB_path = /path/to/fasta/RepBase.fasta

	# similarity threshold to filt BLAST results
	similarity_cutoff = 90.0

	# alighment length threshold to filt BLAST results
	align_length_cutoff = 70

	# flag for assembling unmapped reads using SOAP denvo2
	# 1 for searching; 0 for not searching
	assembly_flag = 1

	#average insert length for SOAPdenovo2 config file
	avg_ins= 400

	#maximum read length for SOAPdenovo2 config file
	max_rd_len= 101

	#contig length cutoff
	contig_len_cutoff = 500

	# minimum number of mapped reads to each contig
	contig_map_read_cutoff=10

	#number of threads
	numOfThread = 32


Output file
------------

### Break_points_with_assemble.gff
Provide summary of breakpoints including identified MGEs and assembled contig sequences in GFF format. Column 9 includes the following information, which is separated by a semicolon (;).

- DStream-support: Number of soft-clipped reads in the downstream of the breakpoint
- UStream-support: Number of soft-clipped reads in the upstream of the breakpoint
- DStreamPolyA: True if Poly-A tail soft-clipped sequence presents in the downstream of the breakpoint
- DStreamPolyT: True if Poly-T tail soft-clipped sequence presents in the downstream of the breakpoint
- UStreamPolyA: True if Poly-A tail soft-clipped sequence presents in the upstream of the breakpoint
- UStreamPolyT: True if Poly-T tail soft-clipped sequence presents in the upstream of the breakpoint
- softReadMGEID: MGE family name assigned using soft-clipped reads
- numOfHitSoftReads: Number of soft-clipped reads that support the MGE family assigned
- numOfUnmapReads: Number of singly unmapped reads (SU, US, MU and UM) supporting the break point
- MGEID: MGE family name assigned using singly-unmapped reads
- numOfHitReads: Number of unmapped reads that support the MGE family assigned
- avgSimilarity: Average similarity value of BLAST hit to MGEID
- avgAlnReadLen: Average aligned read length of BLAST hit to MGEID
- numOfMissedReads: Number of reads that are not related with the MGE family assigned
- HitRatio: Ratio of reads that support the MGE family assigned
- NovelMGEContigID: Assembled contig ID
- ContigToMGEID: MGE family name assigned using assembled contig
- MGEHitPos: Coordinate of MGE hit to contig by BLAST
- NumOfMappedReadToContig: Number of singly-unmapped reads that aligned to the contig
- trimmedContigSeq: Assembled and trimmed contig sequence using dynamic programming
