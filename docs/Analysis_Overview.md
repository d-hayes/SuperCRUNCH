![SuperCrunch Logo](https://github.com/dportik/SuperCRUNCH/blob/master/docs/SuperCRUNCH_Logo.png)

---------------

## Analysis Overview and Detailed Instructions

Complete instructions for performing each step of **SuperCRUNCH** are provided here. The scripts are listed in the approximate order in which they should be used, and split among larger topics of interest. Helpful information can also be displayed for all scripts on the command line by running them using the -h flag. 

### Starting Materials:

+ [Overview](#GSM)
+ [Obtaining Sequence Data](#OSD)
    + [Remove_Duplicate_Accessions.py](#RDA)
+ [Obtaining Taxon Names Lists](#OTNL)
    + [Getting Taxa From Fasta Files](#GTFFF)
+ [Obtaining Loci Search Terms](#OLST)
    + [Searching for UCE loci](#SFUL)

### Taxon Filtering and Locus Extraction:

+ [Overview](#TFLE)
+ [Taxa_Assessment.py](#TA)
+ [Rename_Merge.py](#RM)
+ [Parse_Loci.py](#PL)

### Orthology Filtering

+ [Overview](#OF)
+ [Cluster_Blast_Extract.py](#CBE)
+ [Reference_Blast_Extract.py](#RBE)
+ [Contamination_Filter.py](#CF)

### Sequence Quality Filtering and Selection

+ [Overview](#SQFS)
+ [Filter_Seqs_and_Species.py](#FSS)
+ [Make_Acc_Table.py](#MAT)
+ [Infer_Supermatrix_Combinations.py](#ISC)

### Sequence Alignment

+ [Overview](#SA)
+ [Adjust_Direction.py](#AD)
+ [Coding_Translation_Tests.py](#CTT)
+ [Align.py](#A)

### Post-Alignment Tasks

+ [Overview](#FFT)
+ [Relabel_Fasta.py](#RF)
+ [Trim_Alignments.py](#TAS)
+ [Fasta_Convert.py](#FC)
+ [Concatenation.py](#C)

A typical **SuperCRUNCH** run includes executing a majority of these steps. However, this workflow is extremely flexible and can be tailored to achieve a variety of goals.   

---------------

---------------

## **Starting Material** <a name="GSM"></a>

To run **SuperCRUNCH**, you will need to provide a fasta file of downloaded nucleotide sequence records, a file containing a list of taxa, and a file containing a list of loci and associated search terms. Information on how to create these files and recommendations are provided below.

---------------

### Obtaining Sequence Data <a name="OSD"></a>

The starting molecular data for **SuperCRUNCH** consists of a fasta file of downloaded nucleotide sequence records from the [NCBI nucleotide database (GenBank)](https://www.ncbi.nlm.nih.gov/nucleotide/). The simplest way to obtain these data is to search for a taxonomic term on the nucleotide database and download all the available records in fasta format. However, for larger taxonomic groups (ex. all birds or frogs) this may not be possible as there will be too much data to download directly. In this case, it is better to split searches using the taxonomy of the group (order, clade, family, etc), download each record set in fasta format, and then combine the resulting fasta files. The [NCBI taxonomy browser](https://www.ncbi.nlm.nih.gov/Taxonomy/Browser/wwwtax.cgi) is particularly useful for identifying the relevant taxonomic search terms. Advanced users may choose to use additional search terms to filter the results, which could help reduce the resulting file size, but this is not necessary. The downloaded fasta files may be quite large in size (several GB) and it is unlikely you can use a text editor to open and edit them, so combining files requires a different approach. The fasta files will all have the extension '.fasta', and they can be quickly combined using Unix. If the fasta files are all in the same working directory, you can use the following command to accomplish this:

`cat *.fasta > Combined.fasta`

The starting fasta file can be in the interleaved format or sequential format, as demonstrated below: 

**Interleaved format:**
```
>KP820543.1 Callisaurus draconoides voucher MVZ265543 aryl hydrocarbon receptor (anr) gene, partial cds
TAAAATTTCCTTTGAAAGGAACCTTTTTGTGGACACCAGGGATGAATTGGGTAATGTAATGGCGAGCGAT
TGGCAGGAAAATATTTTGCCAGTGAGGAATAACAGCATTCTCAAACAAGAGCAGACAGAGTGCCCCCAGG
AAAATAATTTAATGCTTCCTGAAGACAGCATGGGCATTTTTCAGGATAACAAAAATAATGAACTGTACAA

>KP820544.1 Urosaurus ornatus voucher UWBM7587 aryl hydrocarbon receptor (anr) gene, partial cds
TAAAATCTCCTTTGAAAGGAACCTTTTTGTGGACACCAGGGATGAATTAGGTAATGTAATGGCCAGCGAT
TGGCAGGAAAATATTTTGCCAGTGAGGAATAACAGCATCCTCAAACAAGAACAGACTGAGTGCCCCCAGG
TAAAATTTCCTTTGAAAGGAACCTTTTTGTGGACACCAGGGATGAATTGGGTAATGTAATGGCGAGCGAT
AAACTAATTTAATGCTTCCTGAAGACAGTATGGGTATTTTTCAGGATAACAAAAATAATGAACTGTACAA
```

**Sequential format:**
```
>KP820543.1 Callisaurus draconoides voucher MVZ265543 aryl hydrocarbon receptor (anr) gene, partial cds
TAAAATTTCCTTTGAAAGGAACCTTTTTGTGGACACCAGGGATGAATTGGGTAATGTAATGGCGAGCGATTGGCAGGAAAATATTTTGCCAGTGAGGAATAACAGCATTCTCAAACAAGAGCAGACAGAGTGCCCCCAGGAAAATAATTTAATGCTTCCTGAAGACAGCATGGGCATTTTTCAGGATAACAAAAATAATGAACTGTACAA

>KP820544.1 Urosaurus ornatus voucher UWBM7587 aryl hydrocarbon receptor (anr) gene, partial cds
TAAAATCTCCTTTGAAAGGAACCTTTTTGTGGACACCAGGGATGAATTAGGTAATGTAATGGCCAGCGATTGGCAGGAAAATATTTTGCCAGTGAGGAATAACAGCATCCTCAAACAAGAACAGACTGAGTGCCCCCAGGAAACTAATTTAATGCTTCCTGAAGACAGTATGGGTATTTTTCAGGATAACAAAAATAATGAACTGTACAA
```

The record labels must have a `>` followed by a unique accession number, followed by a sequence description. If curated properly, the description line will contain a taxon name, locus information, and possibly several other identifiers, all separated by whitespace. There should not be any vertical bars (|) separating information contained in the sequence label. **SuperCRUNCH** cannot properly parse this format.

The following vertical bar fasta format is ***NOT*** supported by **SuperCRUNCH**:

```
>gi|2765657|emb|Z78532.1|CCZ78532 C.californicum 5.8S rRNA gene and ITS1 and ITS2 DNA
CGTAACAAGGTTTCCGTAGGTGAACCTGCGGAAGGATCATTGTTGAGACAACAGAATATATGATCGAGTG
AATCTGGAGGACCTGTGGTAACTCAGCTCGTCGTGGCACTGCTTTTGTCGTGACCCTGCTTTGTTGTTGG

>gi|2765656|emb|Z78531.1|CFZ78531 C.fasciculatum 5.8S rRNA gene and ITS1 and ITS2 DNA
CGTAACAAGGTTTCCGTAGGTGAACCTGCGGAAGGATCATTGTTGAGACAGCAGAACATACGATCGAGTG
AATCCGGAGGACCCGTGGTTACACGGCTCACCGTGGCTTTGCTCTCGTGGTGAACCCGGTTTGCGACCGG
```

If your starting sequence set is in this format, you will need to convert it to the simpler whitespace fasta format before attempting to run **SuperCRUNCH**.


---------------

### Remove_Duplicate_Accessions.py <a name="RDA"></a>

Sometimes combining several fasta files can produce duplicate entries, in other words records with identical accession numbers. This will cause problems with the way **SuperCRUNCH** loads fasta files, as there can be no duplicate accession numbers. You will know this is the case if you try to run a script and it produces the following error:

`ValueError: Duplicate key 'AF443291.2'`

In this instance, there are two records each containing the accession number AF443291.2. In order to use the offending fasta file with **SuperCRUNCH**, you'll need to clean the file to remove any duplicate accession numbers. This script can be used for this purpose.


#### Basic Usage:

```
python Remove_Duplicate_Accessions.py -i <fasta file> -o <output directory>
```

#### Argument Explanations:

##### `-i <full-path-to-file>` 
 
> **Required**:  The full path to a fasta file with GenBank sequence data to filter.

##### `-o <path-to-directory>` 

> **Required**: The full path to an existing directory to write  output fasta file.

#### Example Use:

```
python Remove_Duplicate_Accessions.py -i /bin/starting_material/Iguania_GenBank.fasta -o /bin/starting_material/cleaned/
```
> Above command will remove duplicate sequence records in `Iguania_GenBank.fasta` and the output is written to the specified directory `/bin/starting_material/cleaned/`.

By running this script, all duplicate entries will be removed and a new fasta file will be written to the output directory called *Cleaned.fasta*. This fasta file should be used to run **SuperCRUNCH**.

---------------

### Obtaining Taxon Names Lists <a name="OTNL"></a>

**SuperCRUNCH** requires a list of taxon names that is used to filter sequences. Lists of taxon names can be obtained through general databases, such as the NCBI Taxonomy Browser. In many cases there are specific databases dedicated to major groups, for example the [Reptile Database](http://www.reptile-database.org/), [AmphibiaWeb](https://amphibiaweb.org/), and [Amphibian Species of the World](http://research.amnh.org/vz/herpetology/amphibia/), which usually contain up-to-date taxonomies in a downloadable format. 

The taxon names list required is a simple text file which contains one taxon name per line. The file can contain a mix of species (binomial) and subspecies (trinomial) names, and components of each taxon name should be separated by a space (rather than undescore). Sometimes using Excel or other applications to generate the taxon names text file will include hidden characters not compatible with **SuperCRUNCH**, so make sure to open the file in a text editor and ensure the format includes Unix line breaks (line breaks marked by `\n`, rather than `\r\n`) and UTF-8 encoding. Below are some example contents from suitable taxon name lists.

Partial contents of a list containing species (binomial) names:

```
Lygodactylus regulus
Lygodactylus rex
Lygodactylus roavolana
Lygodactylus scheffleri
Lygodactylus scorteccii
Lygodactylus somalicus
```

Partial contents of a list containing species (binomial) and subspecies (trinomial) names:

```
Leycesteria crocothyrsos
Leycesteria formosa
Linnaea borealis americana
Linnaea borealis borealis
Linnaea borealis longiflora
```

**SuperCRUNCH** offers the option to exclude subspecies from searches, so a taxon list containing a mix of species and subspecies does not need to be pruned by hand if subspecies are not desired in the analysis. The effects of this option with different types of taxon lists is explained in usage of the `Taxa_Assessment.py` script.

---------------

### Getting Taxa From Fasta Files <a name="GTFFF"></a>

In some cases it may be desirable to obtain a list of taxon names directly from a fasta file of sequence records. This option is available using the `Fasta_Get_Taxa.py` module, which is described below. Note that this is inevitably an imperfect process, as messy records can produce spurious names. Although this is a useful shortcut, the resulting list of names should always be inspected and edited before its use in any other steps.

### Fasta_Get_Taxa.py

The goal of this script is to search through all fasta files in a directory to construct 'species' and 'subspecies' label sets directly from the description lines. Two output files are written, which are lists of all unique 'species' and 'subspecies' labels found. 
The names are created by combining the second and third elements on the description line (for binomial names) and by combining the second, third and fourth elements on the description line (for trinomial names). There are several filters in place to try to prevent junk names from being produced, but this is inherently imperfect. Although the filters should work relatively well for constructing binomial names, the trinomial names are a much more difficult problem, and the subspecies list will likely contain some spurious names. The script makes no attempt to address synonomy, and if multiple names are used for the same taxon then they will all be recovered and written to the list.

**The resulting taxon list files should be carefully inspected before using them for any other purpose.** After editing, the lists could be combined to create a taxon list of species and subspecies names. 

For smaller data sets, `Fasta_Get_Taxa.py` can help to generate a taxon list quickly and easily. `Fasta_Get_Taxa.py` was intended to be used for population level data sets, which are unlikely to have a large number of taxa. Smaller size data sets allow for careful inspection for spurious names and synonomies, whereas these task would become arduous at larger taxonomic scales. 


#### Basic Usage:

```
python Fasta_Get_Taxa.py -i <fasta file> -t <taxon file> -o <output directory>
```

#### Argument Explanations:

##### `-i <path-to-directory>`

> **Required**: The full path to a directory with fasta file(s) of GenBank sequence data. Fasta files in the directory must have extensions '.fasta' or '.fa' to be read.

##### `-o <path-to-directory>`

> **Required**: The full path to an existing directory to write output files.

#### Example Use:

```
python Taxa_Assessment.py -i bin/FastaSet/ -o bin/FastaSet/Output/
```

Two output files are created in the specified output directory, including:

+ `Species_Names.txt`: List of unique binomial names constructed from record descriptions. If records are labeled correctly this should correspond to the genus and species. This file should be inspected.
+ `Subspecies_Names.txt`: List of unique trinomial names constructed from record descriptions. If records actually contain subspecies labels they will be captured in this list, however if the records only contain a binomial name then spurious names may be produced. This file should be VERY carefully inspected.


---------------

### Obtaining Loci and Search Terms <a name="OLST"></a>

**SuperCRUNCH** requires a list of loci and associated search terms to initially identify the content of sequence records. For each locus included in the list, **SuperCRUNCH** will search for associated abbreviated names and longer labels in the sequence record. The choice of loci to include will inherently be group-specific, and surveys of phylogenetic and phylogeographic papers may help to identify an appropriate marker set. There is no limit to the number of loci, and **SuperCRUNCH** can also be used to search for large genomic data sets available on the NCBI nucleotide database, such as those obtained through sequence capture experiments (UCEs, anchored enrichment, etc.). For detailed instructions on searching for UCEs, see the next section.

The format of the locus text file involves three tab-delimited columns. The first column contains the locus name that will be used to label output files. It must not contain any spaces or special characters (including underscores and hyphens), and should be kept short and simple. The second column contains the known abbreviation(s) for the gene or marker. Abbreviations should not include any spaces, but can contain numbers and other characters (like dashes). The second column can contain multiple abbreviations, which should be separated by a semi-colon with no extra spaces between abbreviations. The third column contains a longer label of the gene or marker, such as its full name or description. This third column can also contain multiple search entries, which also should be separated by a semi-colon with no extra spaces between label entries. The abbreviations and labels are not case-specific, as they are converted to uppercase during actual searches along with the description lines of the sequences. Although the locus text file can be created using Excel or other applications, it must be a tab-delimited text file. Similar to the taxon names file, make sure to open the file in a text editor and ensure the format includes Unix line breaks (line breaks marked by `\n`, rather than `\r\n`) and UTF-8 encoding, otherwise extra characters may interfere with parsing the file correctly with **SuperCRUNCH**.

The success of finding loci depends on defining appropriate locus abbreviations and labels. Examples of how searches operate can be found below, which should help guide how to select good search terms. For any locus, it is a good idea to search for the locus on  on GenBank and examine several records to identify the common ways it is labeled.

Here is an example of the formatting for a locus file containing three genes to search for:

```
CMOS	CMOS;C-MOS	oocyte maturation factor
EXPH5	EXPH5	exophilin;exophilin 5;exophilin-5;exophilin protein 5
PTPN	PTPN;PTPN12	protein tyrosine phosphatase;tyrosine phosphatase non-receptor type 12
```

In this example:
+ CMOS contains two abbreviations and one label search term. 
+ EXPH5 contains one abbreviation and four label search terms.
+ PTPN contains two abbreviations and two label search terms. 

The above example is a subset of loci from a locus search terms file I used for squamate reptiles. The complete locus search terms file (`Locus-Search-Terms_Squamate_Markers.txt`) is provided in the example data folder [here](https://github.com/dportik/SuperCRUNCH/tree/master/data).

**How does the actual locus searching work?**

+ For locus abbreviations, the sequence record label is split by spaces, stripped of punctuation, and converted to uppercase. Each resulting component is checked to see if it is identical to an included locus abbreviation. If so, a match is found. 

+ For locus labels, the sequence record label is converted to uppercase (punctuation is left intact). The line is then checked to see if contains the particular locus label. If so, a match is found. 

+ If a locus abbreviation ***or*** a locus label is matched to the contents of a sequence record, the record will pass the filtering step. 

**Example of locus abbreviation search:**

If the locus file contains:

`CMOS	cmos;c-mos	oocyte maturation factor`

The abbreviations will include:

```
CMOS
C-MOS
```

Notice the search terms are converted to uppercase.

If the sequence record contains the following label:

`>JX838886.1 Acanthocercus annectens voucher CAS 227508; oocyte maturation factor (CMOS) gene, partial cds`

Then it will result in the following components:

```
>JX838886.1
ACANTHOCERCUS
ANNECTENS
VOUCHER
CAS
227508
OOCYTE
MATURATION
FACTOR
CMOS
GENE
PARTIAL
CDS
```

Notice the line is stripped of all punctuation (including parentheses) and converted to uppercase. In this example, a match will be found using the CMOS search term, but not the C-MOS term.

**Example of locus label search:**

If the locus file contains:

`EXPH5	EXPH5	exophilin;exophilin 5;exophilin-5;exophilin protein 5`

The labels will include:

```
EXOPHILIN
EXOPHILIN 5
EXOPHILIN-5
EXOPHILIN PROTEIN 5
```
Notice the search terms are converted to uppercase.

If the sequence record contains the following label:

`>JX999516.1 Liolaemus pictus voucher LP111; exophilin 5 (EXPH5) gene, partial cds`

It will be converted to the following search line:

`>JX999516.1 LIOLAEMUS PICTUS VOUCHER LP111; EXOPHILIN 5 (EXPH5) GENE, PARTIAL CDS`

Notice punctuation and parentheses are left intact and the line is simply converted to uppercase. The line is then checked to see if any supplied locus label is contained within. In this example, the `EXOPHILIN 5` and `EXOPHILIN` labels are both contained in the line and would produce a match, but `EXOPHILIN-5` and `EXOPHILIN PROTEIN 5` would not. The more specific or complex a label search term is, the less likely it is to produce an exact match. My recommendation is to find the simplest common denominator among records and include that label, along with more complex search labels.

---------------

### Searching for UCE loci <a name="SFUL"></a> 

The strategy for obtaining sets of UCE sequences is a little different from the smaller locus sets. First, you'll want to do a specific GenBank search for the taxonomic term of interest *and* `ultra conserved element` and/or `uce`. This will produce a much more manageable set of sequences to work with, as searching for several thousand loci in a large sequence set will inevitably take a long time. 

To generate a locus search terms file, I retrieved the uce names from the ***uce-5k-probes.fasta*** file located [here](https://github.com/faircloth-lab/uce-probe-sets/tree/master/uce-5k-probe-set). Unfortunately, there does not appear to be a standard naming convention for the UCE loci on GenBank. If the sequences have been properly curated, then the description lines *should* contain the uce name somewhere (uce-10, uce-453, uce-5810, etc). If so, they will be compatible with the the 5k UCE locus search terms file (`Locus-Search-Terms_UCE_5k_set.txt`) I've made available in the data folder [here](https://github.com/dportik/SuperCRUNCH/tree/master/data).

Here are partial contents from the UCE locus search term file:

```
uce-5805	uce-5805	xxxxxxxxxxxx
uce-5806	uce-5806	xxxxxxxxxxxx
uce-5808	uce-5808	xxxxxxxxxxxx
uce-5810	uce-5810	xxxxxxxxxxxx
```

Notice the third column is junk. Unfortunately, UCE loci have been numbered in a suboptimal way. For example, the label `uce-1` is used instead of `uce-0001`. This causes problems when searching for locus labels, because the term `uce-1` is contained inside of `uce-10`, `uce-104`, `uce-1638`, etc. Because of this, the label search will not work properly, and so we have to rely exclusively on the abbreviation to find the correct sequences.

Here is an example of some frog UCE records I searched:

```
>KY160876.1 Kaloula kalingensis voucher RMB1887 ultra conserved element locus uce-5806 genomic sequence
ATATTTGTGTTTATTTTCTACTTGTATTAATTGACAACATTTGCCTGTTGGCTCAAGGGAATCAGTGTTC
CCATTTTATGCACTCTATTTTAAAATGCAGACAGTGGTAGAACAGATGTGTTTTTTTTAACCCCATA...

>KY160875.1 Kaloula pulchra voucher KU328278 ultra conserved element locus uce-5806 genomic sequence
ATATTTGTGTTTATTTTCTACTTGTATTAATTGACAACATTTGCCTGTTGGCTTAAGGGAATCATTGTTG
CCATTTTATGCACTCTATTTTAAAATGCATACAGTGGTAGAACAGATGTGTTTTTTTAACCCCATAG...

>KY160874.1 Kaloula picta voucher KU321376 ultra conserved element locus uce-5806 genomic sequence
ATATTTGTGTTTATTTTCTACTTGTATTAATTGACAACATTTGCCTGTTGGCTCAAGGGAATCAGTGTTG
CCATTTTATGCACTCTATTTTAAAATGCAGACAGTGGTAGAACAGATGTGTTTTTTTTAACCCCATA...

>KY160873.1 Kaloula conjuncta negrosensis voucher KU328639 ultra conserved element locus uce-5806 genomic sequence
ATATTTGTGTTTATTTTCTACTTGTATTAATTGACAACATTTGCCTGTTGGCTCAAGGGAATCAGTGTTG
CCATTTTATGCACTCTATTTTAAAATGCAGACAGTGGTAGAACAGATGTGTTTTTTTTAACCCCATA...
```

The locus abbreviation terms successfully retrieved the corresponding loci in the example above. 

The 5k UCE locus search terms file (`Locus-Search-Terms_UCE_5k_set.txt`) is freely available in the data folder [here](https://github.com/dportik/SuperCRUNCH/tree/master/data), and it can be used to retrieve UCE data from the 5k set as long as the records have the UCE locus name in the description lines.

---------------

---------------

## **Taxon Filtering and Locus Extraction** <a name="TFLE"></a>

![F1](https://github.com/dportik/SuperCRUNCH/blob/master/docs/Fig1.jpg)

This section deals with the first filtering steps, which include screening sequence records for taxon names and locus identity. To run these steps in **SuperCRUNCH**, you will need to provide a fasta file of downloaded nucleotide sequence records, a file containing a list of taxa, and a file containing a list of loci and associated search terms. Information on obtaining these files is provided in the section above. The `Taxa_Assessment.py` and `Rename_Merge.py` scripts are optional, but highly recommended. `Taxa_Assessment.py` identifies all valid and invalid taxon names contained within the starting fasta file. `Rename_Merge.py` is an optional data cleaning step that can be used to replace invalid taxon names with updated valid names for corresponding  sequence records. This relabeling step allows these records to pass the taxonomy filter in `Parse_Loci.py`, rather than be discarded. The original or updated fasta file is processed using `Parse_Loci.py`. For each locus included, a fasta file is produced. These fasta files contain records with valid taxon names that matched one or more of the search terms for the locus.

---------------

### Taxa_Assessment.py <a name="TA"></a>

The goal of this script is to search through records in a fasta file of NCBI nucleotide sequences to determine whether or not they contain a taxon name present in the description line that matches a taxon name in the user-supplied taxon list. The taxon names list can contain a mix of species (binomial name) and subspecies (trinomial name) labels.

Two output fasta files are written to the specified output directory: one containing only records with valid taxon names (`Matched_Taxa.fasta`), and one containing records with invalid taxon names (`Unmatched_Taxa.fasta`). The accession numbers for each of these fasta files are also written to separate files (`Matched_Records_Accession_Numbers.log`, `Unmatched_Records_Accession_Numbers.log`). Two log files are written which contain lists of the valid (`Matched_Taxon_Names.log`) and invalid taxon names (`Unmatched_Taxon_Names.log`) found across all records. The `Unmatched_Taxon_Names.log` file can be used to create the base file needed to relabel taxa in the `Rename_Merge.py` script. 

The decision to include or exclude subspecies labels is up to the user, and can be specified using the `--no_subspecies` flag. For a thorough explanation of how taxonomy searches are conducted and how this flag affects this step (and others), please see below. For all searches the sequence description line and the supplied taxon name are converted to uppercase, so the list of taxon names is not case-sensitive.

#### Basic Usage:

```
python Taxa_Assessment.py -i <fasta file> -t <taxon file> -o <output directory>
```

#### Argument Explanations:

##### `-i <path-to-file>`

> **Required**: The full path to a fasta file of GenBank sequence data.

##### `-t <path-to-file>`

> **Required**: The full path to a text file containing all taxon names to cross-reference in the fasta file.

##### `-o <path-to-directory>`

> **Required**: The full path to an existing directory to write  output files.

##### `--no_subspecies`

> **Optional**: Ignore the subspecies component of taxon labels in the taxon names file and in the fasta file to search.

#### Example Use:

```
python Taxa_Assessment.py -i bin/Analysis/Start_Seqs.fasta -t bin/Analysis/Taxa_List.txt -o bin/Analysis/Output/
```
> Above command will search sequence records in `Start_Seqs.fasta` to find taxon names present in `Taxa_List.txt`, the output is written to the specified directory.

```
python Taxa_Assessment.py -i bin/Analysis/Start_Seqs.fasta -t bin/Analysis/Taxa_List.txt -o bin/Analysis/Output/ --no_subspecies
```
> Above command will search sequence records in `Start_Seqs.fasta` to find taxon names present in `Taxa_List.txt`, the output is written to the specified directory. This search will exclude the subspecies component of taxon labels in the taxon names file and fasta file.


#### Taxonomy searches with and without subspecies:

To understand how the `--no_subspecies` flag can impact analyses, it is important to demonstrate how the taxon list is being parsed. Regardless of the type of names present in this file (species or subspecies), two lists are constructed. One if filled with species (binomial) names, and the other with subspecies (trinomial) names.

Example taxon list file:

```
Leycesteria crocothyrsos
Leycesteria formosa
Linnaea borealis americana
Linnaea borealis borealis
Linnaea borealis longiflora
```

The resulting parsed lists are:

`species = [Leycesteria crocothyrsos, Leycesteria formosa, Linnaea borealis]`

`subspecies = [Linnaea borealis americana, Linnaea borealis borealis, Linnaea borealis longiflora]`

Notice that even though there wasn't a binomial name provided for *Linnaea borealis*, it was automatically generated based on the subspecies labels. This is true regardless of whether the `--no_subspecies` flag is included or not. 

**How does the `--no_subspecies` flag impact searches?**

I will use the above taxon list and the following example records to illustrate:

```
>FJ745393.1 Leycesteria crocothyrsos voucher N. Pyck 1992-1691 maturase K (matK) gene, partial cds; chloroplast
>KC474956.1 Linnaea borealis americana voucher Bennett_06-432_CAN maturase K (matK) gene, partial cds; chloroplast
```

Each sequence record is always parsed to construct a species (binomial) and subspecies (trinomial) name. This would produce the following results:

```
Leycesteria crocothyrsos #species
Leycesteria crocothyrsos voucher #subspecies
Linnaea borealis #species
Linnaea borealis americana #subspecies
```

As you can see above, every subspecies contains a species label. This allows a series of checks to be performed. If the `--no_subspecies` flag is omitted, the following checks are performed:

1. Is the reconstructed species name in the species list? 
    1. If no, the record is ignored.
    2. If yes, the subspecies is examined.
2. Is the reconstructed subspecies name in the subspecies list? 
    1. If no, the species name will be used. 
    2. If yes, the subspecies name will be used instead.

In the example above, `Leycesteria crocothyrsos` is in the species list, but `Leycesteria crocothyrsos voucher` is an obviously incorrect name and is absent from the subspecies list. In this case, the species name `Leycesteria crocothyrsos` will be used for that record. In the other example, `Linnaea borealis` is in the species list, but `Linnaea borealis americana` is also present in the subspecies list, so `Linnaea borealis americana` will be used for that record.

If the `--no_subspecies` flag is included, the following checks are performed:

1. Is the reconstructed species name in the species list? 
    1. If no, the record is ignored.
    2. If yes, the species name is used.

In the example above, `Leycesteria crocothyrsos` and `Linnaea borealis` would be the names used. Essentially, the `--no_subspecies` flag 'sinks' all the subspecies, and they are all lumped under the relevant species label.

Let's use another example.

Here, the taxon list file contains only species (binomial) names:

```
Draco beccarii
Draco biaro
Draco bimaculatus
Draco blanfordii
Draco boschmai
Draco bourouniensis
Draco cornutus
Draco cristatellus
```

If only binomial names are present, then the resulting species list will be populated and the resulting subspecies list will be empty:

`species = [Draco beccarii, Draco biaro, Draco bimaculatus, Draco blanfordii, Draco boschmai, Draco bourouniensis, Draco cornutus, Draco cristatellus]`

`subspecies = []`

In this example the `--no_subspecies` flag will have no effect on the analysis. That is, regardless of whether the `--no_subspecies` flag is used or not, there aren't any subspecies to reference and the only possible outcome is to find species names.

There are also some special cases depending on combinations of the taxon list and sequence set.

Given the following taxon list:

```
Linnaea borealis americana
Linnaea borealis borealis
Linnaea borealis longiflora
```

And the following record description lines:

```
>KJ593010.1 Linnaea borealis voucher WAB_0132469163 maturase K (matK) gene, partial cds; chloroplast
>KC474956.1 Linnaea borealis americana voucher Bennett_06-432_CAN maturase K (matK) gene, partial cds; chloroplast
>KP297496.1 Linnaea borealis borealis isolate BOP012344 internal transcribed spacer 1, partial sequence; 5.8S ribosomal RNA gene, complete sequence; and internal transcribed spacer 2, partial sequence
>KP297498.1 Linnaea borealis longiflora isolate BOP022790 internal transcribed spacer 1, partial sequence; 5.8S ribosomal RNA gene, complete sequence; and internal transcribed spacer 2, partial sequence
```

The following taxa would be detected and included from each record if the `--no_subspecies` flag is omitted:

```
KJ593010.1 -> Linnaea borealis
KC474956.1 -> Linnaea borealis americana
KP297496.1 -> Linnaea borealis borealis
KP297498.1 -> Linnaea borealis longiflora
```

Any record of `Linnaea borealis` missing a valid subspecies label will be lumped in with all other `Linnaea borealis`, whereas those containing valid subspecies labels will be assigned to the correct subspecies. 

The following taxa would be detected and included from each record if the `--no_subspecies` flag is included:

```
KJ593010.1 -> Linnaea borealis
KC474956.1 -> Linnaea borealis
KP297496.1 -> Linnaea borealis
KP297498.1 -> Linnaea borealis
```
This effectively groups all the subspecies under the species name `Linnaea borealis`.

**To summarize:**

+ If the taxon names list contains only species then searches for subspecies labels cannot occur, and therefore the presence or absence of the `--no_subspecies` flag has no effect.
+ If the taxon names list contains a mix of species and subspecies labels, then the `--no_subspecies` flag can substantially change the outcome. 
+ Using the `--no_subspecies` flag reduces all subspecies names to corresponding species names, and is expected to result in less taxa recovered. Depending on your conceptual view of subspecies, you may find this to be an awesome choice, or you may find it to be a terrible choice. 
+ Omitting the `--no_subspecies` flag is expected to produce a greater number of taxa, but only if valid subspecies are present in the starting sequences.
+ There is no downside to having subspecies labels in the taxon list file, because they can effectively be ignored while capturing all relevant species labels.

---------------

### Rename_Merge.py <a name="RM"></a>

The goal of this script is to search through records in a fasta file and replace invalid taxon names with new names that are compatible with the taxon list. The invalid taxon names file (`Unmatched_Taxon_Names.log`) from the `Taxa_Assessment.py` can be used to help create the replacement names file, as it contains all the taxon names that failed the taxon filter. A second column can be added, and as each name is inspected a replacement name can be added to the second column, or if the name cannot be rescued then the row can be deleted. The final replacement names file should be two-columns in tab-delimited format, and an example is shown below. Similar to other input text files, make sure to open the replacement names file in a text editor and ensure the format includes Unix line breaks (line breaks marked by `\n`, rather than `\r\n`) and UTF-8 encoding, otherwise extra characters may interfere with parsing the file correctly with **SuperCRUNCH**.

If the optional `-m` flag is used, records that are successfully relabeled are merged with another fasta file. Ideally, this fasta file should be the one containing all the records with valid taxon names (*Matched_Taxa.fasta*). The resulting merged fasta file (*Merged.fasta*) should then be used for the `Parse_Loci.py` script.

#### Basic Usage:

```
python Rename_Merge.py -i <fasta file> -r <taxon renaming file> -o <output directory>
```

#### Argument Explanations:

##### `-i <path-to-file>`

> **Required**: The full path to a fasta file with taxon names to replace (`Unmatched_Taxa.fasta`).

##### `-r <path-to-file>`

> **Required**: The full path to a two-column text file containing all taxon names to be replaced, and the replacement names.

##### `-o <path-to-directory>`

> **Required**: The full path to an existing directory to write output files.

##### `-m <full-path-to-file>`

> **Optional**: The full path to a fasta file containing valid taxon names (`Matched_Taxa.fasta`). 

#### Example Use:

```
python Rename_Merge.py -i bin/Rename/Unmatched_Taxa.fasta -r bin/Rename/taxon_relabeling.txt -o bin/Rename/Output/
```
> Above command will attempt to rename sequence records in `Unmatched_Taxa.fasta` following the file `taxon_relabeling.txt`, and the output is written to the specified directory.
```
python Rename_Merge.py -i bin/Rename/Unmatched_Taxa.fasta -r bin/Rename/taxon_relabeling.txt -o bin/Rename/Output/ -m bin/Rename/Matched_Taxa.fasta
```
> Above command will attempt to rename sequence records in `Unmatched_Taxa.fasta` following the file `taxon_relabeling.txt`, and merge these relabeled records with those in `Matched_Taxa.fasta`. The output is written to the specified directory.


In the replacement names file, the first column should contain the name that needs to be replaced (the invalid name), and the second column should contain the replacement name. Currently, `Rename_Merge.py` only supports species (binomial) name relabeling, so altering subspecies labels is not possible.

Example contents of a replacement names file:

```
Chamaeleo harennae	Trioceros harennae
Chamaeleo hoehneli	Trioceros hoehnelii
Chamaeleo hoehnelii	Trioceros hoehnelii
Chamaeleo jacksonii	Trioceros jacksonii
Chamaeleo johnstoni	Trioceros johnstoni
Chamaeleo melleri	Trioceros melleri
Chamaeleo montium	Trioceros montium
Chamaeleo narraioca	Trioceros narraioca
```

Note that components of a species name are separated with a space, and the columns are separated with a tab character. 

Although the replacement step should help rescue many records, there are some labels in the *Unmatched_Taxon_Names.log* that simply can't be corrected. These include things like:

```
A.alutaceus mitochondrial
A.barbouri mitochondrial
Agama sp.
C.subcristatus tcs1
C.versicolor sox-4
Calotes sp.
Calumma aff.
Calumma cf.
Liolaemus kriegi/ceii
Tsa anolis
Unverified bradypodion
Unverified callisaurus
```

These records have been labeled improperly, or the identity of the organism is uncertain (*sp., cf., aff.*). These are not useful for the analysis, and should rightfully be discarded using the taxonomy filter in `Parse_Loci.py`. Yuck!

In other cases, taxon names may have been updated and now represent synonymies, or may have been accidentally misspelled. Using a organism-specific taxonomy browser can help clarify these situations. Synonymies, misspellings, and name changes represent examples of records that are worth rescuing through relabeling, and using `Rename_Merge.py` to do so will result in higher quality data. 

---------------

### Parse_Loci.py <a name="PL"></a>

The goal of `Parse_Loci.py` is to search through the fasta file of starting sequences and identify records for each gene/marker/locus. Each record that matches a locus must also contain a valid taxon name to be retained. To run `Parse_Loci.py`, you will need to provide a fasta file of downloaded nucleotide sequence records, a file containing a list of taxa, and a file containing a list of loci and associated search terms.  

The taxon names list can contain a mix of species (binomial) and subspecies (trinomial) names. Detailed instructions for the format of this file is provided in the `Taxa_Assessment.py` section. The optional `--no_subspecies` flag can be used, and its effect is also described in great detail in the `Taxa_Assessment.py` section.

The locus file contains the search terms that are used to identify matching records. Detailed information about the format of this file can be found in the **Obtaining Loci and Search Terms** section. 
 
#### Basic Usage:

```
python Parse_Loci.py -i <fasta file> -l <locus term file> -t <taxon file> -o <output directory>
```

#### Argument Explanations:

##### `-i <path-to-file>`

> **Required**: The full path to a fasta file of GenBank sequence data.

##### `-l <path-to-file>`

> **Required**: The full path to a three-column text file containing loci information to search for within the fasta file.

##### `-t <path-to-file>`

> **Required**: The full path to a text file containing all taxon names to cross-reference in the fasta file.

##### `-o <path-to-directory>`

> **Required**: The full path to an existing directory to write output files.

##### `--no_subspecies`

> **Optional**: Ignore subspecies labels in both the taxon names file and the fasta file.

#### Example Use:

```
python Parse_Loci.py -i bin/Loci/Merged.fasta -l bin/Loci/locus_search_terms.txt -t bin/Loci/Taxa_List.txt -o bin/Loci/Output/
```
> Above command will use the locus search terms file `locus_search_terms.txt` and the taxon names file `Taxa_List.txt` to parse records in `Merged.fasta`, writing outputs to the specified directory.
```
python Parse_Loci.py -i bin/Loci/Merged.fasta -l bin/Loci/locus_search_terms.txt -t bin/Loci/Taxa_List.txt -o bin/Loci/Output/ --no_subspecies
```
> Above command will use the locus search terms file `locus_search_terms.txt` and the taxon names file `Taxa_List.txt` to parse records in `Merged.fasta`, ignoring the subspecies component of taxon names. Outputs are written to the specified directory.


Several output files are created in the directory specified. For each locus included, a fasta file will be written with sequences that pass the locus and taxon filters. If no sequences are found for a locus, a corresponding fasta file will not be produced. A log file summarizing the number of records written per locus is also written to the output directory, and is called *Loci_Record_Counts.log*. An example of the contents of this file is shown below:

```
Locus_Name	Records_Written
BDNF	1246
CMOS	1263
CXCR4	164
EXPH5	650
KIAA1549	0
```
In the example above, the files `BDNF.fasta`, `CMOS.fasta`, `CXCR4.fasta`, and `EXPH5.fasta` will be written to the output directory, but not `KIAA1549.fasta` because no sequences were found. 

Identifying loci in the sequence records through word matching is not expected to be perfect, and there may be non-target sequences in the resulting fasta files. For this reason, the fasta files should be subjected to orthology filtering before attempting to create sequence alignments or performing analyses, as described in the next section.

---------------

---------------

## **Orthology Filtering** <a name="OF"></a>

![F2](https://github.com/dportik/SuperCRUNCH/blob/master/docs/Fig2.jpg)

There are two main methods in **SuperCRUNCH** that can be used to filter out non-orthologous sequences in the locus-specific fasta files. Each relies on using BLAST searches to extract homologous sequence regions, but they differ in whether they require user-supplied reference sequences. `Cluster_Blast_Extract.py` works by creating sequence clusters based on similarity using CD-HIT-EST. A BLAST database is constructed from the largest cluster, and all sequences from the fasta file are blasted to this database. In contrast, `Reference_Blast_Extract.py` relies on a previously assembled set of reference sequences to construct the BLAST database, and all sequences from the fasta file are blasted to this database. Both methods offer the ability to specify the BLAST algorithm to use (*blastn, megablast, dc-megablast*), and multiple options for extracting BLAST coordinates. These topics are discussed in greater detail below. `Cluster_Blast_Extract.py` is recommended for 'simple' sequence record sets, in which each record contains sequence data for the same region of a single locus. `Reference_Blast_Extract.py` is recommended for more complex sequence records, in which each record contains multiple loci (long mtDNA fragment, whole organellar genome, etc.) or the records contain non-overlapping fragments of the same locus. The reference sequence set ensures that only the regions of interest are extracted from these records. Examples of reference sequence sets are available in the data folder [here](https://github.com/dportik/SuperCRUNCH/tree/master/data).

An optional contamination filtering step is also available, called `Contamination_Filter.py`. This step requires a user-supplied set of sequences that represent the source of 'contamination'. For example, amphibian mtDNA sequences can be screened against human mtDNA sequences to make sure they are actually amphibian. Any set of reference sequences can be used, and the context depends on what the contamination source is expected to be. This step will remove all sequences scoring greater than 95% identity for a minimum of 100 continuous base pairs to the reference ‘contamination’ sequences. 

The fasta files resulting from this step can be used for the quality filtering and sequence selection stage. 

---------------

### Cluster_Blast_Extract.py <a name="CBE"></a>

`Cluster_Blast_Extract.py` is one of two methods for detecting and removing non-orthologous sequences from locus-specific fasta files. This method is recommended for simple sequence record sets, in which each record contains sequence data for the same region of a single locus. `Cluster_Blast_Extract.py` works by creating sequence clusters based on similarity using CD-HIT-EST. A BLAST database is constructed from the largest cluster, and all sequences from the fasta file are blasted to this database using the specified BLAST algorithm (*blastn, megablast,* or *dc-megablast*). For each sequence with a significant match, the coordinates of all BLAST hits (excluding self-hits) are merged. This action often results in a single interval, but non-overlapping coordinates can also be produced. Multiple options are provided for handling these cases, with details on this topic provided below. All query sequences with significant hits are extracted based on the resulting BLAST coordinates, and written to a new filtered fasta file. 

#### Basic Usage:

```
python Cluster_Blast_Extract.py -i <fasta file directory> -b <blast algorithm> -m <blast coordinate strategy>
```

#### Argument Explanations:

##### `-i <path-to-directory>`

> **Required**: The full path to a directory containing the parsed locus-specific fasta files. Fasta files in the directory must have the extension '.fasta' to be read.

##### `-b <choice>`

> **Required**: The blast algorithm to use. Choices = *blastn, blastn-short, dc-megablast, megablast*. **Recommended**: *dc-megablast*.

##### `-m <choice>`

> **Optional**: The strategy for dealing with multiple non-overlapping blast coordinates. Choices = *span, nospan, all*. Default = *span*.

##### `--max_hits <integer>`

> **Optional**: The maximum number of blast matches allowed per input sequence. May want to set < 300 for large sequence sets. If omitted, no limit is set.

#### Example Use:

```
python Cluster_Blast_Extract.py -i bin/cluster-blast/ -b dc-megablast -m span --max_hits 300
```
> Above command will perform automated clustering, BLASTing using *dc-megablast* (with hit limit imposed), and the *span* strategy for BLAST coordinates for each unaligned fasta file present in the `cluster-blast/` input directory.


Several output folders are created in the directory containing the input fasta files. The directory labels and their contents are described below:

+ **01_Clustering_Results/**
    + For each locus, this directory contains the output files from cd-hit-est (ex., `LOCUS1_Out_.clstr`), which delimit the sequence clusters.
+ **02_Parsed_Results/**
    + For each locus, this directory contains a set of fasta files which represent the clusters found. These are labeled as `LOCUS1_Out_Cluster_0.fasta`, `LOCUS1_Out_Cluster_1.fasta`, etc.
+ **03_Blast_Results/**
    + For each locus, this directory contains the constructed blast database files (extensions .nhr, .nin, .nsq), the blast results for each fasta cluster (ex., `LOCUS1_Out_Cluster_0_blast_results.txt`), and the merged blast results for all clusters (ex., `LOCUS1_blast_results_merged.txt`). 
+ **04_Trimmed_Results/**
    + For each locus, this directory contains the filtered fasta file (`LOCUS1_extracted.fasta`) and a corresponding log file (`Log_File_LOCUS1.txt`) that indicates the original sequence length, BLAST coordinates found, and extracted sequence length for each record that passed this filter.
    
#### BLAST algorithm choice

The required `-b ` flag specifies the BLAST algorithm to use, which can greatly affect the filtering results. The *blastn* algorithm searches with a word size of 11, whereas *megablast* searches include a word size of 28, making *blastn* more appropriate for interspecies searches and *megablast* more appropriate for closely related or intraspecific searches. However, *discontiguous megablast* (*dc-megablast*) is better at producing non-fragmented hits for divergent sequences using similar word sizes as *blastn*, and as such it works well for interspecific and intraspecific searches. If the goal is to produce species level phylogenetic data sets then *dc-megablast* should be used, but if the focus is on population level phylogenetic data sets then *megablast* may be preferable. You can easily compare the effects of the different BLAST algorithms, as the coordinates used to extract sequences are readily available in the log files produced in the final `/04_Trimmed_Results` directory.

#### BLAST coordinates strategy

The optional `-m ` flag specifies the strategy to use for handling BLAST coordinates. To explain these options, a little background is necessary. When a sequence is blasted, the sections of the query sequence that match the subject sequence (reference) are reported as start and stop coordinates. In general, many hits are produced for the query sequence and the result is many sets of coordinates. Often, these coordinates overlap and can be combined. Take for example the following set of coordinates:

```
[2, 435]
[27, 380]
[30, 500]
```
These can be combined to one set of coordinates: `[2, 500]`. After BLAST searches are finished, the coordinates for each input sequence are merged. Often this produces one set of merged coordinates, like the example above, but sometimes multiple sets of non-overlapping coordinates are produced, like `[2, 70], [100, 500]`. 

How can this happen?

One common reason is the sequence contains a stretch of `N` characters. These will never be matched. Take the following sequence:

```
TCATGTTCCANNNNNNNNNNCGAAAAATGATGCTG
```

This sequence will at best produce the following coordinates: `[1, 10], [20, 35]`. The N's are always ignored. 

Another more problematic reason is that duplicate sequences are found, and the coordinates of both duplicates are returned. This can happen in mitochondrial genomes, which sometimes have undergone gene duplications. In cases like this, the genes tend to be separated by quite a long distance. Given a mitochondrial genome, the duplicate genes may return a set of coordinates that look like this: `[300, 860], [4800, 5300]`. The huge gap between these coordinates is a strong signal that duplicate genes are present. 

Given these different scenarios, there are three options meant to handle non-overlapping coordinates, including `span`, `nospan`, and `all`. 

+ When `-m span` is used, if the non-overlapping coordinate sets are less than or equal to 100 base pairs apart, they are merged. In the above 'N' example, this would produce the final coordinates of `[1, 35]`, and the resulting sequence will contain the original stretch of N characters. If the non-overlapping coordinate sets are more than 100 base pairs apart, the coordinates set containing the greatest number of base pairs is selected. In the 'duplication' scenario above, this would produce `[300, 860]`. The `span` option can safely hand the gene duplication scenario, and also allow longer lower-quality sequences to be extracted - as long as they contain a reasonably small stretch of N's. It is the default option used if the `-m ` flag is omitted.

+ When `-m nospan` is used, no attempt is made to merge the non-overlapping coordinates and the coordinate set containing the greatest number of base pairs is selected. In the above 'N' example, this would produce the final coordinates of `[20, 35]` (different from `span`). In the 'duplication' scenario, this would produce `[300, 860]` (same as `span`). The `nospan` option guarantees long stretches of N's will not be present in the extracted sequences, and can penalize these lower-quality sequences by reducing their length. It will also correctly handle the gene duplication scenario. This option should be viewed as a more conservative implementation of `span`.

+ When `-m all` is used, the non-overlapping coordinate sets are used as is. In the 'N' scenario above, this would produce `[1, 10], [20, 35]`, which would simply remove the stretch of N's from the sequence. That is, `TCATGTTCCANNNNNNNNNNCGAAAAATGATGCTG` becomes 
`TCATGTTCCACGAAAAATGATGCTG`. Although this seems like a desirable outcome, the same strategy will severely backfire for duplications. For the duplication scenario, this would produce `[300, 860], [4800, 5300]`. In other words, two duplicate genes would be extracted, producing a single sequence that is double the length of normal sequences. For duplications, this is a very poor outcome because it will interfere with sequence alignment. This option can be used to detect duplications in mitogenomes, or paralogous sequences. For example, when I ran this option using many reptile mitogenomes, I was able to find all the gene duplications in mitogenomes for the genus *Heteronotia* (a parthenogenic gecko). Beyond this use, I would caution against using this option unless you inspect the results very carefully. 

You can easily compare the effects of the options for the `-m ` flag, as the coordinates used to extract sequences are readily available in the log files produced in the final `/04_Trimmed_Results` directory.

---------------

### Reference_Blast_Extract.py <a name="RBE"></a>

`Reference_Blast_Extract.py` is one of two methods for detecting and removing non-orthologous sequences from locus-specific fasta files. This method is recommended for more complex sequence records, in which some records contain multiple loci (long mtDNA fragment, whole organellar genome, etc.) or the records contain non-overlapping fragments of the same locus. `Reference_Blast_Extract.py` works by creating a BLAST database from the reference sequences, and all sequences from the fasta file are blasted to this database using the specified BLAST algorithm (*blastn, megablast,* or *dc-megablast*). The reference sequence set ensures that only the target region is extracted from the sequence records. For each sequence with a significant match, the coordinates of all BLAST hits (excluding self-hits) are merged. This action often results in a single interval, but non-overlapping coordinates can also be produced. Multiple options are provided for handling these cases, with details on this topic provided below. All query sequences with significant hits are extracted based on the resulting BLAST coordinates, and written to a new filtered fasta file. 

#### Basic Usage:

```
python Reference_Blast_Extract.py -i <input directory> -d <reference fasta name> -e <empirical fasta name> -b <blast algorithm> -m <blast coordinate strategy>
```

#### Argument Explanations:

##### `-i <path-to-directory>`

> **Required**: The full path to a directory containing the reference fasta file and the empirical fasta file.

##### `-d <filename>`

> **Required**: The name of the reference fasta file that will be used to create the blast database. Requires file name only, NOT full path, as it should be located in the input directory (-i).

##### `-e <filename>`

> **Required**: The name of the empirical fasta file to blast to the database to prune sequences. Requires file name only, NOT full path, as it should be located in the input directory (-i).

##### `-b <choice>`

> **Required**: The blast algorithm to use. Choices = *blastn, blastn-short, dc-megablast, megablast*. **Recommended**: *dc-megablast*.

##### `-m <choice>`

> **Optional**: The strategy for dealing with multiple non-overlapping blast coordinates. Choices = *span, nospan, all*. Default = *span*.

##### `--max_hits <integer>`

> **Optional**: The maximum number of blast matches allowed per input sequence. May want to set < 300 for large sequence sets.

#### Example Use:

```
python Reference_Blast_Extract.py -i bin/Ref-Blast/ -d ND2_references.fasta -e ND2.fasta -b dc-megablast -m span --max_hits 300
```
> Above command will form a BLAST database from `ND2_references.fasta` and BLAST sequences from `ND2.fasta` to the database using the *dc-megablast* algorithm, *span* strategy for BLAST coordinate merging, and a hit limit of 300. Both files are in the `bin/Ref-Blast/` directory, where outputs are also written.

Several outputs are created in the specified input directory, including:

+ BLAST database files for the reference sequences (extensions .nhr, .nin, .nsq).
+ A BLAST results file, labeled as `[LOCUS1]_blast_output.txt`.
+ The filtered fasta file, labeled as `[LOCUS1]_extracted.fasta`
+ A corresponding log file (`Log_File_[LOCUS1].txt`) that indicates the original sequence length, BLAST coordinates found, and extracted sequence length for each record with significant BLAST hits to the reference sequences.

Similar to `Cluster_Blast_Extract.py`, the same options exist for choosing a BLAST algorithm (`-b `) and the BLAST coordinates strategy (`-m `). For convenience, this information is also posted here. 

#### BLAST algorithm choice

The required `-b ` flag specifies the BLAST algorithm to use, which can greatly affect the filtering results. The *blastn* algorithm searches with a word size of 11, whereas *megablast* searches include a word size of 28, making *blastn* more appropriate for interspecies searches and *megablast* more appropriate for closely related or intraspecific searches. However, *discontiguous megablast* (*dc-megablast*) is better at producing non-fragmented hits for divergent sequences using similar word sizes as *blastn*, and as such it works well for interspecific and intraspecific searches. If the goal is to produce species level phylogenetic data sets then *dc-megablast* should be used, but if the focus is on population level phylogenetic data sets then *megablast* may be preferable. You can easily compare the effects of the different BLAST algorithms, as the coordinates used to extract sequences are readily available in the log files produced in the final `/04_Trimmed_Results` directory.

#### BLAST coordinates strategy

The optional `-m ` flag specifies the strategy to use for handling BLAST coordinates. To explain these options, a little background is necessary. When a sequence is blasted, the sections of the query sequence that match the subject sequence (reference) are reported as start and stop coordinates. In general, many hits are produced for the query sequence and the result is many sets of coordinates. Often, these coordinates overlap and can be combined. Take for example the following set of coordinates:

```
[2, 435]
[27, 380]
[30, 500]
```
These can be combined to one set of coordinates: `[2, 500]`
After BLAST, the coordinates for each input sequence are combined. Often this produces one set of merged coordinates, like the above example. Sometimes multiple sets of non-overlapping coordinates are produced, like `[2, 70], [100, 500]`. 

How can this happen?

One common reason is the sequence contains a stretch of `N` characters. These will never be matched. Take the following sequence:

```
TCATGTTCCANNNNNNNNNNCGAAAAATGATGCTG
```

This sequence will at best produce the following coordinates: `[1, 10], [20, 35]`. The N's are always ignored. 

Another more problematic reason is that duplicate sequences are found, and the coordinates of both duplicates are returned. This can happen in mitochondrial genomes, which sometimes have undergone gene duplications. In cases like this, the genes tend to be separated by quite a long distance. Given a mitochondrial genome, the duplicate genes may return a set of coordinates that look like this: `[300, 860], [4800, 5300]`. The huge gap between these coordinates is a strong signal that duplicate genes are present. 

Given these different scenarios, there are three options meant to handle non-overlapping coordinates, including `span`, `nospan`, and `all`. 

+ When `-m span` is used, if the non-overlapping coordinate sets are less than or equal to 100 base pairs apart, they are merged. In the above 'N' example, this would produce the final coordinates of `[1, 35]`, and the resulting sequence will contain the original stretch of N characters. If the non-overlapping coordinate sets are more than 100 base pairs apart, the coordinates set containing the greatest number of base pairs is selected. In the 'duplication' scenario above, this would produce `[300, 860]`. The `span` option can safely hand the gene duplication scenario, and also allow longer lower-quality sequences to be extracted - as long as they contain a reasonably small stretch of N's. It is the default option used if the `-m ` flag is omitted.

+ When `-m nospan` is used, no attempt is made to merge the non-overlapping coordinates and the coordinate set containing the greatest number of base pairs is selected. In the above 'N' example, this would produce the final coordinates of `[20, 35]` (different from `span`). In the 'duplication' scenario, this would produce `[300, 860]` (same as `span`). The `nospan` option guarantees long stretches of N's will not be present in the extracted sequences, and can penalize these lower-quality sequences by reducing their length. It will also correctly handle the gene duplication scenario. This option should be viewed as a more conservative implementation of `span`.

+ When `-m all` is used, the non-overlapping coordinate sets are used as is. In the 'N' scenario above, this would produce `[1, 10], [20, 35]`, which would simply remove the stretch of N's from the sequence. That is, `TCATGTTCCANNNNNNNNNNCGAAAAATGATGCTG` becomes 
`TCATGTTCCACGAAAAATGATGCTG`. Although this seems like a desirable outcome, the same strategy will severely backfire for duplications. For the duplication scenario, this would produce `[300, 860], [4800, 5300]`. In other words, two duplicate genes would be extracted, producing a single sequence that is double the length of normal sequences. For duplications, this is a very poor outcome because it will interfere with sequence alignment. This option can be used to detect duplications in mitogenomes, or paralogous sequences. For example, when I ran this option using many reptile mitogenomes, I was able to find all the gene duplications in mitogenomes for the genus *Heteronotia* (a parthenogenic gecko). Beyond this use, I would caution against using this option unless you inspect the results very carefully. 

You can easily compare the effects of the options for the `-m ` flag, as the coordinates used to extract sequences are readily available in the log files produced in the final `/04_Trimmed_Results` directory.

---------------

### Contamination_Filter.py <a name="CF"></a>

`Contamination_Filter.py` is an optional step for additional filtering, which can help identify and remove 'contaminated' sequences. This step requires a user-supplied set of sequences that represent the source of 'contamination'. For example, amphibian mtDNA sequences can be screened against human mtDNA sequences to make sure they are actually amphibian. Any reference sequences can be used, and the context depends on what the contamination source is expected to be. This step will remove all sequences scoring greater than 95% identity for a minimum of 100 continuous base pairs to the reference ‘contamination’ sequences. 

#### Basic Usage:

```
python Contamination_Filter.py -i <input directory> -d <contamination fasta name> -e <empirical fasta name> -b <blast algorithm> -m <blast coordinate strategy>
```

#### Argument Explanations:

##### `-i <path-to-directory>`

> **Required**: The full path to a directory containing the reference fasta file and the empirical fasta file.

##### `-d <filename>`

> **Required**: The name of the contamination reference fasta file that will be used to create the blast database. Requires file name only, NOT full path, as it should be located in the input directory (-i).

##### `-e <filename>`

> **Required**: The name of the empirical fasta file to blast to the database to find bad sequences. Requires file name only, NOT full path, as it should be located in the input directory (-i).

##### `-b <choice>`

> **Required**: The blast algorithm to use. Choices = *blastn, blastn-short, dc-megablast, megablast*. **Recommended**: *megablast*.

##### `--max_hits <integer>`

> **Optional**: The maximum number of blast matches allowed per input sequence.

#### Example Use:

```
python Contamination_Filter.py -i bin/contamfilter/ND2/ -d Human_ND2.fasta -e ND2.fasta -b megablast
```
> Above command will form a BLAST database from `Human_ND2.fasta` and BLAST sequences from `ND2.fasta` to the database using the *megablast* algorithm. Both files are in the `bin/Ref-Blast/` directory, where output files are also written.

Several outputs are created in the specified input directory, including:

+ BLAST database files for the 'contamination' reference sequences (extensions .nhr, .nin, .nsq).
+ A BLAST results file, labeled as `[LOCUS1]_blast_output.txt`.
+ A filtered fasta file, labeled as `[LOCUS1]_extracted.fasta`, which contain all sequences that passed the filter.
+ A fasta file of 'contaminated' sequences, labeled as `[LOCUS1]_extracted_contaminated.fasta`, which contains sequences that failed the filter.
+ A corresponding log file (`Log_File_[LOCUS1].txt`), which contains information on the original sequence length, BLAST coordinates found, and extracted sequence length for each record with significant BLAST hits to the reference sequences.

#### BLAST algorithm choice

The required `-b ` flag specifies the BLAST algorithm to use. For the contamination filter, the goal is to identify and remove sequences with a very high similarity to the references. For this type of search it is best to use *megablast*, which is most appropriate for conducting within-species searches.

---------------

---------------

## **Sequence Quality Filtering and Selection** <a name="SQFS"></a>

![F3](https://github.com/dportik/SuperCRUNCH/blob/master/docs/Fig3.jpg)

The goal of this step is to apply additional sequence filters and select representative sequences using `Filter_Seqs_and_Species.py`. There are multiple options for filtering and selecting sequences, including an option to include or exclude subspecies. Once sequence filtering and selection is completed, the `Make_Acc_Table.py` module can be used to rapidly generate a table of NCBI accession numbers for all taxa and loci.

Although the original goal of `Filter_Seqs_and_Species.py` was to select the best available sequence per taxon per locus, it can also be used to retain all sequences passing the filters. That is, it can be used to filter and create population-level data sets, in which all available sequences for each taxon are retained.

If sequences have been filtered for species-level data sets (one sequence per taxon), the total number of possible supermatrix combinations can be calculated using `Infer_Supermatrix_Combinations.py`. This is a fun exercise.

---------------

### Filter_Seqs_and_Species.py <a name="FSS"></a>

To construct a phylogenetic supermatrix representative sequences must be selected per taxon per locus, and if multiple sequences exist for a taxon for a locus then an objective strategy must be used for sequence selection. `Filter_Seqs_and_Species.py` includes several strategies for choosing among multiple sequences. 

+ The simplest solution is to sort sequences by length and select the longest sequence available (`-f length`). 

+ `Filter_Seqs_and_Species.py` includes another approach for coding loci in which sequences are first sorted by length, and then undergo translation tests to identify a correct reading frame (`-f translate`). Here, the longest sequence is translated in all forward and reverse frames in an attempt to identify a correct reading frame. If a correct reading frame is found, the sequence is selected. If a correct reading frame is not found, the next longest sequence is examined. The process continues until a suitable sequence is found. If no sequences in the set pass translation, the first (longest) sequence is selected (rather than excluding the taxon). If the `-f translate` method is used, the translation table must also be specified using the `--table` flag. All NCBI translation table options are available, and can be selected using integers or shortcut terms provided (*standard, vertmtdna, invertmtdna, yeastmtdna, plastid*, or any integer *1-31*). If the `--table` flag is omitted, the default will be to use the Standard translation table. 

+ The `--randomize` feature can be used to shuffle the starting list of sequences randomly, rather than sorting by length. If used in conjunction with the `-f length` method, the first sequence from the shuffled list is selected (a true random choice). If used in conjunction with the `-f translate` method, translation tests occur down the shuffled list until a suitable sequence is selected (not necessarily a random choice). 

For all selection options, sequences must meet a minimum base pair threshold set by the user. If no sequences meet the minimum length requirement for a taxon, then the taxon will be eliminated during the filtering process. To avoid eliminating short sequences a small integer can be used, although the retention of very short sequences can interfere with sequence alignment.

Similar to previous steps, a list of taxa must be provided. The taxon names list can contain a mix of species (binomial) and subspecies (trinomial) names. Detailed instructions for the format of this file is provided in the `Taxa_Assessment.py` section. The optional `--no_subspecies` flag can be used, and its effect in this step is identical to that described in the `Taxa_Assessment.py` section.

Using one of the sequence selection strategies above, the `Filter_Seqs_and_Species.py` module will create fasta files containing a single representative sequence per taxon for each locus, along with other important output files described below. The filtered fasta files can be used with the `Make_Acc_Table.py` module to generate a table of NCBI accession numbers for all taxa and loci.
 
Although `Filter_Seqs_and_Species.py` was initially designed to filter and select one sequence per species, retaining intraspecific sequence sets may actually be desirable for other projects (e.g., phylogeography). `Filter_Seqs_and_Species.py` includes the option to create these data sets using the `--allseqs` flag. When the `--allseqs` flag is used, rather than selecting the highest quality sequence available for a taxon, all sequences passing the filtration methods are retained. 


#### Basic Usage:

```
python Filter_Seqs_and_Species.py -i <input directory> -f <filter strategy> -l <minimum base pairs> -t <taxon file>
```

#### Argument Explanations:

##### `-i <path-to-directory>`

> **Required**: The full path to a directory which contains the locus-specific fasta files to filter. The filtering options set are applied to every fasta file in this directory. Fasta files in the directory must have extensions '.fasta' or '.fa' to be read.

##### `-f <choice>`

> **Required**: Strategy for filtering sequence data. Choices = *translate, length*.

##### `-l <integer>`

> **Required**: An integer for the minimum number of base pairs required to keep a sequence (ex. 150).

##### `-t <path-to-file>`

> **Required**: The full path to a text file containing all taxon names to cross-reference in the fasta file.

##### `--table <choice>`

> **Required for** `-f translate`: Specifies translation table. Choices = *standard, vertmtdna, invertmtdna, yeastmtdna, plastid*, or any integer *1-31*.

##### `--randomize`

> **Optional**: For taxa with multiple sequences, shuffle order randomly. Overrides sorting by length for all methods specified using `-f`.

##### `--allseqs`

> **Optional**: For taxa with multiple sequences, select all sequences passing the filters instead of a single representative sequence.

##### `--no_subspecies`

> **Optional**: Ignore subspecies labels in both the taxon names file and the fasta file.

#### Example Uses:

```
python Filter_Seqs_and_Species.py -i /bin/Filter/ -f length -l 150 -t bin/Loci/Taxa_List.txt --no_subspecies
```
> Above command will select one sequence per taxon per locus based on the longest available sequence. The minimum base pair length is 150. The subspecies component of taxon names is ignored. Action is performed for all fasta files located in the `Filter/` directory.

```
python Filter_Seqs_and_Species.py -i /bin/Filter/ -f translate --table standard -l 150 -t bin/Loci/Taxa_List.txt
```
> Above command will select one sequence per taxon per locus based on the longest translatable sequence. The standard code is used for translation for all input fasta files. The minimum base pair length is 150. Subspecies are included. Action is performed for all fasta files located in the `Filter/` directory.

```
python Filter_Seqs_and_Species.py -i /bin/Filter/ -f length -l 150 -t bin/Loci/Taxa_List.txt --no_subspecies --randomize 
```
> Above command will select one sequence per taxon per locus randomly. The minimum base pair length is 150. The subspecies component of taxon names is ignored. Action is performed for all fasta files located in the `Filter/` directory.

```
python Filter_Seqs_and_Species.py -i /bin/Filter/ -f length -l 150 -t bin/Loci/Taxa_List.txt --allseqs
```
> Above command will select **ALL** sequences per taxon per locus that pass the minimum base pair length of 150. Subspecies are included. Action is performed for all fasta files located in the `Filter/` directory.

```
python Filter_Seqs_and_Species.py -i /bin/Filter/ -f translate --table vertmtdna -l 150 -t bin/Loci/Taxa_List.txt --allseqs
```
> Above command will select **ALL** sequences per taxon per locus that pass translation (using vertebrate mitochondrial code) and the minimum base pair length of 150. Subspecies are included. Action is performed for all fasta files located in the `Filter/` directory.

Several outputs are created in the specified input directory (one for every input fasta file):

+ `[fasta name]_single_taxon.fasta`: An output fasta file which contains a single filtered sequence per taxon. Produced if the `--allseqs` flag **is not used**.

**OR**

+ `[fasta name]_all_seqs.fasta`: An output fasta file which contains all available sequences per taxon that passed the relevant filters. Produced if the `--allseqs` flag **is used**.

+ `[fasta name]_species_log.txt`: A summary file containing information for each taxon entry. An example of the contents is shown below. In addition to reporting the sequence accession number and length, the number of alternative sequences available is shown. If the `-f translate` method was used, the results of whether the translation test was passed is provided (Y or N). If the `-f length` method was used, this column will contain NA instead.
```
Taxon	Accession	SeqLength	PassedTranslation	SeqsAvailable
Agama lionotus	GQ242168.1	853	Y	1
Amblyrhynchus cristatus	NC_028031.1	1038	Y	21
Amphibolurus muricatus	HQ684202.1	1032	Y	66
Amphibolurus norrisi	AY133001.1	1032	Y	18
Anisolepis longicauda	AF528736.1	1033	Y	1
Anolis acutus	AF055926.2	1029	Y	1
Anolis aeneus	AF055950.1	1033	Y	2
...
```

+ `[fasta name]_accession_list_by_species.txt`: A tab-delimited file in which each line starts with a taxon name and is followed by all accession numbers of sequences passing the length filter from the fasta file. These are the accession numbers for the selected sequence and all alternative sequences indicated in the file above. An example of the contents is shown below:
```
Agama lionotus	GQ242168.1	
Amblyrhynchus cristatus	NC_028031.1	KT277937.1	KR350765.1	KR350766.1	KR350762.1	KR350754.1	KR350759.1	KR350763.1	KR350764.1	KR350758.1	KR350755.1	KR350761.1	KR350753.1	KR350757.1	KR350752.1	KR350760.1	KR350756.1	KR350743.1	KR350746.1	KR350744.1	KR350745.1	
Amphibolurus muricatus	HQ684202.1	HQ684207.1	HQ684201.1	AF128468.1	HQ684203.1	HQ684199.1	HQ684206.1	HQ684204.1	HQ684205.1	KF871666.1	KF871689.1	KF871681.1	KF871658.1	KF871665.1	KF871679.1	KF871692.1	KF871706.1	KF871655.1	KF871688.1	KF871659.1	KF871656.1	KF871653.1	KF871670.1	KF871703.1	KF871699.1	KF871691.1	KF871685.1	KF871701.1	KF871669.1	KF871675.1	KF871705.1	KF871704.1	KF871696.1	KF871700.1	KF871671.1	KF871676.1	KF871702.1	KF871672.1	KF871660.1	KF871668.1	KF871697.1	KF871683.1	KF871667.1	KF871677.1	KF871687.1	KF871698.1	KF871686.1	KF871678.1	KF871664.1	KF871694.1	KF871651.1	KF871684.1	KF871662.1	HQ684200.1	KF871654.1	KF871661.1	KF871680.1	KF871693.1	KF871663.1	KF871673.1	KF871690.1	KF871652.1	KF871650.1	KF871657.1	KF871682.1	KF871695.1	
Amphibolurus norrisi	AY133001.1	HQ684197.1	HQ684208.1	HQ684198.1	HQ684211.1	HQ684210.1	HQ684195.1	HQ684194.1	HQ684188.1	HQ684196.1	HQ684192.1	HQ684191.1	HQ684190.1	HQ684189.1	HQ684209.1	HQ684193.1	KF871674.1	HQ684187.1	
Anisolepis longicauda	AF528736.1	
Anolis acutus	AF055926.2	
Anolis aeneus	AF055950.1	AF317066.1	
...
```

+ `[fasta name]_accession_list_for_Batch_Entrez.txt`: a Batch Entrez style file which contains all the accession numbers for sequences that passed the length filter. This list is a combination of all the accession numbers in the above file. It can be used to download all records using the Batch Entrez portal on NCBI. An example of the contents is shown below:
```
GQ242168.1
NC_028031.1
KT277937.1
KR350765.1
KR350766.1
KR350762.1
KR350754.1
KR350759.1
KR350763.1
KR350764.1
KR350758.1
...
```

These output files provide explicit information regarding which sequence has been selected for all taxa and loci. In addition, they provide accession numbers that can be used to re-create the fasta files used.

---------------

### Make_Acc_Table.py <a name="MAT"></a>

The goal of `Make_Acc_Table.py` is to create a table of accession numbers for each taxon and locus from a directory of fasta files, where each fasta file represents a different locus. These files must have been previously screened such that each taxon is only represented by a single sequence per locus. 

A complete set of taxa is inferred from all the fasta files. By default the names constructed are binomial (genus and species). If subspecies names are also desired, the optional `-s ` flag must be used and a text file of subspecies names must be supplied. The same taxon list from previous steps can be used here, but only the subspecies names are extracted from this list. If a record matches an included subspecies label then the subspecies name will be used, otherwise the binomial name will be used. 

For every taxon, the accession numbers are obtained from each locus. If the taxon is not present in a fasta file (locus), it will appear in the table as a dash. The names of the columns will match the fasta file names, and the rows are composed of taxon names in alphabetical order.

**Note**: This script can process unaligned or aligned fasta files, but they must have the original description lines (not relabeled), and taxa must only contain a single sequence per locus.

#### Basic Usage:

```
python Make_Acc_Table.py -i <input directory>
```

#### Argument Explanations:

##### `-i <path-to-directory>`

> **Required**: The full path to a directory containing the fasta files (single sequence per taxon). Fasta files in the directory must have extensions '.fasta' or '.fa' to be read.

##### `-s <path-to-file>`

> **Optional**: The full path to a text file containing all subspecies names to cross-reference in the fasta file. This can be a taxon names file used in previous steps, or a smaller version only containing subspecies names.

#### Example Uses:

```
python Make_Acc_Table.py -i /bin/filtered_fasta/ -s /bin/taxa/taxon_list.txt
```
> Above command will construct a table of accession numbers based on the fasta files present in the `filtered_fasta/` directory. Subspecies labels from `taxon_list.txt` will be used to find subspecies names in the fasta files.

The accession table will be written as a tab-delimited text file to the input directory as `GenBank_Accession_Table.txt`. Here is an example of the contents from a truncated file:

```
Taxon	12S	16S	CO1	CYTB	ND1
Acanthocercus adramitanus	-	KU097508.1	-	-	-
Acanthocercus annectens	-	MG700133.1	MG699914.1	-	-
Acanthocercus atricollis	-	JX668132.1	-	-	-
Acanthocercus cyanogaster	-	JX668138.1	-	-	-
Acanthocercus yemensis	-	JX668140.1	-	-	-
Acanthosaura armata	NC_014175.1	NC_014175.1	NC_014175.1	NC_014175.1	NC_014175.1
Acanthosaura capra	-	-	-	AY572880.1	-
Acanthosaura crucigera	AB031963.1	MG935713.1	MG935416.1	AY572889.1	-
Acanthosaura lepidogaster	KR092427.1	KR092427.1	KR092427.1	KR092427.1	KR092427.1
Agama aculeata	-	JX668143.1	-	AF355563.1	-
...
```

This file can be opened and manipulated using other applications such as Excel. By default, the columns are sorted in alphabetical order according to the fasta names.


---------------

### Infer_Supermatrix_Combinations.py <a name="ISC"></a>

The goal of `Infer_Supermatrix_Combinations.py` is to calculate how many supermatrix combinations are available, given the number of filtered sequences available for each taxon for each locus. If all taxa have only one sequence available, the answer is one, but if taxa have multiple sequences available, this number will be extremely large. This module relies on the `[locus]_species_log.txt` files produced from the `Filter_Seqs_and_Species.py` module to calculate the number of sequences available per taxon. The log files for all loci should be present in the input directory for the calculation to be accurate.

No output files are created, rather the information is logged to the screen. This includes the total number of sequences available (for all taxa across all loci), the total number of taxa, and the total number of possible supermatrix combinations.

#### Basic Usage:

```
python Infer_Supermatrix_Combinations.py -i <input directory>
```

#### Argument Explanations:

##### `-i <path-to-directory>`

> **Required**: The full path to a directory which contains all the `[locus]_species_log.txt` files.

#### Example Uses:

```
python Infer_Supermatrix_Combinations.py -i /bin/filtered_files/
```
> Above command will calculate the total number of supermatrix combinations based on the  `[locus]_species_log.txt` files present in the `filtered_files/` directory.

Example output on screen:

```
Found 4 loci to examine.


	Parsing information in ITS_extracted_species_log.txt
	Parsing information in MATK_extracted_species_log.txt
	Parsing information in RBCL_extracted_species_log.txt
	Parsing information in TRNL-TRNF_extracted_species_log.txt


Found 4,098 total sequences for 651 taxa.


Number of possible supermatrix combinations (unwieldy integer) = 23,585,393,330,101,509,977,748,274,541,526,924,398,183,623,226,243,709,268,339,387,872,606,971,131,321,031,032,654,793,314,825,517,791,632,690,451,488,529,178,270,403,709,016,922,288,924,652,228,738,591,235,421,706,300,132,306,309,071,552,868,337,445,042,676,361,861,581,482,286,906,518,734,986,582,600,423,724,641,187,945,561,979,141,257,183,422,301,145,122,812,952,763,816,817,181,582,943,498,141,376,096,275,064,842,431,024,332,800,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000.


Number of possible supermatrix combinations (scientific notation) = 2.36E+391, or 2.36*10^391.


```

That's a lot of possible supermatrices! Good thing **SuperCRUNCH** has an objective, repeatable method for selecting sequences.

---------------

---------------

## **Sequence Alignment** <a name="SA"></a>

![F4](https://github.com/dportik/SuperCRUNCH/blob/master/docs/Fig4.jpg)

**SuperCRUNCH** includes two pre-alignment steps and several options for multiple sequence alignment. One pre-alignment step (`Adjust_Direction.py`) adjusts the direction of sequences and produces unaligned fasta files with all sequences written in the correct orientation. This step is necessary to avoid major pitfalls with aligners. Loci can be aligned using `Align.py` with **MAFFT**, **MUSCLE**, **Clustal-O**, or all these aligners sequentially. For coding loci, the **MACSE** translation aligner is also available, which is capable of aligning coding sequences with respect to their translation while allowing for multiple frameshifts or stop codons. To use this aligner the `Coding_Translation_Tests.py` module should be used to identify the correct reading frame of sequences, adjust them to the first codon position, and ensure completion of the final codon. Although MACSE can be run on a single set of reliable sequences (e.g., only those that passed translation), it has an additional feature allowing the simultaneous alignment of a set of reliable sequences and a set of unreliable sequences (e.g., those that failed translation) using different parameters. The `Coding_Translation_Tests.py` module can be used to generate all the necessary input files to perform this type of simultaneous alignment using **MACSE**.


---------------

### Adjust_Direction.py <a name="AD"></a>

The purpose of `Adjust_Direction.py` is to check sequences to ensure their proper direction before performing alignments with `Align.py` or additional filtering using `Coding_Translation_Tests.py`. `Adjust_Direction.py` is designed to work for a directory of fasta files, and it will adjust sequences in all unaligned fasta files and output unaligned fasta files with all sequences in the 'correct' direction. `Adjust_Direction.py` uses **MAFFT** to perform adjustments, and the default setting uses the *--adjustdirection* implementation of mafft. If the optional `--accurate` flag is included, it will use the *--adjustdirectionaccurately* option, which is slower but more effective with divergent sequences. 

The standard output file from **MAFFT** is an interleaved fasta file containing aligned sequences written in lowercase. In this file, sequences that have been reversed are flagged by writing an `_R_` at the beginning of the record ID. `Adjust_Direction.py` takes this output file and converts it to a cleaner format. The alignment is stripped, sequences are re-written sequentially and in uppercase, and the `_R_` is removed. Sequences that were reversed are recorded in a locus-specific log file, and the total counts of reversed sequences across all loci is written to a general log file, described below.

The resulting unaligned fasta files can be used for `Coding_Translation_Tests.py` or for `Align.py`.

#### Basic Usage:

```
python Adjust_Direction.py -i <input directory>
```

#### Argument Explanations:

##### `-i <path-to-directory>`

> **Required**: The full path to a directory which contains the unaligned fasta files. Fasta files in the directory must have extensions '.fasta' or '.fa' to be read.

##### `--accurate`

> **Optional**: Use the --adjustdirectionaccurately implementation in MAFFT, rather than --adjustdirection. It is slower but more accurate, especially for divergent sequences.

#### Example Uses:

```
python Adjust_Direction.py -i /bin/Adjust/
```
> Above command will adjust all unaligned fasta files in directory `Adjust/` using --adjustdirection in MAFFT.

```
python Adjust_Direction.py -i /bin/Adjust/ --accurate
```
> Above command will adjust all unaligned fasta files in directory `Adjust/` using --adjustdirectionaccurately in MAFFT.

Two outputs are created in the specified input directory for each fasta file, including:

+ `[fasta name]_Adjusted_Name_Log.txt`: Contains the full names of the sequences that were reversed in this particular fasta file, if any were adjusted.
+ `[fasta name]_Adjusted.fasta`: An unaligned fasta file which contains the correctly oriented sequences.

An additional output file is created:

+ `Log_Sequences_Adjusted.txt`: Contains the names of all fasta files and the number of sequences that were in the correct orientation or had to be reversed. Example contents:
```
Locus	Seqs_Correct_Direction	Seqs_Direction_Adjusted
12S	529	1
16S	565	13
CO1	484	0
CYTB	513	0
ND1	202	0
```

---------------

### Coding_Translation_Tests.py <a name="CTT"></a>

`Coding_Translation_Tests.py` can be used to identify translatable sequences from unaligned fasta files. This module performs the following tasks:
+ Sequences are translated in all forward frames to check for the presence of stop codons. 
    + If no stop codons are found for a forward frame, this frame is selected and the sequence passes translation.
    + If one stop codon is found and it is present in the final two codon positions of the sequence, this frame is selected and the sequence passes translation.
    + Sequences that have more than one stop codon in all frames fail the translation test. 
+ If a correct frame is identified, the sequence is adjusted so that the first base represents the first codon position. If no correct frame is found, the sequence is not adjusted.
+ For all sequences (pass and fail), the sequence length is adjusted with N's to ensure the final codon is complete. In other words, the total sequence length will be divisible by three.
+ Sequences are written as described above to corresponding output files. There are three output files written per input fasta file, described below.

If the `--rc` flag is included the translation will also be performed for the reverse complement, however if your sequences are all correctly oriented this is not recommended. The translation table should be specified with the `--table` flag. All NCBI translation table options are available, and can be selected using integers (*1-31*) or the shortcut terms provided (*standard, vertmtdna, invertmtdna, yeastmtdna, plastid*). If the `--table` flag is omitted, the default will be to use the Standard translation table. 

The `Coding_Translation_Tests.py` module can be used to determine if sequences are translatable, adjust sequences to the first codon position, and adjust sequence lengths to be divisible by three. Although these outputs were intended to be used for multiple sequence alignment with **MACSE**, the tests and outputs of `Coding_Translation_Tests.py` are likely to be useful for other purposes as well. 

#### Basic Usage:

```
python Coding_Translation_Tests.py -i <input directory> --table <translation table>
```

##### `-i <path-to-directory> `

> **Required**: The full path to a directory which contains the locus-specific fasta files to filter. Fasta files in the directory must have extensions '.fasta' or '.fa' to be read.

##### `--table <choice>`

> **Required**: Specifies translation table. Choices = *standard, vertmtdna, invertmtdna, yeastmtdna, plastid*, or any integer *1-31*. Table will be used for all files found in the input directory.

##### `--rc`

> **Optional**: In addition to forward frames, use reverse complement for translation tests. Not recommended if direction of sequences has already been adjusted.

#### Example Uses:

```
python Adjust_Direction.py -i /bin/Translate/ --table vertmtdna
```
> Above command will perform translation tests for each unaligned fasta file in the directory `Translate/` using the vertebrate mitochondrial code.


An output directory called `Output_Translation_Fasta_Files/` is created in the input directory. For each fasta file included, the following output files are created:

+ `[Fasta name]_All.fasta`: Contains all length and/or position adjusted sequences, pass and fail.
+ `[Fasta name]_Passed.fasta`: Contains all length and position adjusted sequences that passed translation.
+ `[Fasta name]_Failed.fasta`: Contains all length adjusted sequences that failed translation.

An additional output file is created:

+ `Log_Sequences_Filtered.txt`:  A summary log file which indicates how many sequences passed translation and failed translation for each fasta file processed. Example contents:
```
Locus	Seqs_Passed	Seqs_Failed
CO1	479	5
CYTB	507	6
ND1	192	10
ND2	1005	0
ND4	548	3
```

---------------

### Align.py <a name="A"></a>

`Align.py` can be used to perform multiple sequence alignment for a directory of unaligned fasta files using ***MAFFT***, ***MUSCLE***, ***Clustal-O***, and/or ***MACSE***. 

The aligner is specified using the `-a` flag:
+ `-a mafft`: Align using ***MAFFT***.
+ `-a muscle`: Align using ***MUSCLE***.
+ `-a clustalo`: Align using ***Clustal-O***.
+ `-a all`: Align using ***MAFFT***, ***MUSCLE***, and ***Clustal-O*** (sequentially).
+ `-a macse`: Translation align using ***MACSE***. Details are provided below.

**NOTE:** The aligners can be run simultaneously on the same directory of unaligned fasta files without interfering with one another. This can speed up the alignment process if multiple aligners are being used, and is an alternative to the `-a all` option.

***MAFFT***, ***MUSCLE***, and ***Clustal-O*** can be used for all loci (coding and non-coding). The usage of each aligner invokes default settings or auto selection of the alignment strategy. For example: `mafft --auto` and `clustalo --auto`. These settings should be useful for a majority of alignments. The `--accurate` flag can be used to change the settings for ***MAFFT*** and ***Clustal-O*** to the following:
+  For ***Clustal-O***, this de-selects the --auto option and enables full distance matrix for guide-tree calculation, full distance matrix for guide-tree calculation during iteration, and --iter=5, in which the guide-tree and HMM each undergo 5 iterations, rather than only one: `clustalo --full --full-iter --iter=8`.
+ For ***MAFFT***, this option changes the default from auto select to use the FFT-NS-i strategy: `mafft --retree 2 --maxiterate 1000`.

The improved accuracy using these settings comes at the cost of longer run times for each aligner. However, this may be desirable for divergent sequences or difficult alignments.

***MACSE*** should only be used for coding loci, as it is a translation aligner. The use of `-a macse` requires using additional arguments and other optional arguments are available. The `--mpath` flag must be used to supply the full path to the ***MACSE*** JAR file (ideally V2). The `--table` flag can be used to specify the translation table using a shortcut term or an integer (*standard, vertmtdna, invertmtdna, yeastmtdna, plastid, 1-6, 9-16, 21-23*). Note that unlike previous steps, not all tables are available in ***MACSE***. Unless specified, the default table used is the Standard code. The optional `--mem` flag can be used to assign an amount of memory (in GB). The `--accurate` can also be used for ***MACSE*** v2, which invokes `-local_realign_init 0.9 -local_realign_dec 0.9`. The default for these arguments is normally 0.5., and changing this will slow down optimizations but increase alignment accuracy (sometimes considerably). These search settings will more closely resemble ***MACSE*** v1, which had default values of 1.0 for both.

An additional feature of ***MACSE*** is to include a set of reliable sequences (for example those that passed translation) and a set of less reliable sequences that are suspected to contain errors, and align both simultaneously with different penalty parameters. To use this feature in ***MACSE*** the `--pass_fail` flag can be used. However, to work the sequence sets must be contained in two fasta files that follow this naming format:

+ `[prefix]_Passed.fasta`: Fasta file of reliable sequences.
+ `[prefix]_Failed.fasta`: Fasta file of unreliable sequences.

These outputs are produced by default in the `Coding_Translation_Tests.py` module. The prefix portion of the name should ideally be the abbreviation of the gene/locus. If one file is missing, the `--pass_fail` will not work. 

Because ***MACSE*** can deal with frameshifts and sequence errors, it will insert an `!` character at corrected bp locations in the output alignment. A cleaned fasta file is created after the alignment is completed, in which all instances of `!` are replaced by `N`.

Output files vary between aligners but will be moved to output directories created in the main input directory, with details below. 


#### Basic Usage:

```
python Align.py -i <input directory> -a <aligner> 
```

##### `-i <path-to-directory>`

> **Required**: The full path to a directory which contains the unaligned fasta files. Fasta files in the directory must have extensions '.fasta' or '.fa' to be read.

##### `-a <choice>`

> **Required**: Specify whether alignment is by mafft, macse, muscle, or clustalo. If macse, MUST provide flags --mpath and --table. Choices = *mafft, macse, muscle, clustalo, all*.

##### `--mpath <path-to-executable>`

> **Required** for `-a macse`: Full path to a macse jar file (ideally MACSE v2).

##### `--table <choice>`

> **Required** for `-a macse`: Specifies translation table. Choices = *standard, vertmtdna, invertmtdna, yeastmtdna, plastid, 1-6, 9-16, 21-23*.

##### `--mem <integer>`

> **Optional** for `-a macse`: An integer for how much memory to assign to macse (in GB). Default = 1.

##### `--pass_fail`

> **Optional** for `-a macse`: Specifies macse to perform dual alignment. Files in -i directory must follow labeling format: NAME_Passed.fasta, NAME_Failed.fasta.

##### `--accurate`

> **Optional**: Specifies more thorough search settings (for mafft, clustalo, or macse).

#### Example Uses:

```
python Align.py -i /bin/to_align/ -a mafft
```
> Above command will align all fasta files in the `to_align/` directory using mafft with the --auto strategy.

```
python Align.py -i /bin/to_align/ -a mafft --accurate
```
> Above command will align all fasta files in the `to_align/` directory using mafft with the FFT-NS-i strategy.

```
python Align.py -i /bin/to_align/ -a all --accurate
```
> Above command will align all fasta files in the `to_align/` directory using clustalo, muscle and mafft. The `--accurate` flag changes settings using clustalo and mafft only.

```
python Align.py -i /bin/coding_loci/ -a macse --mpath /bin/programs/macse_v2.03.jar --table vertmtdna --mem 10 
```
> Above command will align all fasta files in the `coding_loci/` directory using the macse translation aligner (jar file `macse_v2.03.jar`) with the vertebrate mtdna table and 10GB of memory.

```
python Align.py -i /bin/coding_loci/ -a macse --mpath /bin/programs/macse_v2.03.jar --table vertmtdna --mem 10 --pass_fail
```
> Above command will align all paired fasta files (`[prefix]_Passed.fasta`, `[prefix]_Failed.fasta`) in the `coding_loci/` directory using the macse translation aligner (jar file `macse_v2.03.jar`) with the vertebrate mtdna table and 10GB of memory.

Depending on the alignment option selected, one or more of the directories will be created with the following contents:

+ **Output_CLUSTALO_Alignments/**
    + For each unaligned input fasta file, this directory contains a corresponding output alignment file labeled `[fasta name]_CLUSTALO_Aligned.fasta`. Results from `-a clustalo` and `-a all`.
+ **Output_MAFFT_Alignments/**
    + For each unaligned input fasta file, this directory contains a corresponding output alignment file labeled `[fasta name]_MAFFT_Aligned.fasta`. Results from `-a mafft` and `-a all`.
+ **Output_MUSCLE_Alignments/**
    + For each unaligned input fasta file, this directory contains a corresponding output alignment file labeled `[fasta name]_MUSCLE_Aligned.fasta`. Results from `-a muscle` and `-a all`.
+ **Output_MACSE_Alignments/**
    + For each unaligned input fasta file, this directory contains corresponding output files labeled `[fasta name]_AA.fasta`, `[fasta name]_NT.fasta`, and `[fasta name]_NT_Cleaned.fasta`. Results only from `-a macse`.

I ***strongly*** recommend using multiple aligners and comparing the results. This is arguably the most important step in creating phylogenetic data sets, and obtaining quality alignments is critical before performing any subsequent analyses.

---------------

---------------

## **Post-Alignment Tasks** <a name="FFT"></a>


![F5](https://github.com/dportik/SuperCRUNCH/blob/master/docs/Fig5.jpg)

After multiple sequence alignment is complete, the alignment files are ready for relabeling (`Relabel_Fasta.py`), trimming (`Trim_Alignments.py`), format conversion (`Fasta_Convert.py`), and concatenation (`Concatenation.py`). There are different options for relabeling, depending on whether population-level data are gathered or species-level data are gathered. The relabeling step is required for trimming, and trimming is always recommended. The relabeled and trimmed fasta files can be converted into nexus and phylip format, and for species-level data sets (one representative sequence per taxon) the alignments can be easily concatenated into a phylogenetic supermatrix.

---------------

### Relabel_Fasta.py <a name="RF"></a>

The goal of `Relabel_Fasta.py` is to relabel all sequence records in all fasta files contained in a directory. The fasta files can be aligned or unaligned, but should have the original sequence description lines present.

There are three relabeling strategies available, specified using the `-r` flag: 
+ `-r species`: A taxon name is constructed from the description line. This generally corresponds to the genus and species if records are labeled properly. Subspecies labels can also be included using the optional `-s` flag.
+ `-r accession`: The accession number will be used to label the sequence record.
+ `-r species_acc`. The record is labeled by taxon and accession number. Subspecies labels can also be included using the optional `-s` flag.

In all strategies, any spaces are replaced by underscores. If the optional `-s` flag is included with a text file containing subspecies (trinomial) names, then the taxon component of each of the above options will include the trinomial if present.

The most appropriate relabeling strategy will depend on the final goals. Relabeling with `-r species` is essential for concatenating alignments to create a phylogenetic supermatrix. Relabeling with `-r species_acc` is the best option for population-level data sets, in which each species is represented by multiple sequences. This way, taxon names are present and the trailing accession numbers distinguish sequences belonging to the same taxon.

For each file relabeled, a corresponding labeling key is produced. This tab-delimited file contains the accession number, taxon label, and description line for each record. This serves as a reference key, such that the relevant information can be tracked for all relabeled samples.

Examples of how each relabeling option works is shown in greater detail below.

#### Basic Usage:

```
python Relabel_Fasta.py -i <input directory> -r <relabel option>
```

##### `-i <path-to-directory>`

> **Required**: The full path to a directory containing the unaligned or aligned fasta files. Fasta files in the directory must have extensions '.fasta' or '.fa' to be read.

##### `-r <choice>`

> **Required**: The strategy for relabeling sequence records. Choices = *species, accession, species_acc*.

##### `-s <path-to-file>`

> **Optional**: The full path to a text file containing all subspecies names to cross-reference in the fasta file.


#### Example Uses:

```
python Relabel_Fasta.py -i bin/aligns_to_relabel/ -r species -s bin/subspecies_list.txt
```
> Above command will relabel description lines using the taxon name, and will use the appropriate subspecies labels from `subspecies_list.txt` if they are present in the records. Action is performed for all fasta files located in the `aligns_to_relabel/` directory.

```
python Relabel_Fasta.py -i bin/aligns_to_relabel/ -r acc 
```
> Above command will relabel description lines using the accession number. Action is performed for all fasta files located in the `aligns_to_relabel/` directory.

```
python Relabel_Fasta.py -i bin/aligns_to_relabel/ -r species_acc -s bin/subspecies_list.txt
```
> Above command will relabel description lines using the taxon name plus accession number, and will use the appropriate subspecies labels from `subspecies_list.txt` if they are present in the records. Action is performed for all fasta files located in the `aligns_to_relabel/` directory.

#### Relabeling details:

Here I show how the relabeling works, in greater detail.

Let's use the following set of description lines from a fasta file:
```
>JN881132.1 Daboia russelii activity-dependent neuroprotector (ADNP) gene, partial cds
>KU765220.1 Sceloporus undulatus voucher ADL182 activity-dependent neuroprotector (adnp) gene, partial cds
>DQ001790.1 Callisaurus draconoides carmenensis isolate RWM 1480 cytochrome b (cytb) gene
```
Notice that the third entry has a subspecies label, but the first two entries do not.

Using `-r species` would produce:
```
>Daboia_russelii
>Sceloporus_undulatus
>Callisaurus_draconoides
```
The subspecies label in the third entry is ignored.

Using `-r accession` would produce:
```
>JN881132.1
>KU765220.1
>DQ001790.1
```
Using `-r species_acc` would produce:
```
>Daboia_russelii_JN881132.1
>Sceloporus_undulatus_KU765220.1
>Callisaurus_draconoides_DQ001790.1
```
Again, the subspecies label in the third entry is ignored.

If I supplied a subspecies text file using the `-s` flag that contained `Callisaurus draconoides carmenensis`, then the following would be produced with `-r species`:
```
>Daboia_russelii
>Sceloporus_undulatus
>Callisaurus_draconoides_carmenensis
```
The subspecies label in the third entry is now included.

And this would be produced using `-r species_acc`:
```
>Daboia_russelii_JN881132.1
>Sceloporus_undulatus_KU765220.1
>Callisaurus_draconoides_carmenensis_DQ001790.1
```
The subspecies label in the third entry is now included.

Depending on the relabeling strategy selected, one of the directories will be created with the following contents:

+ **Relabeled_Fasta_Files_Species/**
    + For each locus, this directory contains a corresponding fasta file labeled `[fasta name]_relabeled.fasta` and `[fasta name]_label_key.txt`. Results from `-r species`.
+ **Relabeled_Fasta_Files_Accession/**
    + For each locus, this directory contains a corresponding fasta file labeled `[fasta name]_relabeled.fasta` and `[fasta name]_label_key.txt`. Results from `-r accession`.
+ **Relabeled_Fasta_Files_SpeciesAccession/**
    + For each locus, this directory contains a corresponding fasta file labeled `[fasta name]_relabeled.fasta` and `[fasta name]_label_key.txt`. Results from `-r species_acc`.

Example contents from a `[fasta name]_label_key.txt` file:
```
Taxon	Accession	Description
KU097508.1	Acanthocercus adramitanus	Acanthocercus adramitanus isolate IBES10359 16S ribosomal RNA gene, partial sequence; mitochondrial
MG700133.1	Acanthocercus annectens	Acanthocercus annectens voucher USNM:589391 16S ribosomal RNA gene, partial sequence; mitochondrial
JX668132.1	Acanthocercus atricollis	Acanthocercus atricollis voucher EBG 2167 16S ribosomal RNA gene, partial sequence; mitochondrial
JX668138.1	Acanthocercus cyanogaster	Acanthocercus cyanogaster voucher MVZ 257937 16S ribosomal RNA gene, partial sequence; mitochondrial
JX668140.1	Acanthocercus yemensis	Acanthocercus yemensis voucher MVZ 236454 16S ribosomal RNA gene, partial sequence; mitochondrial
NC_014175.1	Acanthosaura armata	Acanthosaura armata mitochondrion, complete genome
MG935713.1	Acanthosaura crucigera	Acanthosaura crucigera voucher USNM:587019 16S ribosomal RNA gene, partial sequence; mitochondrial
```

---------------

### Trim_Alignments.py <a name="TAS"></a>

The alignments may require some amount of trimming to remove overhanging ends, poorly aligned regions, etc. `Trim_Alignments.py` simplifies this process by automating trimming for a directory of input files. `Trim_Alignments.py` relies on **trimAl**, and three options are available for trimming. Specifying `-a gt` will use the gap threshold method, and although the default value is 0.05 this can be changed using the `--gt ` flag to set a value between 0 and 1. The gt value is the minimum fraction of sequences without a gap required to retain an alignment column. Specifying `-a noallgaps` uses the noallgaps method, which simply removes any alignment column composed entirely of gaps. Finally, specifying `-a both` runs the gap threshold method followed by the noallgaps method.

The input files can be in fasta, nexus, or phylip format, and the format is automatically detected by **trimAl**. The output format must be specified by the `-f ` flag, and includes fasta, nexus, or phylip format. Although any output format can be chosen, fasta format is recommended if the `Fasta_Convert.py` or `Concatenation.py` modules will be used.

**Note:** If the input files have not been relabeled at this point (ie they are aligned fasta files containing the original description lines), the identifier names appearing in the output files will default to accession numbers only. 

#### Basic Usage:

```
python Trim_Alignments.py -i <input directory> -f <output format> -a <trimal method>
```

##### `-i <path-to-directory>`

> **Required**: The full path to a directory which contains the input alignment files. File formats can include fasta, nexus, or phylip, with one of the corresponding file extensions: '.fasta', '.fa', '.nexus', '.nex', '.phylip', or '.phy'. 

##### `-f <choice>`

> **Required**: Specifies the output file format for trimmed alignments. Choices = *fasta, nexus, phylip*.

##### `-a <choice>`

> **Required**: Specifies the trimal method for trimming alignments. Choices = *gt, noallgaps, both*.

##### `--gt <value>`

> **Optional**: Specifies the gap threshold (gt) value for trimal, the minimum fraction of sequences without a gap. Must be between 0 and 1. Default = 0.05.


#### Example Uses:

```
python Trim_Alignments.py -i /bin/Trim/ -f fasta -a gt --gt 0.1
```
> Above command will trim all alignments present in the directory `Trim/` using the gap threshold method with a value of 0.1, and output files in fasta format.

```
python Trim_Alignments.py -i /bin/Trim/ -f phylip -a noallgaps
```
> Above command will trim alignments present in the directory `Trim/` using the noallgaps method and output files in phylip format.

```
python Trim_Alignments.py -i /bin/Trim/ -f nexus -a both --gt 0.08
```
> Above command will trim alignments present in the directory `Trim/` using the gap threshold method with a value of 0.08, and output files in fasta format.


---------------

### Fasta_Convert.py <a name="FC"></a>

The goal of `Fasta_Convert.py` is to convert a directory of aligned fasta files into both phylip and nexus formats. Note this should only be used after records have been renamed with species labels (format `genus_species` or `genus_species_subspecies`) or accession numbers, as the original NCBI description lines will cause severe issues with writing other file formats.


#### Basic Usage:

```
python Fasta_Convert.py -i <input directory>
```

##### `-i <path-to-directory>`

> **Required**: The full path to a directory which contains the ALIGNED fasta files. Fasta files in the directory must have extensions '.fasta' or '.fa' to be read.

#### Example Uses:

```
python Fasta_Convert.py -i /bin/relabeled_alignments/
```
> Above command will convert all fasta alignments in the directory `relabeled_alignments/` into nexus and phylip format.

Two output folders are created in the input directory containing the fasta files. The directory labels and their contents are described below:

+ **Output_Phylip_Files/**
    + For each locus, this directory contains a corresponding phylip file, labeled `[fasta name].phy`.
+ **Output_Nexus_Files/**
    + For each locus, this directory contains a corresponding nexus file, labeled `[fasta name].nex`.


---------------

### Concatenation.py <a name="C"></a>

The goal of using `Concatenation.py` is to combine multiple alignments into a single concatenated alignment. 

To work properly, sequence names should have been relabeled across all files and should take the format of `genus_species` or `genus_species_subspecies`. There cannot be any duplicate taxon labels within any of the alignment files, or an error will be thrown: `ValueError: Duplicate key [taxon label]`. The complete set of taxa is inferred from all the alignments, for each taxon the sequences from all loci are retrieved. If a taxon is absent from an alignment, a missing sequence is generated using the symbol selected with the `-s` flag (options: N, dash, or ? symbol). The input files can be in fasta or phylip format, specified using the `-f` flag. The output format must be specified as fasta or phylip using the `-o` flag. Taxa are written in alphabetical order in the concatenated alignment. The arrangement of sequences in the concatenated alignment corresponds to the alphabetical order of the names of the input fasta files. Besides producing the concatenated alignment, two additional outputs are produced and described below.
 
This script takes advantage of python dictionary structures to read alignments and store sequences. As a result, it works extremely fast and is more than capable of concatenating thousands of large alignment files. 


#### Basic Usage:

```
python Concatenation.py -i <input directory> -r <input format> -s <missing data symbol> -o <output format>
```

##### `-i <path-to-directory>`

> **Required**: The full path to a directory containing the aligned files. 

##### `-f <choice>`

> **Required**: The input file format of the alignments. Choices = *fasta, phylip*. The files must have one of the corresponding extensions to be read: '.fasta', '.fa', '.phylip', or '.phy'.

##### `-s <choice>`

> **Required**: A base pair symbol used to represent missing data sequences. Choices = *dash, N, ?*.

##### `-o <choice>`

> **Required**: The output file format for the final concatenated alignment. Choices = *fasta, phylip*.

#### Example Uses:

```
python Concatenation.py -i /bin/Final_Alignments/ -r fasta -s dash -o phylip
```
> Above command will concatenate all the fasta alignments in the directory `Final_Alignments/` and produce a phylip file. Missing data are represented with the `-` symbol.

```
python Concatenation.py -i /bin/Final_Alignments/ -r phylip -s ? -o fasta
```
> Above command will concatenate all the phylip alignments in the directory `Final_Alignments/` and produce a fasta file. Missing data are represented with the `?` symbol.

Three output files are created in the specified input directory, including:

+ `Concatenated_Alignment.fasta` or `Concatenated_Alignment.phylip`: The final concatenated alignment in the output format specified.
+ `Data_Partitions.txt`: Text file displaying the order in which loci were concatenated and their corresponding base pairs within the alignment.
+ `Taxa_Loci_Count.log`: A count of the number of sequences that were available for each taxon, in other words the number of alignments the taxon was found in. 


---------------

---------------


*Written by Daniel Portik*

*Last updated: January 2019*
