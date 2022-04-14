+++
title = "Genome architecture"
hascode = true
date = Date(2022, 3, 11)
rss = "Genome architecture and how to find it"
+++
@def tags = ["syntax", "code"]

# Genome architecture

\toc

## Introduction

One of the most fundamental pieces of information retrieved from any prokaryotic genome is the total number of encoded genes. The related concepts coding density (ratio coding genome / total genome) and gene density (genes per base) are commonly used in addition to the total number of genes, but the underlying genome architecture that belies a dense or sparsely coded genome is not often discussed past the point of establishing that a genome is or isn’t dense. I wanted to explore the tunable variables that enable dense or sparse genomes. Is there a one size fits all strategy for dense or sparse genomes, are there multiple possible strategies, or is there no shared strategy at all?

## Gathering data

Incomplete and poorly annotated genomes will greatly influence these metrics. For this reason I used the [refseq](https://www.ncbi.nlm.nih.gov/books/NBK50679/#RefSeqFAQ.what_is_a_reference_sequence_r) database. There is a minimum of quality required for inclusion in the refseq database, and the genome annotations are regularly updated to reflect the newest insights.  

I used the following query to retrieve gff files from the NCBI refseq database. 

```
("Bacteria"[Organism] OR "Archaea"[Organism]) AND latest[filter] AND ("complete genome"[filter] OR "scaffold level"[filter] OR "chromosome level"[filter] OR "contig level"[filter]) AND "full genome representation"[filter] AND (all[filter] NOT "from large multi isolate project"[filter] AND all[filter] NOT anomalous[filter] AND all[filter] NOT partial[filter]) AND "genbank has annotation"[Properties] AND "taxonomy check ok"[filter] AND ("1"[ContigCount] : "50"[ContigCount])
```

The most relevant criterion is the max contig count of 50, as fragmented genomes will affect these metrics. 

Retrieved 64268 genomes, which were filtered for presence in the SILVA database (61320) identified superkingdom (61317), and then de-replicated to have one representative for each family (7571).

## Calculating genome properties

Calculating the various properties that consiture genom architecture is relatively straightforward. 

| Genome properties | calc      |
|-------------------|-----------|
| genome size       | $\sum_{i=1}^{\textrm{total\;contigs}} \left(\textrm{contig} \cdot \textrm{length\;contig}\right)$ |
| total gaps        | $\textrm{if\;noncircular, total\;genes}\;-1, \textrm{else,\;total\;genes}$ |
| total gene length | $\sum_{i=1}^{n} (\textrm{end\;of\;gene}_{n} - \textrm{start\;of\;gene}_{n})$ |
| mean gene length  | $\textrm{total\;length\;of\;genes} \cdot \textrm{total\;genes}^{-1}$ |
| gene density      | $\textrm{total\;number\;of\;genes} \cdot \textrm{kilobase}^{-1}$ |
| efficiency        | $\textrm{mean\;gene\;length} \cdot \textrm{total\;number\;of\;genes} \cdot \textrm{genome\;size}^{-1}$ |
| con- & di-vergent overlap proportion   | $(\textrm{divergent+convergent\;overlaps}) \cdot \textrm{total\;genes}^{-1}$ |
| total unidirectional overlaps          | $\textrm{intergenic\;gaps}\;<\;0$ |
| unidirectional overlap proportion      | $\textrm{unidirectional\;overlaps} \cdot \textrm{total\;genes}^{-1}$ |
| sum intergenic gap size                | $\sum_{i=1}^{n-1} (\textrm{start\;of\;gene}_{n+1} - \textrm{end\;of\;gene}_{n}) $ |
| mean intergenic gap size               | $\textrm{sum\;intergenic\;gap\;size} \cdot \textrm{total\;gaps}$ |
| unidirectional overlap size            | $\textrm{for\;intergenic\;gaps}\;<\;0, \frac{|\textrm{mean\;gap\;size}|}{\textrm{total\;gaps}}$ |
| strandedness      | $\log_{10}\left(\frac{\textrm{total\;genes\;+\;strand}}{\textrm{total\;genes\;-\;strand}}\right)$ |

## Visualising large datasets

Plotting data originating from 7571 genomes is difficult, as the density of of points is high. Just adjusting alpha's does not allow for the detailed look into the variables that we require. That is why I decided to use kernel density estimates as a way to visualize the dense data cloud. 

upcoming.


