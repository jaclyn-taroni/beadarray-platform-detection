# Illumina BeadArray Platform Detection

We're interested in automatically detecting platform/annotation based on the Illumina expression beadchip data that is publicly available. Some background discussion [here](https://github.com/AlexsLemonade/refinebio/issues/232). This repository contains some preliminary analyses of human platforms.

## Data and methodology overview

### Series Lists

We're looking at Illumina Human "whole genome" chips. (These chips have additional transcripts beyond what are on the "Ref" platforms.) 

These are the GEO accessions we're using (from the linked issue above):

| Platform Name | GEO Accession |
|----------------|-----------------|
| Human-6 v1.0 | [GPL2507](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GPL2507) |
| Human-6 v2.0 | [GPL6102](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GPL6102) |
| HumanHT-12 v3.0 | [GPL6947](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GPL6947) |
| HumanHT-12 v4.0 | [GPL10558](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GPL10558) |

`data/series_lists` from each of these platforms were obtained through the [GEO Browser](https://www.ncbi.nlm.nih.gov/geo/browse/?view=platforms&display=20) on May 17, 2018. The platform accession was used as a search term -> click on series -> Export -> All Search results & Tab. 

### Non-normalized data

For each platform, we randomly selected 30 series that had supplementary txt files (e.g., there was a chance that there was `non-normalized.txt` files were available). 
We downloaded supplementary files that matched this pattern: `*non*`.
In this sample, newer platforms had more accessions with raw data (as defined by this pattern; see table below).

| platform | no. accessions |
|----------|----------------|
|GPL2507 | 10 |
| GPL6102 | 18 |
| GPL6947 | 27 |
| GPL10558 | 27 |

### List of probes and calculating overlap

Lists of probes were obtained from the following bioconductor packages (`v1.26.0`): [`illuminaHumanv1.db`](https://bioconductor.org/packages/illuminaHumanv1.db/), [`illuminaHumanv2.db`](https://bioconductor.org/packages/illuminaHumanv2.db/), [`illuminaHumanv3.db`](https://bioconductor.org/packages/illuminaHumanv3.db/), [`illuminaHumanv4.db`](https://bioconductor.org/packages/illuminaHumanv4.db/)

For each series, we calculated what proportion of identifiers were in each of the lists of probes and the proportion of probes that were in the identifiers. _Identifiers were (naively) assumed to be in the first column, as this is generally consistent with GEO instructions._

## Results

The newer platforms (v2 and beyond) have [some overlap in their identifiers](https://github.com/jaclyn-taroni/beadarray-platform-detection/blob/master/plots/probes_list_venn.png). However, for most series, the identifiers from the data [had the highest overlap with the platform for which it was labeled](https://github.com/jaclyn-taroni/beadarray-platform-detection/blob/master/plots/ids_heatmap.png). There were some exceptions. 
We highlight the findings from the series labeled `GPL6947` (HumanHT-12 v3.0 ) below.
<br>
<br>
<br>
![](https://github.com/jaclyn-taroni/beadarray-platform-detection/blob/master/plots/GPL6947_jitter_highlight.png?raw=true)

**Fig 1.** Overlap between identifiers from series labeled `GPL6947` and the four platform Bioconductor packages. 


For most series (shown in black), we see the highest overlap with `Humanv3` and some amount of overlap with `v2` and `v4`.
This is consistent with what we would expect based on the relationships between the platforms themselves.
However, some series contained multiple Illumina platforms (and were not SuperSeries), which we would not have detected given our methodology. 
Once we color points from different platforms, we see that they behave as expected. [`GSE34074`](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE34074) `GPL10558` is consistent with the pattern of [most `GPL10558` (HumanHT-12 v4) series](https://github.com/jaclyn-taroni/beadarray-platform-detection/blob/master/plots/GPL10558_jitter_highlight.png). 
The identifiers from [`GSE25580`](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE25580) [`GPL6104`](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GPL6104) (HumanRef-8 v2.0) have the highest overlap with `Humanv2` and less than 50% of the probes from `v2`, `v3`, and `v4` are in the series identifiers, consistent with the smaller subset of transcripts present on Ref chips.

Below, we describe series with overlaps that deviated from this pattern for other platforms.

##### [GPL2507](https://github.com/jaclyn-taroni/beadarray-platform-detection/blob/master/plots/GPL2507_jitter_highlight.png) (Human-6 v1.0)

* [`GSE17241`](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE17241) contains multiple platforms. The [`GPL6106`](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GPL6106) raw data appears to use _bead IDs_ rather than probe IDs.

##### [GPL6102](https://github.com/jaclyn-taroni/beadarray-platform-detection/blob/master/plots/GPL6102_jitter_highlight.png) (Human-6 v2.0)

* [`GSE14295`](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE14295) uses gene symbols rather than probe IDs. We'd be unable to get the probe sequences, which are required for processing, for this experiment.
* [`GSE35102`](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE35102) looks like it might be could be WG-6 `v2` filtered to only probes that are also present on the `v3` chip. This would not matter for processing, as we'd be able to obtain gene identifiers and probe sequences for all the probes.
* [`GSE45331`](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE45331) contains a `PROBE_ID` column that is not the _first_ column, so we missed it using this method.

##### [GPL10558](https://github.com/jaclyn-taroni/beadarray-platform-detection/blob/master/plots/GPL10558_jitter_highlight.png) (HumanHT-12 v4)

* The non-normalized file for [`GSE62374`](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE62374) contains lines of trailing whitespace (e.g., `NA` that were not removed).
* The first column in [`GSE39417`](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE39417) contains gene symbols.
* [`GSE54661`](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE54661) may be a mislabeled and in fact be from a whole genome `v3` platform.
