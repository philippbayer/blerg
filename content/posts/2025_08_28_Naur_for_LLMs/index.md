---
title: "How I use Claude Code - or, AI-coding as normal tech - or, 1985 is back"
date: 2025-08-28
description: Naur helps us build with Claude Code
---

Claude Code has become a core tool in my tool-kit. But, it suffers from the same problems any LLM suffers from: unpredictable outcomes. I've seen Claude Code write working functions that would've taken me days of work, and I've seen Claude Code happily delete load-bearing functions. You never know which outcome you'll get!

I've been thinking of strategies to counteract this behaviour, and I think I've come to *some* useful outcomes.

I work on these assumptions:
- LLMs 'know' nothing. LLMs cannot understand complex code-bases, even if they fit into the context window. 
- LLMs cannot think. We are no closer to AGI than we were five years ago.
- LLMs are 'averagers', giving you the most likely text token next.
(you might disagree on any or all of the above - but I have seen no *consistent* results from research to the contrary)

It follows from these assumptions:
- LLMs are great at boilerplate code, since boilerplate requires no thinking and works with 'averagers'. (By boilerplate I mean code that needs no thinking to write, boilerplate as the unoriginal implementation of existing thought.)
- You have to all of the thinking before you start Claude Code.

Perhaps some examples where I used Claude Code successfully elucidate this approach:

- 'I have written all the code, I want a decent test coverage now'. Claude Code read the code and got me to ~70% test coverage, the remaining 30% of test coverage required thinking and knowing what the code does.
- 'I have written all the code in R, now I want that in a Shiny dashboard'. All the functionality is there, Claude Code will add the required side-bars and sliders.
- 'I ran a bioinformatics analysis and have written all command-line steps in a text file, now I want that in a Nextflow pipeline'. All the functions are there and the order in my text file implies how they relate to each other, Claude Code will add the necessary Nextflow boilerplate.
- 'I have written all the code in Python, please make it production-ready'. Claude Code will add type hints via `typing` and adds better logging and error-handling, but not change how the code itself works. It does not need to understand what the code does.

The common theme is 'Claude Code adds boilerplate on top of a human's thinking.'

Or, counter-examples, where Claude flails around producing nothing:
- The Claude Code example on their github is 'do something awesome'. I've tried this in my RStudio repos and it always added the most basic nonsense steps that added nothing to the overall analysis.
- 'Build me a CV webpage based on my CV pdf'. The resulting webpages are... OK? But basic, because I did not carry out the design thinking, it just replicated the layout in the CV pdf. 
- 'I have half of my analysis steps in a text file, please suggest some further steps based on my research question'. Claude Code would suggest steps that do not lead to answers, it only suggest relatively common steps that are core to many bioinformatics pipelines. Again the 'averager', but it's called research because I'm doing new things. The suggestions do not lead closer to my research question.

The common theme is 'I tried to make it think and see what I got for it: nothing.' But can we *predictably* work with Claude Code and similar tools?

# Programming as Theory Building

Peter Naur wrote this in 1985, 'Programming as Theory Building'. 

> In terms of Ryle’s notion of theory, what has to be built by the programmer is a theory of how certain affairs of the world will be handled by, or supported by, a computer program. On the Theory Building View of programming the theory built by the programmers has primacy over such other products as program texts, user documentation, and additional documentation such as specifications.
{cite=https://gwern.net/doc/cs/algorithm/1985-naur.pdf caption='Naur 1985, Programming as Theory Building'}

Naur's dea was that most of the time writing code is figuring out that Theory - that's why it's far easier writing a program the second time around, you have the Theory now, the second time around you're just writing boilerplate.

As we assume that LLMs cannot think and cannot know, we assume that LLMs cannot build Naur's theories of how the program interacts with the world. You, the human, have to do that for the machine. In my above successful examples I had built Naur's theory before I asked Claude Code. In the failed examples, I tried to get the LLM to come up with the Theory. Since there's no thought, there's no Theory.

I now always design the Theory of the program in detail first, then the LLM will have to write only boilerplate. That results in the least surprises and least random outcomes.

CLAUDE.md brings Naur's theory of the program to the table: a standardised way of telling the LLM how the program interacts with the world. However, it's incredibly wordy and hard to predict which parts of the text will help where, or at all. I can imagine a framework like test-driven development (TTD), where you first write down all the assumptions about how your code interacts with the world in a standardised, concise way. The LLM will read these 'tests' and not need to think, as the details in the tests will describe Naur's theory of the program<->world interaction.

I still use Claude Code every day, and in those cases where I have done the thinking, Claude Code has saved me weeks of work. It even has made my work more joyful: I get to do the fun thinking and problem-solving, Claude Code then implements the boring boilerplate.

A few other tips that helped me:
- sometimes, copy-pasting the relevant function the 'old school way' into any chatbot interface gets me to results faster. I've seen Claude Code launch >30 sub-agents to replace library calls, one sub-agent per library; a regular chatbot does that in one call (slightly less predictably).
- Restarting the entire session can help, LLMs dig themselves into holes. This will probably get worse with longer context-windows.
- Conversely, `claude -r` (resume) and the history function often help me save time when I'm getting sick of explaining the system and the code's background for the tenth time.
- And of course, always use git!! Claude Code sometimes shows you dozens of lines of change and sneaks in deletion of an important line, making you dig through your git history for the deleted line.
