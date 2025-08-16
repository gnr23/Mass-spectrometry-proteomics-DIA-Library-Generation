# DIA Analysis Using OpenSwathWorkflow and MSstats (Galaxy Tutorial)

This repository contains my hands-on learning experience on DIA analysis with DIA library generation, processing with OpenSwathWorkflow and analysis with MSstats

## Overview

End-to-end workflow of **Data Independent Acquisition (DIA) proteomics analysis**  covering:

-   **Spectral library generation** from DDA data (this repo)
    
-   **DIA analysis** with OpenSwathWorkflow and PyProphet (https://github.com/gnr23/Mass-spectrometry-proteomics-DIA-Analysis-using-OpenSwathWorkflow-.git)
    
-   **Statistical analysis** of DIA results using MSstats 
    

The dataset used is a spike-in benchmark, where defined amounts of _E. coli_ proteins are mixed into a human HEK293 cell background.

----------

## Objectives

-   Understand how a **spectral library** is generated from DDA data for DIA use
    
-   Learn how to perform **peptide/protein identification and quantification** with OpenSwathWorkflow
    
-   Apply **statistical models (PyProphet & MSstats)** to control FDR and analyze protein-level differences
    
-   Export and interpret **quantitative results** for downstream biological insights
    

----------

## Snapshot


1

**Library Generation**: Build a spectral library from DDA search results (using tools like MaxQuant, diapysef) and generate PQP/iRT assays

2

**DIA Analysis**: Process mzML files with OpenSwathWorkflow and PyProphet (scoring, FDR estimation, merging runs)

3

**Statistical Analysis**: Perform protein-level quantification and statistical testing using MSstats in Galaxy

----------

## Why It Matters

DIA methods such as **SWATH-MS** overcome the limitations of DDA by collecting comprehensive, unbiased data across the entire mass range. This provides:

-   **Reproducible quantification** across large experiments
    
-   **Better proteome coverage** without stochastic precursor selection
    
-   **Robust statistical inference** when paired with MSstats
    

----------

## My Notes and Outputs

I’ve documented parameter choices, exported results, and workflow insights in:

-   `results.md`: Final outputs (protein quant tables, statistical results, and intermediate files)
    
-   `README.md`: This summary of the exercise
    

----------

## Next Steps

-   Adapt workflow to **different spectral libraries or datasets**
    

----------

## References

-   Röst et al. (2014): _OpenSWATH enables automated, targeted analysis of data-independent acquisition MS data_
