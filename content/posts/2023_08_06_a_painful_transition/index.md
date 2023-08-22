---
title: "A painful transition"
date: 2023-08-06
description: i ramble about transitions
---

Last week I gave a fun talk at the great Western Australian Data Science Innovation Hub (WADSIH) conference. I spoke about the various mistakes I've made so far in my 'journey towards data skills'. I'll write that talk up here sometime; here I'd like to expand on one point I made.

# Mistake: Assuming my skills will just transfer

This is one of the mistakes I mentioned in my talk: I always assumed that my coding skills will easily transfer to the world outside of academia. The truth is that these skills are comparatively basic: they may have opened the door for me, nothing more.

One of the things senior academics tell computer science and bioinformatics students is that their skills translate easily into industry. I feel like that's a comforting white lie. *Yes*, academia will teach you *some* coding skills. *No*, these coding skills are nowhere near to the 'good' jobs: academics are generally not exposed to huge legacy codebases, they're generally not exposed to application architecture, data warehousing, or industry-level *anything*.

On good days, I feel like a junior programmer who can just code faster. On bad days, I feel like a junior programmer. And I've been in bioinformatics for about ten years now...

# It's not just the coding; it's the tools, too.

Industry uses many 'expensive' tools that have never made their way into academia with such illustrious names such as Tableau, Power BI, Apache Kafka, Azure, SageMaker, or Snowflake. They have not made it into academia because academic funding is precarious: every small research group has its own pot of funding. You can think of a n academic group as a start-up, not as part of a university. The university may be rich, but research groups rarely see that money. In fact research groups pay for the privilege of being part of the university: my last employer took a 60% cut of any consulting activities... All that support staff has to be paid, too.

What the precarious funding situation means is that academic research groups usually do not pay for enterprise-level solutions for *anything*. The data backup strategy is flat files on external hard-drives in desk lockers. The metadata is in Excel sheets on the Research Assistant's laptop. The work horse is the local cluster and SLURM or PBS. The knowledge base is whatever is inside the postdoc's head. The ML ecosystem is Jupyter notebooks on the same hard-drives in the desk lockers.
 
Within academia I've always seen Tableau, Kafka etc. as 'nice to have' tools. What the WADSIH conference reiterated to me is that these tools are at the core of industry here: they're not 'nice to have' tools, they're 'must-have' tools, appearing in every talk.

In 2023 these tools have become core within data-heavy industries. That means that job positions require at least a modicum of experience with these tools. Academia does not train its graduates, early career researchers, or mid-career researchers in these tools in any way. It follows from there that academia also does not pay for certifications for these tools. I can understand why some industry places treat fresh PhDs as year 0 graduates: there's just too much to teach, any other graduate will have to learn the same stuff.

I just checked: on Seek for Perth I find *zero* jobs mentioning SLURM, the HPC workload manager most academics have experience with. I find *zero* jobs mentioning snakemake or nextflow, the two main workflow systems in academia. On the other hand, I find 207 jobs mentioning Azure, Microsoft's cloud computing infrastructure, and eight jobs mentioning Kafka, Apache's data feed management system. And Seek has all the academic jobs, too!

It looks similar around Australia: 2 jobs mention snakemake/nextflow (they're the same posting), zero jobs mention SLURM, 3,498 jobs mention Azure, 182 jobs mention Kafka.

So there you have it: it's just not the coding skills anymore. Those skills are the *start*.

Edit, 22.8.2023: *Getting* the certifications for these tools as an academic is not easy, either! The 'AWS certified Solutions Architect Professional' certification, for example, recommends two years of AWS Cloud experience. How are you going to get those two years in academia?
