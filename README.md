# GeneAnnoLogy: quality control and provenance for provisional genome annotations

## Background

Genome sequencing and genome-scale profiling of gene expression, DNA methylation, and protein binding have become routine laboratory procedures due to extraordinary advances in nucleotide sequencing technology.
It is both a blessing and a curse that genome sequencing is no longer restricted to model organisms with large supporting research communities:
the blessing is that so much data can be so easily and cheaply acquired;
the curse is the burden these easily acquired data place on those analyzing genomes without a large supporting research community.

Gold-standard reference assemblies and annotations are simply not possible with current sequencing technologies (although single-molecule long-read sequencing platforms hope to change this), requiring researchers to treat their own genomes with skepticism.
And because optimal parameters for assembly and annotation can be difficult to estimate for a novel genome, iterative testing and refinement is common at the preliminary stages of a genome project.
Many genomes are in a state of continuous refinement, yet it is impractical for researchers to delay analyses of the genome and corresponding publications until the genome assembly and annotation are "finished" to some arbitrary standard of quality.

Databases for many model organisms address this issue by operating on a regular release schedule, providing snapshots of all raw and processed data at a particular point in time, as stable references for analysis and publication.
However, most scientists doing genome sequencing with current technologies do not have the time, resources, or expertise to manage, curate, and document stable "releases" of their genomes.
Coincidentally, these issues bear resemblance to issues the software development community has dealt with for many decades, and robust tools for tracking provenance of computer files (version control systems) are freely available and well documented.

We introduce GeneAnnoLogy, a tool for quality control and provenance tracking for provisional genome projects.
GeneAnnoLogy parses an annotated genome assembly into *iLoci* (cite iLocus paper)—each corresponding to a genic or intergenic region— and provides mechanisms for filtering *iLoci* based on gene content, length, nucleotide composition, and any number of additional annotated characteristics such as homology status or annotation quality.
GeneAnnoLogy also leverages the git version control system (cite git) to take snapshots of the annotation, providing a simple solution for managing rapidly changing annotation data during the incipient stages of a genome project.
We demonstrate and discuss how GeneAnnoLogy's quality control and version control features facilitate reproducible genome analyses.
