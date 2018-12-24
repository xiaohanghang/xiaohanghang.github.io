---
layout:     post
title:      multiple-Speaker source Separation
date:       2018-12-23 14:26:00
summary:    multiple-Speaker Separation method
categories: jekyll pixyll
---

### Memory Time in LSTMs for multi-Speaker source Separation





### Improving Source Separation via multi-Speaker Representations
It was even shown that explicitly presenting speaker representations at the network inputs could slightly improve MSSS accuracy.

It is expected that heavy memory leakage will hinder the network to retain this representation, making this tasks well suited to study long term
dependencies.

challenge
- the network is to be adapted to more than a single person.
- There is no direct way to extract an i-vector for all speakers,since they are speaking simultaneously.


The idea is to first perform blind source Separation,then extract i-vectors on the estimated sources and use these to adapt a second network and so on.


DC method
> by mapping each time-frequency bin to an embedding space such that bins belonging to a different speaker are close together and those belonging to a different speaker are further apart. When assuming only one speaker is active per bin,an unsupervised clustering mechaism,like K-means,can be used to group bins and estimated a binary mask per cluster or target speaker.  
