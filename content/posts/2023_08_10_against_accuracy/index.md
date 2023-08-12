---
title: "Against accuracy"
date: 2023-08-10
description: i ramble about focusing on the wrong thing
---

Let's start with a quote from a recent preprint, Gihawi et al., 2023, [Major data analysis errors invalidate cancer microbiome findings](https://www.biorxiv.org/content/10.1101/2023.07.28.550993v1.full), doi: https://doi.org/10.1101/2023.07.28.550993 which is a re-analysis of the results published after peer review in Poore et al., 2020, [Microbiome analyses of blood and tissues suggest cancer diagnostic approach](https://www.nature.com/articles/s41586-020-2095-1).

>As illustrated in Figure 2, the extremely non-random distribution of normalized values–all but one of which started as raw values of zero–makes it easy for a machine learning classifier to separate the ACC samples from other cancers. If we call the normalized Hepandensovirus value HN, then if the model splits the samples using the simple rule HN>3.078874655, it will label 71/79 (90%) of the positive samples correctly, and only make 77/17,625 (0.4%) errors (Figure 2). This explains why Hepandensovirus was the highest-weighted feature for the machine learning model distinguishing ACC from other cancers in the most stringent decontamination (MSD) data set, despite the fact that only 1/17,624 samples had any reads at all matching this virus.


Peer review and the way research papers are published means that we select for 'outliers' in terms of results. We not only want positive results, we want them to be highly accurate (by 'accuracy' I mean MCC, F1, sensitivity, accuracy; any of those scores that people try to get close to 1.0 or 100% in machine learning). That means that peer reviewers often balk at machine learning accuracies below 95% or 90%. As you can see in the example above, that selection effect may lead to overconfidence.

Here I argue that we should not aim for high accuracies, F1-scores, MCC-scores, etc.: doing so will lead to wrong models. I believe that aiming for high scores is a variant of Goodhart's law: we use these scores to measure how well the model works, but by focusing on them we may end up building models that actually perform worse with real-life data.

# Every unhappy family....

A few years ago I saw a talk at the Telethon Kids Institute (TKI) here in Perth. It was either Timo Lassmann or Dave Tang speaking, I don't remember. My main takeaway was that, counter-intuitively, rare childhood diseases are cumulatively common. A random child might not have *that* rare disease, but chances are it has *a* rare disease. It's the same for datasets: weird rare problems are weird and rare on their own, but every dataset will have several of these 'rare' problems. It's your job to find them!

I use this 90-10-90 rule when training ML models:
- 90% of the time is spent on collecting data, cleaning data, finding mislabels, connecting datasets etc.
- 10% of the time is spent on training the model.
- *Another* 90% of the time is spent on finding all the problems in the model!

The fun thing is that the last step, finding all the errors, will probably decrease your accuracy: wrong models are usually over-confident. In my work with fish environmental DNA data, for example, a few older samples had a Y or an R in their sequenced DNA (NOT A, T, G, or C), which indicates that the DNA sequencer wasn't particularly sure which base was there. But since these Ys are so rare the classifier learned to look for Ys. Other samples from the same lab, with the same species, had the same errors leading to an inflated accuracy. I had to visualise quite a lot of scores before I found the Y-issue. The classifier was very 'sure', but in the real world, it's useless! I am sure many similar issues remain in my data, little unhappy families, rare diseases.

Or think of the recent [dataset issues in the gzip-beats-BERT paper](https://kenschutte.com/gzip-knn-paper2/#dataset-issues). They had a language-specific test-dataset *filipino* where training and test were ~100% identical! Of course that leads to inflated accuracies. That should have been detected by someone simply looking at the model's behaviour and the input data. Or look at Gerry Tonkin-Hill's [recent blog-post](https://github.com/gtonkinhill/TCGA_analysis) on the weird ways spurious correlations in metagenomics normalisation methods can lead to classifiers relying on what is essentially noise: if you don't have simulated data, you will not detect these issues.

I am not talking about the kind of overfitting we usually encounter in machine learning, the kind where the training data accuracy is far better than the testing data accuracy. The kind of problems I talk about appear when there is seemingly no problem during the training process.

# ... is unhappy in its own way

Checking all the cases where your model is overconfident is *boring*. It involves looking at a lot of data, model weights, model predictions for test-data manually, always questioning your own assumptions, digging through pages of stuff, designing and simulating specific test datasets, talking to stake-holders and domain experts. Most people are not into that! Most people got into ML because scikit-learn is so much fun. Learn to look at your data! It's the same in my 'regular' bioinformatics career: absolutely nothing will beat looking at your work for a day or two.

(A note: There's the other special case of naturally constrained accuracies: in my work with 12S, 16S, CO1 marker genes there are many different species that have identical marker genes. It's impossible to get 100% accuracy with these genes, as it's not possible to distinguish these different species with identical genes. I'd guess that in vertebrates, it's probably impossible to get above 92%).

At this point, especially in my messy corner of biology, a high accuracy is a red flag to me. That *could* happen by luck, or because the problem is too easy, but it shouldn't be the rule: a high accuracy indicates that something is going wrong more often than not. You have to work hard to convince me to believe accuracies above 90% or 95%. How much work have you put in to get your accuracies this high? It's just a shame that peer review looks for high accuracies: selecting for high-accuracy models will lead to wrong models, as we've seen on top.

(I guess this is also why I so rarely hear about Kaggle-winning models making it into papers or production: Kaggle hyper-fixates on accuracies and loss, I'm sure the winning models run into real-life issues fast once they're deployed.)

In other words: Don't measure your models only *quantitatively*, also look at the *qualitative* side. Another example: With modern LLMs I've found little correlation between their behaviour in benchmarks and their behaviour with my data. The 'feel' (what Feyerabend calls the 'tacit knowledge') is important, and you can only get that by sitting down with your model and spending time to dig out the model's predictions and assumptions.  
There are no magic bullets.


P.S.: Yes I'm still mad about having a peer reviewer argue negatively against one of my papers, saying the accuracies were too low: *we worked hard on getting them this low!*
