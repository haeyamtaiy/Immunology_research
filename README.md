# Immunology_research
Mixcr pipeline

Download and install mixcr version 3 from conda 
conda install -c imperial-college-research-computing mixcr

script
```
#!/bin/bash
set -e
# Purpose:  mixcr Alignment, Assemble clones and Export 
# Author:   Haeyam Taiy
# Date:     2019-12-18
# Version:  0.1

# Create some output directories
mkdir -p vdjca clns output report

# Loop over the sample files

for fq1 in *_R1_001.fastq.gz
do 
	base=$(basename ${fq1} _R1_001.fastq.gz)
	

	# Create file names
	fq1=${base}_R1_001.fastq.gz
	fq2=${base}_R2_001.fastq.gz
	
	vdjca=vdjca/${base}.vdjca
	report=report/${base}_vdjca_report.txt
	rescued_vdjca=vdjca/${base}_rescued_1.vdjca
	rescued_vdjca_2=vdjca/${base}_rescued_2.vdjca
	rescued_2_extended_vdjca=vdjca/${base}_rescued_2_extended.vdjca
	clns=clns/${base}.clns
	report1=report/${base}_rescued_report.txt
	report2=report/${base}_rescued2_report.txt
	report3=report/$base}_rescued_extended_report.txt
	report4=report/{base}_clns_report.txt
	output=output/${base}_output.txt

	# Analysis
	mixcr align -p rna-seq -s hsa -OallowPartialAlignments=true -r $report $fq1 $fq2 $vdjca 
	mixcr assemblePartial -r $report1 $vdjca $rescued_vdjca
	mixcr assemblePartial -r $report2 $rescued_vdjca $rescued_vdjca_2
	mixcr extend -r $report3 $rescued_vdjca_2 $rescued_2_extended_vdjca
	mixcr assemble -r $report4 $rescued_2_extended_vdjca $clns
	mixcr exportClones $clns $output 
done
```

