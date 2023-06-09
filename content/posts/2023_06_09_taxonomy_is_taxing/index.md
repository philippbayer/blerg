---
title: "Taxonomy is taxing"
date: 2023-06-09T20:32:22+08:00
description: i ramble about taxonomy
---

# Taxonomy is taxing.

It's been a year since I started at [Minderoo OceanOmics](https://www.minderoo.org/oceanomics/), jumping from a plant bioinformatics background into fish eDNA. The difference has been huge: I went from working with three species to working with about 6,000 species (in Australia; worldwide it's more than 30,000 species). For fish taxonomy, there are at least three databases: [The World Register of Marine Species (WoRMS)](https://www.marinespecies.org/), [Eschmeyer's Catalog of Fishes](https://researcharchive.calacademy.org/research/ichthyology/catalog/fishcatmain.asp), and [Fishbase](https://fishbase.org.au) (plus [NCBI Taxonomy](https://www.ncbi.nlm.nih.gov/Taxonomy/Browser/wwwtax.cgi), [Australian Faunal Directory](https://biodiversity.org.au/afd/home), [Global Biodiversity Information Facility (GBIF)](https://www.gbif.org/), [Atlas of Living Australia](https://www.ala.org.au/), etc. pp.). They all have different goals; Eschmeyer, for example, focuses mostly on taxonomic changes, synonyms etc., while Fishbase tries to do more than that.

I had to learn the hard way how messy taxonomy can get; here are some of my main mistakes.

## Mistake 1: a taxonomic name has a genus and a species name.
 
Here's a lazy shortcut to check whether a taxonomic label is a species: count the spaces in the string, there should be only one, as in *Homo sapiens* or *Carcharhinus altimus*. But then you find species in sequencing and sighting databases like *Phrynocephalus mystaceus aurantiacocaudatus* or, the taxonomic database equivalent of 'I'll do that later', *Ulva* sp. BW-2013 (clade G) cf. or *Molannodes* SC sp.

Then there's the hybrids! Those have an X in the middle, the X stands for 'eXtreme headache'. My favorite so far is '(*Megalobrama amblycephala* x *Megalobrama terminalis*) x (*Megalobrama amblycephala* x *Parabramis pekinensis*)'. That's one species; a nightmare to parse. ([TaxoNERD](https://github.com/nleguillarme/taxonerd/issues/11) can help!)

## Mistake 2: there's one 'common' name for every scientific name.

The common name for *Homo sapiens* is 'human'. With fish the situation is more complex, mainly for historic reasons, with many fish species having a pile of common names. For *Aldrichetta forsteri*, Fishbase lists the common English names: Conmuri, Coorong mullet, Estuary mullet, Forster's mullet, Freshwater mullet, Mullet, Pilch, Pilchard, Victor Harbor mullet, Yellow eye mullet, Yelloweye, Yelloweye mullet, and Yellow-eye mullet. Usually there's a preferred name but those differ from database to database. The Australian Fauna Directory lists Yelloweye Mullet as the preferred common name (Wikipedia spells it 'Yellow-eye mullet' - have fun writing your text parsers!). There's also a Yelloweye Puffer and a Yelloweye Mullet, but those are different species.

## Mistake 3: a species name is reasonably stable.

In the olden days, Fishbase decided to use the scientific name as stable IDs for a species. But species names change constantly, often being merged into other species or different genera. *Pagrus ruber* became *Argyrops spinifer*, for example. You *can* keep such a database straight, but it's a nightmare.

Far better: World Register of Marine Species (WoRMS) uses a stable numeric AphiaID. When in doubt, ask [Eschmeyer](https://researcharchive.calacademy.org/research/ichthyology/catalog/fishcatmain.asp)! Fishbase has a tool to pull out all known synonms and the accepted names for a taxonomic name: NCBI BLAST results, for example, quite often result in an outdated name or a synonym.

## Mistake 4: a taxonomic family is stable.

[Open Tree Of Life](https://tree.opentreeoflife.org/about/open-tree-of-life) collates published phylogenies into a large synthetic tree. Enter any of the larger fish families and you'll get an error that there's no monophyly for this family: 'This taxon is in our taxonomy but it's "broken" (not monophyletic) in the latest synthetic tree.'. Try it with the Gobiidae (the favorite study family of the previous Japanese Emperor!). What that means is that this is not 'one' family but likely several, to be split up in the future. This happens with most larger fish families.

## Mistake 5: 'Fixed' names will propagate across databases.

I mentioned *Argyrops spinifer* above. In the Australian export documentation system, these were renamed into *Argyrops bleekeri* [Source](https://www.agriculture.gov.au/biosecurity-trade/export/controlled-goods/fish/industry-advice-notices/2022/2022-02). Eschmeyer and Fishbase say these are two different species. I don't know hwhy either. NCBI Taxonomy does not include names for species that have no data deposited; lots of known fish out there with no Taxonomy ID.

## Mistake 6: surely the families are identical everywhere?

*Sphyrna lewini*, the scalloped hammerhead, is within the family Sphyrnidiae in Eschmeyer and Fishbase. In [NCBI Taxonomy](https://www.ncbi.nlm.nih.gov/Taxonomy/Browser/wwwtax.cgi?id=7823) it's in the family Carcharhinidae. I guess NCBI Taxonomy is wrong there.

## Mistake 7: I can mostly trust genomics databases.

I'd say a good 5 to 10% of the genomic marker sequences deposited for fish on NCBI have the wrong species label. Curating such a database for Australian fish has been part of my recent work. There's a public curated list with a UK emphasis within [meta-fish-lib](https://github.com/genner-lab/meta-fish-lib/blob/main/assets/exclusions.csv). Look at all these wrong species! Fixing those within NCBI is not easy, there's no real 'complain here' pathway. Usually the submitting group itself has to fix those issues. All you can do is keep a blacklist and always evaluate your hits. 

Even if the NCBI entry gets fixed it can get confusing: [NC_004409.1](https://www.ncbi.nlm.nih.gov/nuccore/NC_004409.1) has *Lycodes toyamensis* in the title but was submitted as, and still has the ORGANISM field, *Icelus toyamensis*. According to Eschmeyer, those are two different species. If you BLAST this mitogenome it does look like a *Lycodes*, but who knows if the species is correct - there's no changelog that indicates what the reasoning for the new name was.

## Mistake 8: ChatGPT can fix all this.

ChatGPT 3.5 hallucinates nonsense about scientific names, roughly guessing from their Latin meaning. ChatGPT 4 usually says 'I don't know' as details about fishes appear far more rarely in the training data, than, say, the opinions of Redditors on Game Of Thrones.
