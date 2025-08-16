# Library Generation for DIA Analysis

## **Questions:**

   How does the peptide identification work in data independent acquisition (DIA) mass spectrometry (MS) data?
    
-   What is a spectral library and how to generate a spectral library using Maxquant, diapysef and OpenSwath tools?
    

# **Objectives:**

   Generating a spectral library from data dependent acquisition (DDA) MS data
    
-   Understanding DIA data principles and characteristics
    
-   Optimizing and refining a spectral library for the analysis of DIA data

# **Bioinformatics tools used:**

MaxQuant, OpenSwathAssayGenerator, OpenSwathDecoyGenerator, TargetedFileConverter etc.

# Background

For complex protein mixtures, bottom-up mass spectrometry is the standard approach. In bottom-up proteomics, proteins are digested with a specific protease into peptides and the measured peptides are in silico reassembled into the corresponding proteins.

To enable the identification and quantification of such high numbers of proteins in a time-wise limited measurement there has been a lot of MS method and data acquisition optimization.

**Data dependent analysis** **(DDA)** as its name calls is highly dependent on the abundance of the respective peptides at a given retention time. This dependency on both time as well as intensity leads to measurement specific identifications between multiple measurements.

Another more recently developed methodology the so-called **data independent acquisition (DIA)**. However due to some unique characteristics of this acquisition method it requires spectral libraries, which contain the peptide sequences as well as their respective fragment spectrum (MS2) information.

One of the most popular proteomics software is **MaxQuant** because it is easy to use and a free software that offers functionalities for nearly all kinds of proteomics data analysis challenges . Mass spectrometry raw data is normally obtained in a vendor specific, proprietary file format. MaxQuant can directly take those raw files as input. For peptide identification MaxQuant uses a search engine called “**Andromeda**”.

![intro1](images/intro1.png)

# Agenda

 - Get data 
 - MaxQuant Analysis of DDA 
 -data Quality control results 
 -Filter for unique peptides in the evidence and msms output 
 - diapysef library generation 
 - Spectral library refinement with **OpenSwathAssayGenerator**
  
 - Adding decoy sequences with **OpenSwathDecoyGenerator**     
 - Converting the spectral library with **TargetedFileConverter** Conclusion

# Get data

Create a new history in galaxy project and import the fasta and raw files as well as the experimental annotation and the iRTassays file from [Zenodo](https://zenodo.org/record/4293493)

```
https://zenodo.org/record/4293493/files/Human_database_plus_iRT.fasta
https://zenodo.org/record/4293493/files/Ecoli_database.fasta
https://zenodo.org/record/4293493/files/iRTassays.tsv
https://zenodo.org/record/4293493/files/HEK_Ecoli_exp_design.txt
https://zenodo.org/record/4293493/files/Sample1.raw
https://zenodo.org/record/4293493/files/Sample2.raw
https://zenodo.org/record/4293493/files/Sample3.raw
https://zenodo.org/record/4293493/files/Sample4.raw
https://zenodo.org/record/4293493/files/Sample5.raw
```
![getdata1](/images/get_Data1.png)

Once the files are green, generate a collection for all .raw files (and name it DDA_data) as well as for both .fasta files (and name it FASTA)

# **MaxQuant**  Analysis of DDA data

1.  **MaxQuant**  (  Galaxy version 2.0.3.0+galaxy0)  with the following parameters:
    -   In  _“Input Options”_:
        -   param-collection  _“FASTA files”_:  `FASTA`  collection
        -   _“identifier parse rule”_:  `>([^ ]*)`
        -   _“description parse rule”_:  `^>.*\|.*\|[^ ]+ (.*) OS.*$`
    -   In  _“Search Options”_:
        -   param-file  _“Specify an experimental design template”_:  `HEK_Ecoli_exp_design.txt`
        -   _“minimum unique peptides”_:  `1`
        -   _“Match between runs”_:  `Yes`
    -   In  _“Protein quantification”_:
        -   _“Use only unmodified peptides”_:  `No`
    -   In  _“Parameter Group”_:
        -   param-collection  _“Infiles”_:  `DDA_data`  collection
            -   _“missed cleavages”_:  `1`
            -   _“variable modifications”_:  `Oxidation (M)`
    -   _“Generate PTXQC (proteomics quality control pipeline) report? (experimental setting)”_:  `True`
    -   In  _“Output Options”_:
        -   _“Select the desired outputs.”_:  `Protein Groups`  `Peptides`  `mqpar.xml`  `Evidence`  `MSMS`;
        
        ![maxquant1](/images/maxquant_1.png)
 ![maxquant2](/images/maxquant_2.png)
  ![ptxqc](/images/ptxqc.png)
        
       **comments**:  
       1. Proteins that share all their peptides with other proteins cannot be unambiguously identified. Therefore, MaxQuant groups such proteins into one protein group and only one common quantification will be calculated. The different protein properties are separated by semicolon. 
       2. _“identifier parse rule”_  allows for the organism label to be present in the spectral library. For investigations of single organisms and to keep only the Uniprot identifier one can adjust the  _“identifier parse rule”_  accordingly. 
       3. The PTXQC softwarewas built to enable direct proteomics quality control from MaxQuant result files.
       4.   Approximately 5,370 proteins were found (from ProteinGroups output)
       5. Approximately 34,970 peptides were found (from Peptides output)


# **Filter** for unique peptides in the _evidence_ and _msms_ output

Filtering the  **MaxQuant**  search results for unique peptides[]

1.  **Filter**  with the following parameters: -  param-file  _“Infile”_:  `MaxQuant_Evidence`
    -   _“With following condition”_:  `len(c9.split(';')) < 2`
    -   _“Number of header lines to skip”_:  `1`
2.  **Rename**  galaxy-pencil  the output file to ‘Filter on MaxQuant_Evidence’
    
    > Tip: Renaming a dataset[](https://training.galaxyproject.org/training-material/topics/proteomics/tutorials/DIA_lib_OSW/tutorial.html#tip-renaming-a-dataset-1)
    
3.  **Filter**  with the following parameters: -  param-file  _“Infile”_:  `MaxQuant_MSMS`
    -   _“With following condition”_:  `len(c12.split(';')) < 2`
    -   _“Number of header lines to skip”_:  `1`
4.  **Rename**  galaxy-pencil  the output file to ‘Filter on MaxQuant_MSMS’

![maxquant1](/images/filter_1.png)
![maxquant1](/images/filter_2.png)


**comment** 
1. To avoid ambigious protein mapping later, we filter the **MaxQuant** search results for unique peptides only
2.  ~103,000 lines in the evidence and ~98,000 lines in the msms.

# *diapysef* library generation

Generation of a spectral library using the unique  **MaxQuant**  search results and indexed retention time  _iRT_  peptides
1.  ****diapysef library generation**  generates spectral library for DIA analysis**  (  Galaxy version 0.3.5.0):
    -   In  _“MaxQuant output file msms.txt_:
        -   param-file  _“Infile”_:  `Filter on MaxQuant_MSMS`
    -   In  _“MaxQuant output file evidence.txt_:
        -   param-file  _“Infile”_:  `Filter on MaxQuant_Evidence`
    -   In  _“Retention time alignment method”_:
        -   param-file  _“Infile”_:  `iRTassays.tsv`
![diapysef](/images/diapysef_1.png)

**notes** : 
-to allow for improved alignment, synthetic (non-endogenous) peptides were spiked-in to all samples prior to the MS measurement. 
-First the measured retention times (RTs) of those spiked-in peptides are extracted in the **MaxQuant** search results. Using the measured RTs as well as arbitrary assigned values (ranging from -26 to 99) a linear regression through the 11 synthetic peptides is applied. 
-Based on this linear regression fit _indexed_ retention times (iRTs) are assigned to all other peptides.
-8 iRT peptides were found in the first DDA file and 9, 8, 10, 10 in the consecutive 4 DDA files.

# Spectral library refinement with  **OpenSwathAssayGenerator**


Spectral library optimization and refinement using  **OpenSwathAssayGenerator**
1.  **OpenSwathAssayGenerator**  (  Galaxy version 2.6+galaxy0)  with the following parameters:
    -   _“Output file type – default: determined from file extension or content”_:  `tabular (tsv)`
    -   _“MZ threshold in Thomson for precursor ion selection”_:  `0.015`
    -   _“upper MZ limit for precursor ions”_:  `1000.0`
    -   _“MZ threshold in Thomson for fragment ion annotation”_:  `0.015`

![OpenSwathAssayGenerator](/images/OpenSwathAssayGenerator.png)
**notes** : 

- The spectral library coming from **diapysef library generation** contains all observed fragment ions from the DDA runs resulting in a relativly large spectral library. 
- Large libraries can lead to increased processing times as well as fewer identifications after adjusting based on the False Discovery Rate (FDR). Thus, it is recommened to optimize and refine the spectral library by e.g. filtering for peptides with at least 6 transitions (increased confidence) and limiting the maximum also to 6 transitions (avoiding inflated libraries).
- Furthermore, the scan range can be adjusted (here between 400 - 1000 m/z) covering the same m/z range as in the DIA measurements.
- The refined library contains ~178,900 lines around 700,000 lines were removed.

# Adding decoy sequences with  **OpenSwathDecoyGenerator**

Adding  _decoy_  transitions to the spectral library

1.  **OpenSwathDecoyGenerator**  (  Galaxy version 2.6+galaxy0)  with the following parameters:
    -   _“Output file type – default: determined from file extension or content”_:  `tabular (tsv)`
    -   _“Advanced Options”_:  `Show Advanced Options`
        -   _“MZ threshold in Thomson for fragment ion annotation”_:  `0.015`
        
![OpenSwathdecoy](/images/OpenSwathDecoyGenerator.png)


**notes** : 

-To enable correct false discovery rate (FDR) computation later on, we add computanionally generated decoy sequences to the spectral library. Those “non-observed” sequences can be generated based on the observed sequences with slight modifications. The most commonly used methods are either **shuffle** (randomly altering the amino acid sequence of each observed transition) or **reverse** (by reversing the actually obeserved transitions). Those artificially generated transitions were labelled as **decoy** and are later on considered as known false positives. Example: By applying an FDR of 1 % we only allow for e.g only 1 such decoy transition out of 100 identifications. Thus one could estimate that the remaining 99 non-labeled identifications contain also 1 % false positive hits.

# Converting the spectral library with  **TargetedFileConverter**

Converting the final spectral library from  _.tsv_  to the sqlite  _.pqp_  format

1.  **TargetedFileConverter**  (  Galaxy version 2.6+galaxy0)  with the following parameters:
    -   _“Output file type – default: determined from file extension or content”_:  `pqp`

we choose .pqp file format since we will use later on **OpenSwathWorkflow** taht needs to have the spectral library in _.pqp_ format to being able to combine multiple runs after the DIA analysis and before applying the FDR scoring