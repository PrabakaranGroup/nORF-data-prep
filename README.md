# nORF-data-prep
The following is a command-line walkthrough for processing [OpenProt](http://www.openprot.org/) ([Brunet et al., 2018](https://doi.org/10.1093/nar/gky936)) and [sorfs.org](http://www.sorfs.org/) ([Olexiouk et al., 2018](https://doi.org/10.1093/nar/gkx1130)) into a human novel open reading frame (nORF) dataset.

## Prerequisites
1. Clone this repository
```
git clone https://github.com/PrabakaranGroup/nORF-data-prep.git
```
2. R with the following packages installed:
* tidyverse
* Biostrings
* GenomicRanges
* rtracklayer
* GenomicFeatures
* Guitar

## Download Files

#### Step 1: Download OpenProt entries

The code below downloads and names `.tsv`, `.bed`, and `.fasta` files with the following parameters from OpenProt (http://www.openprot.org/p/download):
* Release: 1.3
* Species: Homo Sapiens
* Assembly: GRCg38.p5
* Protein Type: AltProts and Isoforms
* Annotation: Ensembl (GRCh38.83)
* Supporting Evidence: All predicted

```
./ensemblOpenProtDownload.sh
```

#### Step 2: Download sorfs.org entries

The code below downloads files with the following parameters from sorfs.org (http://www.sorfs.org/BioMart):
* Database: Homo Sapiens
* Floss Classification: Good, Extreme
* Main Attributes: Sorf ID, Chromosome, Sorf Start, Sorf End, Strand, Spliced Start Parts, Spliced Stop Parts, Start Codon, Sorf Length, AA sequence, Transcript sequence, Biotype, Annotation, Ensembl Transcript ID

```
wget -O sorfsDownload.txt 'http://biomart.biobix.be/martservice/results?query=<!DOCTYPE Query><Query client="true" processor="TSV" limit="-1" header="1"><Dataset name="BioMart" config="Human"><Filter name="human__classification_104" value="Extreme,Good" filter_list=""/><Attribute name="human__sorf_id_104"/><Attribute name="human__chr_104"/><Attribute name="human__sorf_begin_104"/><Attribute name="human__sorf_end_104"/><Attribute name="human__strand_104"/><Attribute name="human__start_parts_104"/><Attribute name="human__stop_parts_104"/><Attribute name="human__sorf_length_104"/><Attribute name="human__start_codon_104"/><Attribute name="human__aa_seq_104"/><Attribute name="human__biotype_104"/><Attribute name="human__annotation_104"/><Attribute name="human__id_104"/><Attribute name="human__tr_seq_104"/></Dataset></Query>'
```

## Process Files

Use the provided `processDatasets.R` to create custom BED and GTF files. 
Specifically, this script will:
 * A) Extract sorfs.org entries and OpenProt entries labelled as 'AltProt' with mass spec AND/OR ribo-seq evidence. These are the entries that qualify as novel ORFs.
 * B) Remove duplicate entries based on matching genomics coordinates
 * C) Output subsetted `.gtf` and `.bed` files based on input argument: openprot, openprot2pep, sorfs, sorfsMS, all

```
#Combined sorfs + openprot
Rscript processDatasets.R all
```

## Classify Entries

This script classifies the combined OpenProt and sorfs.org entries by their transcript type/relation to enannotated protein coding regions.
```
#Combined sorfs + openprot
Rscript classifynORFsCode.R
```

## Optional Alternatives
#### Refseq Download and processing
OpenProt also generates novel ORFs with refseq annotations (sorfs.org does not). To work with this data instead of default ensembl entries, download and classify these refseq entries with the follwing code:

```
#Download files
./refseqOpenProtDownload.sh
#Process data
Rscript processDatasets.R openprotRefseq
#Classify entries
Rscript classifyRefSeq.R
```
#### Sorfs.org with mass spec evidence
Optionally, the code below downloads and processes a similar file for sorfs.org entries that also have mass spec evidence from their PRIDE ReSpin pipeline.
```
wget -O sorfsDownload_MS.txt 'http://biomart.biobix.be/martservice/results?query=<!DOCTYPE Query><Query client="true" processor="TSV" limit="-1" header="1"><Dataset name="BioMart" config="Human"><Filter name="human__classification_104" value="Extreme,Good" filter_list=""/><Attribute name="human__sorf_id_104"/><Attribute name="human__chr_104"/><Attribute name="human__sorf_begin_104"/><Attribute name="human__sorf_end_104"/><Attribute name="human__strand_104"/><Attribute name="human__start_parts_104"/><Attribute name="human__stop_parts_104"/><Attribute name="human__sorf_length_104"/><Attribute name="human__start_codon_104"/><Attribute name="human__aa_seq_104"/><Attribute name="human__tr_seq_104"/><Attribute name="human__biotype_104"/><Attribute name="human__annotation_104"/><Attribute name="human__id_104"/><Attribute name="ReSpin__file_106"/></Dataset></Query>'
Rscript processDatasets.R sorfsMS
```

## Authors

Written by Matt Neville (Cambridge Department of Genetics), Chaitanya Erady (Cambridge Department of Genetics), Robin Kohze (Cambridge Department of Genetics), and Narendra Meena (Department of Biology, IISER Pune).

## Acknowledgments
Supervisor: Sudhakaran Prabakaran (https://prabakaran-group.org/)
