---
layout: post
title: Calculating MGas/s on a Geth Node for Observability
description: Calculating and Visualizing MGas/s on a Geth Node
date: 2024-11-03 12:00:00 -0000
tags: observability ethereum devops grafana loki
---

## Calculating MGas/s on a Geth Node for Observability

As part of my work as Platforms Engineer at OP Labs I've been working on increasing the observability of our systems. A key metric of these systems is how quickly our op-geth nodes are able to process blocks in a blockchain network. The metric to monitor this type of performance is known as Gas per Second, or MGas/s since blocks typically have in the order of millions of gas in them.

Please note that this method of measuring MGas is based on my approach. Feel free to provide any suggestions or improvements.

---

### Observability Tools and Procedures

Tools utilized for MGas measurement

* Grafana
* Loki
* Geth

### Typical Geth Log Example

Whenever geth generates a block, it emits a log similar to the following:

```logfmt
t=2024-11-03T17:52:35+0000 lvl=info msg="Imported new potential chain segment" number=127528789 hash=0x72d243d273fcb82e05a8891c19da93e6a54fd313a78826baac3a86edee41fb50 blocks=1 txs=17 mgas=5.157262 elapsed=91.736ms mgasps=56.21803667723872 snapdiffs="3.62 MiB" triedirty="0.00 B"
```

Key fields to consider include

* t (timestamp)
* mgas
  * The amount of gas in the block, measured in millions
* mgasps
  * The Millons of gas per second, given the elapsed time for importing this chain segment.

### Loki Query for 10-Minute Rolling Average of MGas Per Second

```
avg(sum_over_time({pod="op-geth", filter_1="value_1", filter_2="value_2"} |= `Imported new potential chain segment` | logfmt | unwrap mgasps [10m])) 
```

#### Breakdown of each section

* The inner section will filter for the log lines which contain the string `Imported new potential chain segment` 
  * Note the filters will need to be adjusted for each individual monitoring setup to target an individual geth pod 
* logfmt tells grafana we are using the logfmt logging format, and specifies how we can extract specific values from the logs
* unwrap mgasps extracts the mgasps logging field

```logql
{pod="op-geth", filter_1="value_1", filter_2="value_2"} |= `Imported new potential chain segment` | logfmt | unwrap mgasps 
```

#### Outer Section

```logql
avg(sum_over_time(<<>>> [10m])) 
```

* The avg(...) function calculates the average of mgasps over a 10-minute period by summing the values using sum_over_time.
