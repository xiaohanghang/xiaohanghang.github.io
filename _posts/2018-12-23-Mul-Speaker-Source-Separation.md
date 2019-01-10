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


#### Deep Clustering method
In deep clustering,a neural network is trained to assign an embedding vector to each element of a multi-dimensional signal,such that clusering the embeddings yields a desired segmentation of the signal.

permutation problem:there are multiple valid output masks that differ only by a permutation of the order of source,so a global decision is needed to choose a permutation.Deep clustering solves the permutation problem by framing mask estimation as a clustering problem.

> by mapping each time-frequency bin to an embedding space such that bins belonging to a different speaker are close together and those belonging to a different speaker are further apart. When assuming only one speaker is active per bin,an unsupervised clustering mechaism,like K-means,can be used to group bins and estimated a binary mask per cluster or target speaker.

- V = f(x) 使用神经网络， The embedding vector is normalized to unit length,so that |v|^2 = 1,Define an(N*C)-dimensional target matrix Y,With C the number of target speakers, so that y
