# About dcHiC

This page briefly rehashes some of my research work on dcHiC, an application that combines uses machine learning, statistical modeling, and data visualization to identify significant changes in the genome's 3D structure.
This work was done with the Ay Lab at the La Jolla Institute for Immunology; the application is released [open-source](https://github.com/ay-lab/dcHiC) and [published](https://www.biorxiv.org/content/10.1101/2021.02.02.429297v1) (first author). 

## The Classic "3D Genome" Background (skip if you are in the field)

Since the 1970s, the scientific community has known that the genome folds in a highly-structured, 3-dimensional manner. You can conceptualize this as a long string;
indeed, if fully stretched the DNA in the genome would be six feet long. Inside the nucleus, however, it must fit into a space that is 1/10th the width of a human hair. This packing isn't done arbitrarily; instead, there is a very systematic way that the genome is compacted which directly influences its downstream function. This is usually depicted something like this:

![3D Genome Image](https://www.science.org/cms/10.1126/science.347.6217.10/asset/f8aa4730-f26e-4123-8e9a-d5c728d13617/assets/graphic/347_10_f1.jpeg)

With this type of structure, areas of DNA that are very far away from each other in linear distance can be proximal in the three-dimensional structure. One way to probe this architecture is an assay known as Hi-C: a high-throughput method to identify contacts between different areas of chromatin. The fundamentals are quite simple: (1) crosslinking cells with formaldehyde, (2) digesting the DNA with a restriction enzyme that leaves sticky ends, (3) filling in the sticky ends and marking them with biotin, (4) ligating the crosslinked fragments, (5) shearing the resulting DNA and pulling down the fragments with biotin, and (6) sequencing the pulled down fragments using paired-end reads. 

![Hi-C Assay Image](https://s3.amazonaws.com/4dn-dcic-public/static-pages/InfoBoxes/IsHC_fig1.png)

While there are many variations of Hi-C, the procedure always involves cross-linking DNA, "snipping" parts that have been linked together, and sequencing them to generate some type of map. This procedure produces a genome-wide sequencing library that provides a proxy for measuring the three-dimensional distances among all possible locus pairs in the genome. Hi-C produces maps like these:

![Hi-C Maps from different species](https://www.researchgate.net/profile/Quentin-Szabo/publication/332332535/figure/fig2/AS:746205512491008@1554920666132/Examples-of-Hi-C-profiles-from-different-species-Hi-C-maps-visualized-with-Juicebox.png)

In each map, the x and y-axes represent the linear distance, in base pairs, over a chromosome. The main diagonal reflects expected interactions—parts of the "long DNA string" that are close in linear distance are likely also close in 3D distance. However, the off-diagonal interactions reflect interesting biology: these represent something truly unusual—long-distance interactions that might mean something. 

Well... what could they mean?

For a new-comer in biology, this could be a bit complicated. Bear with me! 

For a long time, we've known that chromosomes segregate into "chromosome territories." In side the nucleus, chromosome 1 mostly occupies its own space, as do chromosomes 2, 3, etc. You've probably seen this in classic chromosome blotting images like [this](https://www.researchgate.net/profile/Wen-Deng-4/publication/264428089/figure/fig4/AS:271835758632999@1441822106129/Western-Blotting-and-chromosome-aberration-analysis-a-Western-Blotting-analysis-for.png). Using Hi-C, however, we have been able to identify new levels of nuance in DNA organization: compartments, TAD's, and loops. 

![Compartment/TAD/Loops Image](https://www.frontiersin.org/files/Articles/626541/fcell-08-626541-HTML/image_m/fcell-08-626541-g001.jpg)

Compartments are the key aspect of biology that my research studied: they are areas of DNA that preferentially segregate with each other; there are two main ones, the A compartment and the B compartment. The A compartment reflects areas of "active" DNA (also known as "open" chromatin or euchromatin), with increased transcription and activity. The B compartment reflects areas of "inactive" DNA (also known as "closed" chromatin or heterochromatin). 

Far from being a binary, however, there are also varying degrees of compartmentalization—some areas are 'more A' than others, and others 'more B' than others. As such, compartmentalization is largely a spectrum; there are also areas of DNA that lie in between compartments in 3D space which exhibit tendencies of A and B. While comaprtments are the most oft-used measure to determine genome activity at a broad level, "subcompartments" have also been termed, which more precisely stratify "tiers" of activity within A and B. Most published research in subcompartments show that there are 6 of these. 

Compartments give way to finer-grained biological characteristics into which I will not go into much detail here, including Topologically-Associating Domains (TAD's) and loops. TAD's are areas in compartments that preferentially associate with each other, and a loop can be thought of as a precise strong "link," typically with a few specific binders, between two areas of DNA. 

## About dcHiC

dcHiC is a tool for differential compartment analysis of Hi-C datasets. Traditional compartment "analysis" usually comprises a few steps:
- Taking the Hi-C heatmap, where interactions must be binned at a certain resolution to allow for processing (usually ~100,000 base pairs (100kb) or so)
- Performing some preprocessing (computing some standard distance normalization/correlation calculations)
- Performing PCA (eigendecomposition) on the map, and getting the first eigenvector. Positive values correspond to the A compartment, and negative values correspond to the B compartment.
- Downstream analysis usually involves taking pairs of compartment profiles, and comparing sign flips (compartment switches)

### The Issues

There are a few issues with this method of analysis:
- It's binary: Comparing two cell profiles is useful, but is becoming difficult with ever-more complex experimental setups and the proliferation of data. For instance, in a time series of data mapping a normal cell's path to a cancer cell, methods that could identify the most significant switches in genome activity over the course of the data would be very useful. 
- It's imprecise: The precise meaning of a compartment varies between cell line. A "weak A" in one cell profile might register as a "weak B" in another, because compartment identification relies on eigendecomposition of Hi-C maps, each of which carry some intrinsic biological biases (in the cross-linking process, on the restriction enzyme, etc.). 
- It's imprecise (v2): Because eigendecomposition is a computationally expensive process, Hi-C maps must be binned at a coarse resolution (>50kb). Every time the resolution is improved by a factor of *x*, the map sizes increases by a factor of *x*^2. 

### dcHiC's Solutions

dcHiC offers solutions to these features and more. From the compartment computation side, it employs some numerical optimizations to SVD to analyze. While the standard in academia is usually 50-100kb, which might take a few hours to a few days (depending on data size) using standard R/C++ methods, dcHiC performs the same computations in a few minutes to a few hours. While other methods top out at about 10-25kb due to processing power limitations, dcHiC performs until 1-5kb. This allows for an unprecedented degree of nuance in calculations. 

To compare, dcHiC first normalizes employs a motley of multivariate regression and statistical scores to find the most significant changes across compartments. It employs [Independent Hypothesis Weighting](https://www.nature.com/articles/nmeth.3885) (Benjamini-Hochberg method combined with a seperate covariate) to increase power of detection. The complete workflow is below. 

![dcHiC Workflow](https://github.com/jeffreyywangg/about-dcHiC/blob/main/dchic_workflow.jpg)

In effect, this is a solution to a yet-unsolved problem in comparative genomics: a rigorous, biologically-meaningful way to identify significant compartment changes across multiple cell lines. There a few other nice features that make dcHiC a great tool for analysis:

#### Differential Loops in Compartments (+ Subcompartments)

As mentioned earlier, there are finer-grain biological entities that reside within compartments which often offer a more nuanced view of the biological underpinnings of certain processes (like compartment flips). To help identify these, dcHiC offers two other features:
- Identifying subcompartments (6-part Hidden Markov Model with PC values)
- Finding significant loops anchored in differential compartments (which may be the active gene that causes the compartment change) 

#### Data Viz

dcHiC outputs beautiful, standalone HTML files for visualization of results: these are scrollable, pannable, and customizable. 

![dcHiC data viz](https://github.com/jeffreyywangg/about-dcHiC/blob/main/dcHiC_dataViz.png)

## My Work and Honors

I began working on this project in July of 2019 and worked on it at a large capacity until summer of 2021. Here are some cool honors I have received!

- First author of a [paper](https://www.biorxiv.org/content/10.1101/2021.02.02.429297v1) under peer review
- Regeneron Science Talent Search Finalist (Top 40 in the nation; $25,000 prize)
- A [presentation](https://www.youtube.com/watch?v=SwSuhfl0p34) in the Regulatory and Systems Genomics working group at the 28th Intelligent Systems for Molecular Biology conference (<20% of submissions accepted) 
- Work featured in [Forbes](https://www.forbes.com/sites/carolineseydel/2021/03/26/be-relentless-meet-four-teenage-geneticists-who-are-forging-their-own-paths/) and [ABC News](https://www.10news.com/positivelysandiego/positively-san-diego-2-local-high-school-seniors-hoping-their-projects-help-people-after-being-named-finalists-in-prestigious-science-competition)
- California / San Diego Science Fair Finalist
