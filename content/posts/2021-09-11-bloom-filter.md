---
title: "[Notes]: Bloom Filters"
description: "Notes on Bloom filters"
tags: [notes, bloom, filter, leveldb, concurrency]
date: 2021-09-11T00:00:00-08:00
logoText: "cat bloom-filter.md"
---

## Prelude

These are just some of my notes and references on Bloom filters while doing research for building an
LSM tree based database.

## Notes

- A Bloom filter is a probablistic data structure that allows checks for set membership. These
  checks for set membership can return false positives but will never produce false negatives. More
  clearly: "if a negative match is returned, the element is guaranteed not to be a member of the
  set" (Alex Petrov - Database Internals).

- Used to lower I/O costs incurred by the tiered structure of LSM-trees

## Useful references

- [Alex Petrov - Database Internals](https://databass.dev)
- [Jamie Talbot - What are Bloom filters?](https://blog.medium.com/what-are-bloom-filters-1ec2a50c68ff)
- [Onat - Let's Implement a Bloom Filter](https://onatm.dev/2020/08/10/let-s-implement-a-bloom-filter/)
  - [Github - plum - A Rust implementation of a Bloom filter](https://github.com/distrentic/plum)
- [Niv Dayan, et al. - Optimal Bloom Filters and Adaptive Merging for LSM-Trees](https://nivdayan.github.io/monkey-journal.pdf)
- [Kirsch and Mitzenmacher - Building a Better Bloom Filter](https://www.eecs.harvard.edu/~michaelm/postscripts/tr-02-05.pdf)
- Issues with Bloom filters in RocksDB and LevelDB. A lot of great context and information double
  hashing schemes.
  - [Github Issue - Bloom filters using a flawed double hashing scheme](https://github.com/facebook/rocksdb/issues/4120)
  - [Peter Dillinger - Adaptive Approximate State Storage](http://peterd.org/pcd-diss.pdf)
  - [Peter C. Dillinger and Panagiotis Manolios - Bloom Filters in Probabilistic Verification](https://www.ccs.neu.edu/home/pete/pub/bloom-filters-verification.pdf)
  - [Paulo Sergio Almeida - A Case for Partitioned Bloom Filters](https://arxiv.org/pdf/2009.11789.pdf)
