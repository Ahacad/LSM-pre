---
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
title:
---

# On Performance Stability in LSM-based Storage Systems

<div class="text-gray-400">Check the website <a href="https://ahacad.github.io/LSM-pre">https://ahacad.github.io/LSM-pre</a> for online slides.</div>

<a href="https://ahacad.github.io/LSM-pre" target="_blank" alt="GitHub"
  class="abs-br m-6 text-xl icon-btn opacity-50 !border-none !hover:text-white">
  <carbon-logo-github />
</a>
---

# A look ahead

<span class="text-gray-400">*On Performance Stability in LSM-based Storage Systems*</span>

<div class="grid grid-cols-2">
<img class="m-auto" src="/pics/lsm-tree.jpg" alt="LSM-tree" width="250"/>
<img class="m-auto" src="/pics/stalls-rocksdb.png" alt="Write stalls in RocksDB" width="250"/>
</div>

<div class="mt-8" />

- Memory FASTER THAN disk
- **writes stalls** happen when manipulating disks, and it affects usability
- let's try to solve it by tweaking **merge shedulers**

<arrow v-click="1" x1="600" y1="220" x2="698" y2="190" color="#324f15aa" width="1" arrowSize="1" />

---

# Table of contents
<div class="mt-8"/>

<TOC class="mt-24" />
---

# Roadmap
<div class="mt-8"/>

<TOC class="mt-24" count="1"/>
---

---
title: Introduction to LSM-tree
---

<img class="m-auto" src="/pics/database-storage-engines.png" alt="in-place vs. out-of-place" width="800"/>

---

# LSM (Log-Structured Merge-Tree) - 1: Basics

<div class="mt-12"/>

<div class="grid grid-cols-2">
<figure>
<img class="m-auto" src="/pics/inplace-outofplace.png" alt="in-place vs. out-of-place" width="200"/>
<figcaption class="pl-50 text-xs">Fig. 1</figcaption>
</figure>
<figure>
<img src="/pics/lsm-tree-original.png" alt="LSM-tree Original design" width="250"/>
<figcaption class="pl-30 text-xs">Fig. 2</figcaption>
</figure>
</div>

- unlike B+ tree (in-place update) which overwrites old entries, we store updates into new locations <span class="text-gray-400">(Fig. 1)</span>
- only append operations (insert OR update), delete = anti-matter entry, like writing everything into a "log"

- **buffers** all writes in memory ($C_0$), and **flushes** them to disk and **merges** later<span class="text-gray-400">(Fig. 2)</span>

- quick write (due to sequential I/Os), but slow read
---

# LSM (Log-Structured Merge-Tree) - 2: Merging

<div class="mt-12"/>
<div class="grid grid-cols-2">
<div>

  - today's LSM-tree use <span class="font-bold">merging</span> to reduce components examined when querying
  - two merging policies: leveling merge policy & tiering merge policy
  - <span class="font-bold">leveling</span>: 1 component, merged with Level $i-1$ until big enough and then merged into Level $i+1$
  - <span class="font-bold">tiering</span>: T components at Level $i$, then merged together to Level $i+1$

</div>

<figure class="m-auto">
<img src="/pics/policies.png" alt="in-place vs. out-of-place" width="300"/>
<figcaption class="pl-34 text-xs">Fig. 1</figcaption>
</figure>

</div>
---

# LSM (Log-Structured Merge-Tree) - 3: Partitioning

<div class="mt-12"/>

<figure class="m-auto">
<img src="/pics/partition.png" alt="partitioned LSM-tree with Leveling Merge Policy" width="300"/>
<figcaption class="pl-8 text-xs">Partitioned LSM-tree, Leveling Merge Policy</figcaption>
</figure>

<div class="mt-4"/>

- **Partitioning**: large LSM disk component range-partitioned into multiple files for optimization
- partitioning and merge policies can be used together, currently LevelDB and RocksDB use partitioned leveling policy

- **Write Stalls**: memory speed faster than I/Os, writing to memory will be **stalled** (the *write stall* problem)

- **merges** are major cause of stalls, since components are merged multiple times, but writes only flush once

---

# Roadmap
<div class="mt-8"/>

<TOC class="mt-24" count="2"/>

---

---
title: Measuring Latency
---

# Measuring Latency: 2 phases

<div class="mt-12"/>

<img class="m-auto" src="/pics/measuring-models.png" alt="Measuring-Models" width="350"/>

<div class="mt-4"/>

- testing phase: use the closed system to measure *maximum write throughput* ($M$)

- running phase: use the open system with a 95% $M$ to see if the write latency will be stable

---

# Measuring Latency
<div class="mt-12"/>

<img class="m-auto" src="/pics/two-phase-evaluation.png" alt="Two-Phase Evaluation of bLSM" width="650"/>
<div class="mt-12"/>

- write latency = processing latency + queuing latency

---

# Roadmap
<div class="mt-8"/>

<TOC class="mt-24" count="3"/>
---

# Full Merges Analyses - 1: Scheduling

- **global component constraint** better than local component constraint:
  - limit on disk componet number
  - more component = less write stalls BUT worse query performance
- **process writes** as quickly as possible minimizes latencies
- **concurrent merges** matter
- finally, the paper proposes a <span class="font-bold text-red-400">greedy scheduler</span>

---

# Full Merges Analyses - 2: Experiments

---

# Merge Shedulers 2 - partitioned merges

---

# Roadmap
<div class="mt-8"/>

<TOC class="mt-24" count="4"/>

---

# Lessons and Conclusions

- the two-phase approach to evaluate the impact of write stalls
- design choices for LSM merge schedulers: full-merges and partitioned merges
- consider **performance variance** and **write throughput** together

---

# Roadmap
<div class="mt-8"/>

<TOC class="mt-24" count="5"/>

---

# References

- [1]C. Luo and M. J. Carey, “On performance stability in LSM-based storage systems,” Proc. VLDB Endow., vol. 13, no. 4, pp. 449–462, Dec. 2019, doi: 10.14778/3372716.3372719.
- [2] C. Luo and M. J. Carey, “LSM-based Storage Techniques: A Survey,” The VLDB Journal, vol. 29, no. 1, pp. 393–418, Jan. 2020, doi: 10.1007/s00778-019-00555-y.
- [3] P. O’Neil, E. Cheng, D. Gawlick, and E. O’Neil, “The log-structured merge-tree (LSM-tree),” Acta Informatica, vol. 33, no. 4, pp. 351–385, Jun. 1996, doi: 10.1007/s002360050048.
- [4] R. Sears and R. Ramakrishnan, “bLSM: a general purpose log structured merge tree,” in Proceedings of the 2012 international conference on Management of Data - SIGMOD ’12, Scottsdale, Arizona, USA, 2012, p. 217. doi: 10.1145/2213836.2213862.
 

- [5] P. Guo, “Log Structured Merge Tree.” [Online]. Available: https://lrita.github.io/images/posts/database/lsmtree-170129180333.pdf
- [6] “The Log-Structured Merge-Tree (LSM Tree) | the morning paper.” https://blog.acolyer.org/2014/11/26/the-log-structured-merge-tree-lsm-tree/ (accessed May 30, 2021).
