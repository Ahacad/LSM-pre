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
- let's try to solve it by tweaking **merge schedulers**

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
<img class="mx-auto" src="/pics/partition.png" alt="partitioned LSM-tree with Leveling Merge Policy" width="300"/>
<figcaption class="pl-78 text-xs">Partitioned LSM-tree, Leveling Merge Policy</figcaption>
</figure>

<div class="mt-4"/>

- **Partitioning**: large LSM disk component range-partitioned into multiple files for optimization
- partitioning and merge policies can be used together, currently LevelDB and RocksDB use partitioned leveling policy

- **Write Stalls**: memory speed faster than I/Os, writing to memory will be **stalled** (the *write stall* problem)

- **merges** are major cause of stalls, since components are merged multiple times, but writes only flush once
---

# LSM (Log-Structured Merge-Tree): Recap

<div class="mt-12"/>

<img class="m-auto" src="/pics/lsm-tree.jpg" alt="LSM-tree" width="250"/>

<div class="mt-8"/>

- multi-level, only add, flush and merge
- two merge policies: leveling and tiering
- one optimization: partitioning
- We'll analyze LSM-tree later

 
---

# Roadmap
<div class="mt-8"/>

<TOC class="mt-24" count="2"/>

---
title: Measuring Latency
---

# Measuring Latency: 2 phases

<div class="mt-12"/>

<img class="m-auto" src="/pics/measuring-models.png" alt="Measuring-Models" width="350"/>

<div class="mt-12"/>

- testing phase: use the closed system to measure *maximum write throughput* ($M$)

- running phase: use the open system with a 95% $M$ to see if the write latency will be stable
- previous experiments used only the closed system, but the open system is more realistic
- so we combine them both
---

# Measuring Latency: Experiment
<div class="mt-12"/>

<img class="m-auto" src="/pics/two-phase-evaluation.png" alt="Two-Phase Evaluation of bLSM" width="650"/>
<div class="mt-12"/>

- write latency = processing latency + queuing latency
- write stalls
<arrow v-click="1" x1="400" y1="265" x2="460" y2="240" color="#324f15aa" width="1" arrowSize="1" />
---

# Roadmap
<div class="mt-8"/>

<TOC class="mt-24" count="3"/>
---

# Full Merges Analyses - 1: Scheduling

<div class="mt-12">

- **global component constraint** better than local component constraint
- **process writes** as quickly as possible minimizes write latency
- **concurrent merges** matter:
  - process merges on each level concurrently
- finally, the paper proposes a <span class="font-bold text-red-400">greedy scheduler</span> for better variability performances
  - 3 schedulers for comparison: the *single-thread* scheduler, the *fair* scheduler, and the <span class="font-bold text-red-400">greedy</span> scheduler
  - a single-thread scheduler does not work concurrently
  - a *fair* scheduler allocate I/O bandwidth to all merges equally
  - a <span class="font-bold text-red-400">greedy scheduler</span> prefers the merge with smallest remaining bytes first
  - the greedy scheduler minimizes number of components
  
</div>
---

# Full Merges Analyses - 2: Variability

<img class="m-auto" src="/pics/full-running-1.png" alt="in-place vs. out-of-place" width="600"/>

- **stable write throughput** can be achieved
- the greedy scheduler does well
---

# Full Merges Analyses - 3: Global Constaint

<img class="m-auto" src="/pics/component-constraint.png" alt="in-place vs. out-of-place" width="360"/>

- **global component constraint** better than local component constraint:
  - component constraint: limit on disk componet number
  - more components = less write stalls BUT worse query performance and takes more space
  - local constraint: limit on each level
  - global constraint: limit on all components
---

# Full Merges Analyses - 4: Write Quickly

<img class="m-auto" src="/pics/write-limit.png" alt="in-place vs. out-of-place" width="360"/>

- **process writes** as quickly as possible minimizes write latency:
  - when *component constraint* violated, needs to slow down or stop writes
  - current implementations (LevelDB, RocksDB, bLSM) prefers slowdown
---

# Full Merges Analyses - 5: Query Performance Analyses

<img class="m-auto" src="/pics/query-analyses.png" alt="in-place vs. out-of-place" width="600"/>

- use Bloom filters to speed up point lookup
- greedy scheduler has better performance because it minimizes components
- write stalls (see the arrow)
<arrow v-click="1" x1="480" y1="360" x2="540" y2="350" color="#324f15aa" width="1" arrowSize="1" />
---

# Full Merges Analyses - 6: Size Ratio

<img class="m-auto" src="/pics/size-ratio.png" alt="in-place vs. out-of-place" width="360"/>

<div class="mt-12"/>

- higer size ratio means fewer merges for tiering
---

# Full Merges Analyses: Recap
<div class="mt-12"/>

- utilize **concurrency** schedulers
- a proposed **greedy scheduler** works well
- some other analyses for possible improving:
  - gloabl component constraint
  - write quickly
  - size ratio ($T$)
---

# Roadmap
<div class="mt-8"/>

<TOC class="mt-24" count="4"/>
---

# Partitioned Merges Analyses

<img class="mx-auto" src="/pics/partition.png" alt="partitioned LSM-tree with Leveling Merge Policy" width="300"/>

- single-threaded scheduler will be enough
---

# Lessons and Conclusions

<div class="mt-20"/>

- consider performance variance together with write throughput for usability
- use the new two-phase approach to evaluate the impact of write stalls
- a good scheduler can help achieve stable write throughput:
  - for full merges, the proposed **greedy scheduler**
  - for partitioned merges, 
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
---

# References

- [7] “RocksDB | A persistent key-value store,” RocksDB. http://rocksdb.org/ (accessed Jun. 05, 2021).
- [8] google/leveldb. Google, 2021. Accessed: Jun. 05, 2021. [Online]. Available: https://github.com/google/leveldb
- [9] K. Rott, intfrr/lsmtree. 2021. Accessed: Jun. 05, 2021. [Online]. Available: https://github.com/intfrr/lsmtree

- Tip: search "lsm-tree" or similar keywords on GitHub for community implementations
