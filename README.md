# GeneAnnoLogy: quality control and provenance for provisional genome annotations

## Background

Genome sequencing and genome-scale profiling of gene expression, DNA methylation, and protein binding have become routine laboratory procedures/services due to extraordinary advances in nucleotide sequencing technology.
It is both a blessing and a curse that genome sequencing is no longer restricted to model organisms with large supporting research communities:
the blessing is that so much data can be so easily and cheaply acquired;
the curse is the burden these easily acquired data place on scientists analyzing genomes without a large supporting research community.

Although single-molecule long-read sequencing platforms promise substantial improvements in the near future, gold-standard reference assemblies and annotations are simply not possible with current sequencing technologies, requiring researchers to treat data with skepticism.
And because optimal parameters for assembly and annotation can be difficult to estimate for a novel genome, iterative testing and refinement is common at the preliminary stages of a genome project.
Many genomes are in a state of continuous refinement, yet it is impractical for researchers to delay analyses of the genome and dissemination of corresponding research products to the community until the genome assembly and annotation are "finished" to some arbitrary standard of quality.

Databases for many model organisms address this issue by operating on a regular release schedule, providing snapshots of all raw and processed data at a particular point in time, as stable references for analysis and publication.
However, most scientists doing genome sequencing with current technologies do not have the time, resources, or expertise to manage, curate, and document stable "releases" of their genomes.
Coincidentally, these issues bear resemblance to issues the software development community has dealt with for many decades, and robust tools for tracking provenance of computer files (version control systems) are freely available and well documented.

We introduce GeneAnnoLogy, a tool for quality control and provenance tracking for provisional genome projects.
GeneAnnoLogy parses an annotated genome assembly into *iLoci* (cite iLocus paper)—each corresponding to a genic or intergenic region— and provides mechanisms for filtering *iLoci* based on gene content, length, nucleotide composition, and any number of additional annotated characteristics such as homology status or annotation quality.
GeneAnnoLogy leverages the git version control system (cite git) to take snapshots of the annotation, providing a simple solution for managing rapidly changing annotation data during the incipient stages of a genome project.
We demonstrate and discuss how GeneAnnoLogy's quality control and version control features facilitate reproducible genome analyses.

## Methods

### Design

GeneAnnoLogy was devised to address reproducibility challenges as discussed in the **Background** section.
The program acts as an interface to an *annotation repository* designed for portable storage of annotation data, granular annotation refinement, tracking of changes, and dissemination to the wider community.
User input consists of one or more annotation data files in GFF3 format, from which the repository is initially populated.
Subsequent repository maintenance is mediated through various GeneAnnoLogy commands, and may include integration of annotation data from additional sources or updates to existing annotations.

The defining feature of version control systems is the ability to take a snapshot (a *commit* in version control parlance) of repository contents after any change or group of changes is applied.
At any time, a user may examine the contents of the repository at the time of a particular snapshot.
Traditionally, this feature has been applied in software engineering to track down and isolate changes that introduce bugs into code.
In the context of genome annotation, version control enables a continuous data refinement strategy, with the ability at any time to temporarily revert data to a previous state to, for example, replicate a published analysis.

Other common version control tasks are also available through GeneAnnoLogy.
These include a *status* check to determine whether any changes have been applied to repository contents since the last commit, and a *diff* to enumerate uncommitted changes or, alternatively, differences between two different commits.
Typically, the *status* and *diff* commands of a version control system report file names and line numbers associated with each change, but GeneAnnoLogy instead reports changes and differences from the perspective of genome biology, such as the number of gene loci with uncommitted changes and the magnitude of those changes in terms of annotated gene structure.

### Implementation

To go here.

### Repository structure

To go here.

### Repository management

To go here.

## Results and Discussion

To go here.

## Conclusions

To go here.
