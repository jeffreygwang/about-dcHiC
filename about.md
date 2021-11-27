# About dcHiC

> Note: This is VERY incomplete. 

This page briefly rehashes some of my research work on dcHiC, an application that combines uses machine learning, statistical modeling, and data visualization to identify significant changes in the genome's 3D structure.
This work was done with the Ay Lab at the La Jolla Institute for Immunology; the application is released open-source (ay-lab: dcHiC) and published (first author). 

## The Classic "3D Genome" Background (skip if you are in the field)

Since the 1970s, the scientific community has known that the genome folds in a highly-structured, 3-dimensional manner. You can conceptualize this as a long string;
indeed, if fully stretched the DNA in the genome would be six feet long. Inside the nucleus, however, it must fit into a space that is 1/10th the width of a human hair. 
As such, interactions 

One way to probe this is the Hi-C assay: Hi-C reveals compartmentalization of the genome. 

## About dcHiC

dcHiC is a tool for differential compartment analysis of Hi-C datasets. dcHiC is the only tool to perform Hi-C compartment analyses between multiple datasets. The application
provides some cool functionality: 

- Comprehensive identification of significant compartment changes between any number of cell lines (with replicates)
- Beautiful standalone HTML files for visualization of results
- Option to perform compartment computations on intra-chromosomal (cis) and inter-chromosomal (trans) Hi-C counts
- Optimized PCA calculations (capable of analysis up to 5kb resolution)
- HMM subcompartment identification based on compartment values
- Identification of differential loops anchored in significant differential compartments (using Fit-Hi-C)

- Regeneron Science Talent Search Finalist (Top 40 in the nation; $25,000 prize)
- A [presentation](https://www.youtube.com/watch?v=SwSuhfl0p34) in the Regulatory and Systems Genomics working group at the 28th Intelligent Systems for Molecular Biology conference
- Work featured in Forbes and ABC News
- California / San Diego Science Fair Finalist
