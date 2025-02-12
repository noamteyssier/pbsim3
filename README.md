1. About PBSIM

We have developed PBSIM, a simulator for all types of Pacific Biosciences (PacBio) and Oxford Nanopore Technologies (ONT) long reads.

PBSIM can simulate whole genome sequencing (WGS) and transcriptome sequencing (TS) of the PacBio RS II continuous long reads (CLR), PacBio Sequel CLR, PacBio Sequel high-fidelity (HiFi) reads, and ONT reads. PBSIM does not directly simulate HiFi reads, but only simulate generation of CLR by multi-pass sequencing; the output of PBSIM simulation is input into ccs software (https://github.com/PacificBiosciences/ccs), which generates HiFi reads.

In addition, PBSIM can simulate full-length sequencing of user entered templates.

Note: To compress the output files, SAMtools (https://github.com/samtools/samtools) and gzip (https://www.gnu.org/software/gzip/) must be installed in the PBSIM execution environment.


2. Run PBSIM with the sample data

(1) WGS simulation

To run model-based simulation using quality score model:

    pbsim --strategy wgs
          --method qshmm
          --qshmm data/QSHMM-RSII.model
          --depth 20
          --genome sample/sample.fasta

In the example above, simulated reads are randomly sequenced from the reference sequence ("sample/sample.fasta") and errors are introduced into simulated reads. The coverage depth is 20.
  Quality scores are generated by FIC-HMM of real reads.
QSHMM-RSII.model: quality score model constructed from PacBio RS II reads. 
QSHMM-ONT.model and QSHMM-ONT-HQ.model: quality score model constructed from ONT reads. 
  If the reference sequence is a multi-FASTA file, Output files for simulated reads are created for each FASTA. three output files are created for each FASTA.
"sd\_0001.ref": a single-FASTA file which is copied from the reference sequence.
"sd\_0001.fq.gz": a simulated read dataset in the FASTQ format, compressed with gzip.
"sd\_0001.maf.gz": a list of alignments between the reference sequence and simulated reads in the MAF format, compressed with gzip.

Note: OSHMM-ONT-HQ.model is recommended when simulating a read set with an average accuracy of 90% or higher.

To run model-based simulation using error model:

    pbsim --strategy wgs
          --method errhmm
          --errhmm data/ERRHMM-RSII.model
          --depth 20
          --genome sample/sample.fasta

Errors are generated by FIC-HMM of real reads.
ERRHMM-RSII.model: error model constructed from PacBio RS II reads. 
ERRHMM-SEQUEL.model: error model constructed from PacBio Sequel reads. 
ERRHMM-ONT.model and ERRHMM-ONT-HQ.model: error model constructed from ONT reads. 

Note: ERRHMM-ONT-HQ.model is recommended when simulating a read set with an average accuracy of 90% or higher.

Note: All quality codes of simulated reads by error model is "!".


To run sampling-based simulation:

    pbsim --strategy wgs
          --method sample
          --sample sample/sample.fastq
          --depth 20
          --genome sample/sample.fasta

The sampling-based simulation can only be used for the WGS simulation. In the sampling-based simulation, read length and quality score are the same as those of a read sampled randomly from the real read dataset ("sample/sample.fastq").
  If you need various data with different conditions using the same sample, you would be better to use --sample-profile-id option as below.
At the first simulation, a sample profile is stored while simulation. The sample profile consists of two files: "sample-profile" + ID + "fastq", and + "stats".

    pbsim --strategy wgs
          --method sample
          --sample sample/sample.fastq
          --depth 20
          --genome sample/sample.fasta
          --sample-profile-id pf1

At the second simulation, the sample profile is used. No sample fastq is needed this time.

    pbsim --strategy wgs
          --method sample
          --depth 50
          --difference-ratio 20:10:70
          --genome sample/sample.fasta
          --sample-profile-id pf1


To run multi-pass sequencing:

    pbsim --strategy wgs
          --method qshmm
          --qshmm data/QSHMM-RSII.model
          --depth 20
          --genome sample/sample.fasta
          --pass-num 10

If the number of passes (--pass-num) is two or more, multi-pass sequencing is performed. The output files are the following three types.
"sd\_0001.ref": a single-FASTA file which is copied from the reference sequence.
"sd\_0001.bam": a simulated read dataset in the BAM format.
"sd\_0001.maf.gz": a list of alignments between the reference sequence and simulated reads in the MAF format, compressed with gzip.

Note: sampling-based simulation cannot be done with multi-pass sequencing.



(2) TS simulation

To run model-based simulation using quality score model:

    pbsim --strategy trans
          --method qshmm
          --qshmm data/QSHMM-RSII.model
          --transcript sample/sample.transcript

In the example above, simulated reads are randomly sampled from transcript sequences ("sample/sample.fasta"). The number of sequencing for each transcript is specified in "sample.transcript".
The user have to inputs the sequencing templates, that is, the transcript sequences and and their expression profile into PBSIM (see sample/sample.fasta). This information is given to PBSIM as one tab-delimited file, the format is 1-line-1-transcript, and the items have a transcript-ID, and the number of expressions (sense), number of expressions (anti-sense), and nucleotide sequences. The same number of reads as the numbers of expression will be generated.


To run model-based simulation using error model:

    pbsim --strategy trans
          --method errhmm
          --errhmm data/ERRHMM-RSII.model
          --transcript sample/sample.transcript


To run multi-pass sequencing:

    pbsim --strategy trans
          --method qshmm
          --qshmm data/QSHMM-RSII.model
          --transcript sample/sample.transcript
          --pass-num 10


Note: sampling-based simulation cannot be done with TS.



(3) Template full-length sequencing simulation

To run model-based simulation using quality score model:

    pbsim --strategy templ
          --method qshmm
          --qshmm data/QSHMM-RSII.model
          --template sample/sample.template

Simulation of sequencing error is performed using nucleotide sequences input by the user as templates. PBSIM uses model-based simulation to introduce errors into the templates. You can use the error rate options, but not the length options.


To run multi-pass sequencing:

    pbsim --strategy templ
          --method qshmm
          --qshmm data/QSHMM-RSII.model
          --template sample/sample.template
          --pass-num 10


Note: sampling-based simulation cannot be done with template full-length sequencing.



3. Model-based simulation

For each read, the length is randomly drawn from the gamma distribution with given mean and standard deviation.
  The exponential function which is fit to read accuracy distribution is utilized to simulate with given mean. For each read, the accuracy is randomly drawn from the simulated distribution.
  Errors from single molecule sequencing which generates long reads are considered to be stochastical. However, it has been reported that base-calling qualities of PacBio sequencers are not uniform, and low quality regions are sometimes observed in reads. To simulate these low quality regions, the non-uniformity is generated by FIC-HMM trained using real long reads. "data/QSHMM-\*.model" are quality score models for each sequencer of PacBio and ONT. Error models were also built using FIC-HMM in the same manner as building the quality score models. The training data of the error models are the alignments between real reads and their reference genomes. "data/ERRHMM-\*.model" are error models for each sequencer of PacBio and ONT.
  In WGS, simulated reads are randomly sampled from the reference sequence. The percentage of both directions of reads is same. Errors are introduced into the simulated reads as follows:
Quality score model: for each position of the read, all error types (substitution, insertion, and deletion) are introduced according to quality score at that position. All error rates are calculated from quality score and the ratio of error types given by the user. With regards to a deletion, there is no quality score for the deletion itself; thus the quality score of the 5’neighbor is used. We observed that inserted nucleotides are often the same as their following nucleotides. According to this bias, half of the inserted nucleotides are chosen to be the same as their following nucleotides, and the other half are randomly chosen.
Error model: the FIC-HMM determines the error type at each positon. The error ratio is built into the error model.
  By setting minimum and maximum of read length, and range of that chosen from the distribution model can be restricted. Note that mean and standard deviation of the chosen read length are influenced by this restriction. Minimum and maximum of read accuracy are determined by mean of accuracy.



4. Sampling-based simulation

The lengths and quality scores of reads are simulated by randomly sampling them from real reads provided by the user. Subsequently, their nucleotide sequences are simulated by the same way as the model-based simulation. The restriction of read length and accuracy can be set by the options.
Note that the sampling-based simulation cannot be done with multi-pass sequencing and TS.



5. Input files

A reference genome in FASTA format is required for WGS simulation, specified with the --genome option. 

A transcript dataset in our original format is required for TS simulation, specified with the --transcript option. The transcript dataset is a tab-delimited file, its format is 1-line-1-transcript, and the items have transcript-ID, and the number of expressions (sense), number of expressions (anti-sense), nucleotide sequences.

A template dataset in FASTA format is required for template full-length sequencing simulation, specified with the --template option.

A real read dataset in FASTQ format is required for sampling-based simulation, specified with the --sample-fastq option. FASTQ format must be Sanger standard (fastq-sanger).

Input file must be a text file.



6. Output files

If a reference genome is multi-FASTA format, simulated datasets are generated for each reference sequence numbered sequentially.
Three output files are created for each reference sequence.

"sd\_<num>.ref": a single-FASTA file which is copied from the reference sequence.
"sd\_<num>.fq.gz": a simulated read dataset in the FASTQ format, compressed with gzip.
"sd\_<num>.maf.gz": a list of alignments between the reference sequence and the simulated reads in the MAF format, compressed with gzip.

For the multi-pass sequencing, BAM format files are created instead of FASTQ format files.
"sd\_0001.bam": a simulated read dataset in the BAM format.

"sd" is prefix which can be specified with the --prefix option.



7. Quality score model

"data/QSHMM-\*.model" are FIC-HMM for quality scores. We utilized HMM with the latest model selection criteria called factorised information criteria (FIC-HMM) (Hamada et al., 2015). The model were trained for each read accuracy of each sequencer of PacBio and ONT. For read accuracy with insufficient training data, constant quality scores that match the accuracy were used.



8. Error model

"data/ERRHMM-\*.model" are FIC-HMM for errors. Error models were also built using FIC-HMM in the same way as when building the quality score models. The training data of the error models are the alignments between real reads and their reference genomes. The model were trained for each read accuracy of each sequencer of PacBio and ONT. For read accuracy with insufficient training data, the closest accuracy model is transformed and used in PBSIM.



9. Deletion homopolymer bias of ONT reads

ONT reads have a deletion homopolymer bias; the longer the homopolymer length, the higher the deletion rate. The option --hp-del-bias allows the deletion homopolymer bias. The option specifies the deletion rate at 10-mer, where the deletion rate at 1-mer is 1. The bias intensity from 1-mer to 10-mer is proportional to the length of the homopolymer. However, the error model only slightly simulates the bias. In the quality score model, the deletion rate can be changed flexibly;however, in the error model, the error ratio is built into the model, so the change is limited. When simulating homopolymer bias, a quality score model should be used.


10. Runtime and memory

When a coverage depth is 100x and a length of reference genome is about 10M, PBSIM generates simulated dataset in several minutes. The runtime is roughly proportional to the coverage depth and the length of reference genome. PBSIM requires several times as much memory as the size of the reference sequence.
