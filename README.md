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
The program acts as an interface to an *annotation repository* designed for portable storage of annotation data, granular annotation refinement, tracking of changes, and dissemination to a wider community.
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

The GeneAnnoLogy program is part of the AEGeAn Toolkit (https://brendelgroup.github.io/AEGeAn) and makes extensive use of the GenomeTools C library (http://genometools.org).
It is implemented as a program to be run on the UNIX command line and is compatible with popular UNIX operating systems such as Linux and Mac OS X.
GeneAnnoLogy's command-line interface makes use of a *subcommand* design, in which a master program (`geneannology`) is used to manage a variety of processing tasks by invoking various sub-programs.
A detailed description of these tasks and programs is provided in the **Repository management** section.

Data input consists of well-formed gene annotations in GFF3 format.
The requirement that data be "*well-formed*" refers to the fact that gene structures can be annotated using a variety of alternative conventions.
For example, it is common to annotate a gene's coding sequence with multiple `CDS` features, each corresponding to an exon or a portion thereof devoted to encoding a protein.
A less common but equally valid convention is to annotate exon structure with `exon` features and the location of the coding sequence with `start_codon` and `stop_codon` features.
GeneAnnoLogy is designed to support any such convention, provided that the explicitly declared features describe valid gene structures in sufficient detail so that implicitly declared features can be inferred.
GeneAnnoLogy leverages several relevant modules from the AEGeAn Toolkit to provide this functionality: the *AgnInferCDSStream*, *AgnInferExonsStream*, and *AgnInferParentStream* modules for inferring implicitly annotated features, and the *AgnGeneStream* module for validating input data.

```
# Two alternative annotation conventions that encode identical information and are equally valid.
Chr1	SNAP	gene	755208	757581	.	+	.	ID=gene1
Chr1	SNAP	mRNA	755208	757581	.	+	.	ID=mRNA1;Parent=gene1
Chr1	SNAP	exon	755208	755377	.	+	.	Parent=mRNA1
Chr1	SNAP	exon	755717	756070	.	+	.	Parent=mRNA1
Chr1	SNAP	exon	756808	757581	.	+	.	Parent=mRNA1
Chr1	SNAP	CDS	755305	755377	.	+	0	ID=CDS1;Parent=mRNA1
Chr1	SNAP	CDS	755717	756070	.	+	2	ID=CDS1;Parent=mRNA1
Chr1	SNAP	CDS	756808	757136	.	+	2	ID=CDS1;Parent=mRNA1
###
Chr1	SNAP	gene	755208	757581	.	+	.	ID=gene1
Chr1	SNAP	mRNA	755208	757581	.	+	.	ID=mRNA1
Chr1	SNAP	exon	755208	755377	.	+	.	Parent=mRNA1
Chr1	SNAP	exon	755717	756070	.	+	.	Parent=mRNA1
Chr1	SNAP	exon	756808	757581	.	+	.	Parent=mRNA1
Chr1	SNAP	start_codon	755305	755307	.	+	.	Parent=mRNA1
Chr1	SNAP	stop_codon	757134	757136	.	+	.	Parent=mRNA1
```

Operations on annotation data are implemented in a streaming fashion, with data files as the source and an annotation repository as the endpoint.
Following design principles implemented more generally by GenomeTools and AEGeAn (cite GenomeTools paper and iLocus paper), efficient sequential processing of individual annotations is achieved by composing distinct modular *node streams*, each designed for a particular annotation processing task.
Thus, annotations can in general be processed one by one, keeping memory requirements low.
Some operations, however, involve merging data from multiple sources and require loading all annotations into memory at an intermediate stage in the processing procedure.
These do not have the same memory efficiency advantages of the truly streaming operations, but are implemented using the same node stream interface.

GeneAnnoLogy leverages the *AgnLocusStream* module to organize gene annotations into *iLoci*, each corresponding to a gene or set of overlapping genes (cite iLocus paper).
Each iLocus is written to a distinct file in the annotation repository via the *AgnRepoStream* module.
A detailed description of repository structure is provided in the **Repository structure** section.

GeneAnnoLogy's version control mechanisms are delegated entirely to the git version control system (cite git).
A GeneAnnoLogy repository follows precise patterns of data organization but in all other respects is a basic git repository.
In fact, while the most common operations on the repository (such as data import, data updates, and logging a new commit) must be mediated through the `geneannology` program, other operations (such as branching, merging, pushing, pulling, and cloning) can be invoked directly by the `git` program.
As a result, GeneAnnoLogy annotation repositories are compatible with services and tools designed to work on git repositories.
For example, only trivial effort is required to host a GeneAnnoLogy repository on GitHub and use GitHub as a focal point for data distribution and community engagement.

### Repository structure

```
├── .git/
├── NC_004353.4/
├── NC_004354.4/
├── NC_024512.1/
├── NT_033777.3/
├── NT_033778.4/
├── NT_033779.5/
├── NT_037436.4/
├── gene-locus.map
└── sequence-gene.map
```

A GeneAnnoLogy annotation repository consists of a metadata directory (`.git/`), two map files (`gene-locus.map` and `sequence-gene.map`), and a set of data directories.
The metadata directory stores the repository's version history, and is intended only for internal use by the `git` program.
The map files are created by GeneAnnoLogy, and maintain the relationship between genes, iLoci, and sequences, enabling efficient targeted processing of annotation data without the need for loading all data into memory or issuing many file-system-level commands.
The map files are intended only for internal use by the `geneannology` program.

The actual gene annotations are stored in data directories.
GeneAnnoLogy maintains a dedicated data directory for each genomic sequence, such as a chromosome, scaffold, or contig.
Within each data directory, annotations are stored in GFF3 format with a dedicated file for each iLocus.
GFF3 files use a plain text encoding and can easily be examined by end users.
Even minor manual edits by end users are typically acceptable, assuming that the genomic coordinates of genes are not changed (which could require recomputing iLoci, renaming of some annotation files, and rebuiling map files).
However, changes to the annotation data will typically be mediated by the `geneannology` program, as described in the **Repository management** section.

### Repository management

GeneAnnoLogy provides several operations to facilitate management of annotation data in the repository, supporting a range of maintenance strategies.
Initial setup of a new annotation repository involves two operations: `init` and `commit`.
The `init` operation initializes a new annotation repository and populates it with a set of gene models provided by the user.
The `commit` operation records the first snapshot of the repository's contents in the version history.
The `init` command is only invoked once for each repository, while the `commit` command can be invoked frequently during the annotation lifecycle to record snapshots of subsequent updates to the repository.

The `clean` operation discards the complete complement of annotations currently in place in the repository and replaces it with a new user-provided annotation set.
This operation is intended for applying upgrades to an entire annotation set.

The `union` operation integrates a set of user-provided annotations into the repository, and can be used to annotate new gene models or maintain two or more alternative sources of annotation for the same assembly side-by-side in the repository.
In the latter case, the `source` property of each annotated feature (GFF3 column 2) is crucial in distinguishing annotations by source.
Accordingly, the `union` command (like most `geneannology` commands) includes a `--source` option for setting or resetting the `source` property when integrating user-provided annotations into the repository.

The `merge` operation also integrates a set of user-provided annotations into the repository, but rather than maintaining multiple sources of annotation side-by-side, `merge` is used to maintain a non-redundant set of annotations.
User-provided annotations are merged using one of two strategies.
The `complement` merge strategy integrates any user-provided annotations that do not overlap with annotations already in the repository, but discards those that do overlap.
The `replace` merge strategy integrates all user-provided annotations, discarding any annotations already in the repository that overlap with the user-provided annotations.
These merge operations can be applied to entire genome scale annotation sets, or they can involve small targeted additions or refinements to the annotation.

The `delete` operation discards a user-specified annotation, or all annotations within a user-specified genomic region, facilitating the removal of spurious gene predictions or other artifacts of the annotation workflow.

The `show` operation aggregates the repository contents into a single GFF3 file for distribution or analysis with external programs.
This operation includes several filtering mechanisms and other options that furnish a powerful tool for data analysis.
The annotations reported by the `show` command can be restricted to a user-specified genomic region or annotation source; they can be filtered based on their length, exon count, and other intrinsic characteristics; they can be filtered based on arbitrary user-specified numerical or categorical attributes; and any of these filters can be combined.
By default, the `show` operation reports the contents of the most recent commit, but also includes the option of reporting annotations from previous commits.

After the the initial repository setup, invoking the `commit` operation stores a permanent snapshot of any changes that have been applied to the repository since the most recent commit.
At any time, a previous commit can be temporarily or permanently restored using git's `checkout` command.

## Results and Discussion

To go here.

## Conclusions

To go here.
