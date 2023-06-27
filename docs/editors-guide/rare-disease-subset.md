# Rare Disease Subset in Mondo

A rare disease is a disease that affects fewer than 200,000 people in the United States. However, this definition is not universal, and it varies between rare disease sources, geographical locations, etc. Consequently, rare disease criteria also vary between sources, and one might have difficulties reconciling sources and creating a list for rare diseases. Mondo recognizes these impediments and differences of view. This document explains how Mondo handles rare diseases, how one could get a rare disease list, and the efforts made to bring the community together.

#### What are rare disease in Mondo?
We rely on rare disease authorities (see table below) to determine all diseases recognized as “rare”, according to their own criteria. When a disease is considered “rare” according to a source, the corresponding Mondo term is annoted with `in_subset` annotation expressing that this disease is considered rare according to this source: `X_rare`  (where X is the rare disease authority).

example: calciphylaxis (MONDO:0017215) is considered "rare" according to NORD, GARD, and Orphanet, as shown by the 'in_subset' annotations 'nord_rare', 'gard_rare', 'orphanet_rare'.
![Screenshot 2023-06-27 at 2 59 17 PM](https://github.com/monarch-initiative/mondo/assets/12737987/f7347c08-9a64-4fbc-8be0-41eb6efa334d)


#### The Mondo rare disease subset include ALL rare diseases according to ALL rare disease sources.

The Mondo rare disease subset represents the UNION of rare diseases list according to all authority sources. Most of the diseases in this subset will be viewed as “rare” by most authorities, but some will be viewed as “rare” by only some authorities.
In addition to the rare diseases as defined by the authoritative sources, this subset includes diseases identified as "rare" based on the method used in our [2021 publication](https://mondo.monarchinitiative.org/pages/analysis/).

Rare disease terms in Mondo are annotated with `in_subset``mondo_rare` (see image above), and include
- terms annotated with the ‘bearer of’ some rare axiom
- **AND** terms with the annotation in_subset ‘X_rare’ (which includes ALL the rare disease authorities represented in Mondo)
- **AND** limited to the ‘human disease’ branch (which excludes ‘susceptibility’ terms)

#### Where to find the Mondo rare disease subset, and what is included in the file?
The Mondo rare disease can be found as a product of the Mondo release [here](https://github.com/monarch-initiative/mondo/tree/master/subsets), in .obo and .json format

This mondo-rare subset includes **all rare diseases** (as defined above) AND their **ontological parents**.

Users can filter using the subset annotation “mondo_rare” to get a list of Mondo rare diseases without including their ontological parents.

Users can filter using the subset annotation “X_rare” (where X is a rare disease authoritative source) to get a list of "rare disease according to X".


#### Current rare disease authority sources represented in Mondo

| Rare disease authority | link | in_subset annotation in Mondo | Rare disease source criteria |
| --- | --- | --- | --- |
|NORD |(website) | nord_rare | *(coming soon)*|
|GARD |(website) | gard_rare | *(coming soon)*|
|Orphanet |(website) | orphanet_rare | *(coming soon)*|



