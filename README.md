# Sequencing coverage scripts

## Supplimental scripts to the [Phyluce](https://github.com/faircloth-lab/phyluce) package for calculating sequencing coverage of loci and [Trinity](https://github.com/trinityrnaseq/trinityrnaseq/wiki) contigs using [Bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml)



Scripts for use in calculating sequencing coverage for Trinity contigs for multiple samples, and UCE loci (or other fasta formatted files). 

1. Phyluce must be installed, see info pages on the package. 
2. Bowtie2 must be installed (homebrew will work here: 'brew install bowtie2')
3. GNU grep must be installed ('brew install grep' will install ggrep, which is currently hard coded into the contig_coverage.py script...change the code for your flavor of GNU grep). 
3. Both scripts require a configuration file containing names of samples to process. No header is required. ([example](https://github.com/BioNrd/Sequencing-Coverage-Scripts/blob/master/config_file_example.txt)) 
4. assembly_coverage.py requires: 
 - Directory from illumiprocessor script step (typically called something like 'uce-clean')
 - Directory of Trinity assemblies.
 	- This directory contains subdirectories named after the sample names, where each directory contains a file called 'contigs.fasta' containing the assemblies. (e.g., ./Trinity_Assemblies/[TAXON1]/contigs.fasta)
 	  - NOTE: After the script is run, all files in the directory EXCEPT contigs.fasta will be removed. This prevents the SAM and BAM files taking up too much disk space while procesessing many samples at a time. 
5. contig_coverage.py requires:
 - Directory from illumiprocessor script step (typically called something like 'uce-clean')
 - Fasta file concatenated from loci files generated in subsequent Phyluce script steps (e.g., phyluce_assembly_get_fastas_from_match_counts; phyluce_align_seqcap_align, see notes below)
     - NOTE: If using post alignment files, the gap and other 'alignment' characters must be stripped from the fasta file.


#### Converting alignments to concatenated fasta format for script input. For input alignments, slected alignments at a point that makes the most sense to you, perhaps prior to running GBLOCKS, but that is your choice.

>Convert from nexus to fasta:
>```
>phyluce_align_convert_one_align_to_another --alignments /path/to/incomplete_matrix/mafft-nexus/ --output /path/to/mafft-fasta --input-format nexus --output-format fasta
>```
>Combine the fasta files from within the directory of fasta files into a single file:
>```bash
>cat * > ../combined.fasta
>```
>Remove linebreaks in fasta file:
>```bash
>awk '/^>/ {printf("\n%s\n",$0);next; } { printf("%s",$0);}  END {printf("\n");}' < combined.fasta > combined_singleline.fasta
>```
>Remove gaps and inserted ? from files (need files in unaligned format):
>```bash
>sed 's/[?-]//g' combined_singleline.fasta > cleaned_concat.fasta
>```



### Example calls: 
    python assembly_coverage.py --rawreads ../path_to/uce-clean/ --input ../path_to/trinity_assembly/ --output <output_directory> --config <config_file> --threads <#CPUs>

    python contig_coverage.py --rawreads ../path_to/uce-clean/ --input <cleaned_concat.fasta> --output <output_directory> --config <config_file> --threads <#CPUs>

### Output: 
Three files are generated for each sample:

1. [SAMPLENAME]-smds.coverage.per.locus
 - Contains raw histogram output from [bedtools genomecov](http://bedtools.readthedocs.io/en/latest/content/tools/genomecov.html)
2. [SAMPLENAME]-smds.coverage.per.locus.summary
 - Contains per locus average coverage
3. [SAMPLENAME]-smds.per.base.coverage
 - Contains a per base coverage, calculated using the -d flag in [bedtools genomecov](http://bedtools.readthedocs.io/en/latest/content/tools/genomecov.html)

### Summary Scripts
Contains R scripts for parsing output, and summarizing things like average coverage per locus, etc. Code is heavily commented to show what it is doing. 




**Scripts rely heavily on inspiration and code from [CONCOCT](https://github.com/BinPro/CONCOCT/blob/master/scripts/map-bowtie2-markduplicates.sh) and [Phyluce](https://github.com/faircloth-lab/phyluce) used under open source licenses:  
[License A](https://github.com/BinPro/CONCOCT/blob/master/LICENSE.txt),
[License B](https://github.com/faircloth-lab/phyluce/blob/master/LICENSE.txt)**
