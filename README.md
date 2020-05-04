# RNA Sequencing Vs Targeted Sequencing
This is a pipeline created by our team in order to compare T cell receptors (TCR) from 20 blood samples (10 individuals, 6 month interval) obtained through TCR targeted sequencing data with RNA sequencing which has been extracted and analysed for TCR-specific reads. 

> Is the RNA sequencing method faithful and informative? 

**Extracting TCRs from RNAseq data**

MiXCR is a universal framework that processes big immunome data from raw sequences to quantitated clonotypes. MiXCR efficiently handles paired- and single-end reads, considers sequence quality, corrects PCR errors and identifies germline hypermutations. The software supports both partial- and full-length profiling and employs all available RNA or DNA information, including sequences upstream of V and downstream of J gene segments.

- [Detailed Documentation](https://mixcr.readthedocs.io/en/master/)

The latest version 3.0.11 can be installed through [conda](https://anaconda.org/imperial-college-research-computing/mixcr)
```
conda install -c imperial-college-research-computing mixcr
```

## Mixcr analysis

### Alignment

The align command aligns raw sequencing reads to reference V, D, J and C genes of T- and B- cell receptors. 
Mixcr supports fasta, fastq, fastq.gz and paired-end fastq and fastq.gz input. 
```
mixcr align -p rna-seq -s hsa -OallowPartialAlignments=true -r $report $fq1 $fq2 $vdjca 
```
In our case we have samples which are paired end reads "$fq1" and "$fq2", and the species we are aligning to is *homo sapiens* "-s hsa". 

"-OallowPartialAlignments=true" option is needed to prevent MiXCR from filtering out partial alignments, that don’t fully cover CDR3 (the default behaviour while processing targeted RepSeq data). MiXCR will try to assemble contigs using those alignments and reconstruct their full CDR3 sequence on the next step.

"$vdjca" is the output file containing the aligned reads & "-r $report" contains the summary of alignment procedure. 

### Assemble Partial

Mixcr allows assemblePartial function which performs an overlap of already aligned reads from the previous step (*.vdjca files) realigns resulting contig, and checks if initial overlap has covered enough part of a non-template N region. Default thresholds in this procedure were optimized to assemble as many contigs as possible while producing zero false overlaps (no false overlaps were detected in all of the benchmarks we have performed).
```
mixcr assemblePartial -r $report1 $vdjca $rescued_vdjca
mixcr assemblePartial -r $report2 $rescued_vdjca $rescued_vdjca_2
```
The latest version (3.0.1) Mixcr is compatible with the function extend which Perform extension of incomplete TCR CDR3s with uniquely determined V and J genes using germline sequences.
```
mixcr extend -r $report3 $rescued_vdjca_2 $rescued_2_extended_vdjca
```

### Assemble clones

The assemble command builds clonotypes from alignments obtained with align. Clonotypes assembly is performed for a chosen assembling feature.
```
mixcr assemble -r $report4 $rescued_2_extended_vdjca $clns
```

### Export clones

Export clones from a binary file (.clns) to a human-readable text file use commands exportClones 
```
mixcr exportClones $clns $output 
```

### Cloud

To run all of 40 of our samples (paired end reads therefore 80 files) we put these commands into a for loop and were run on the microsoft Azura cloud as it required a long time period for mixcr to produce all the desired outputs. Conda, Mixcr and tmux were installed onto the virtual machine made. 

Installing Conda:
```
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
rm Miniconda3-latest-Linux-x86_64.sh
source .bashrc
conda install scikit-learn pandas jupyter ipython
```
Installing mixcr using conda:
```
conda install -c imperial-college-research-computing mixcr 
```
Installing tmux in order to keep the script running in the background even when logged out of the virtual machine:
```
sudo apt-get install tmux
tmux (./paired_mixcr.sh) # To run the job
tmux attach -t 0 # To see the screen output 
```
