---
layout: default
title: "GSoC 2026: Final Submission"
date: 2026-08-20
---

# GSoC 2026: MDACrawl:

As a part of Google Summer of Code 2026, I contributed to [WESTPA](https://westpa.github.io/westpa/) and [MDAnalysis](https://www.mdanalysis.org/) on the project 
["Interface for post-simulation analysis (“crawling”) of WESTPA simulations"](https://summerofcode.withgoogle.com/programs/2026/projects/VJMrp2UW). My mentors for this project were [Jeremy Leung](https://github.com/jeremyleung521), [Lillian Chong](https://github.com/ltchong) and [Nilay Verma](https://github.com/nilay-v3rma).
<br>

The original goal for this project was to make post processing of simulation data far easier and be able to analyze and compute observables without much effort. Since WESTPA (unlike conventional [MD](https://en.wikipedia.org/wiki/Molecular_dynamics) engines) runs hundreds of short parallel simulations at once, it splits simulation data in multiple places. The original way to process this data was to use the [w_crawl](https://github.com/westpa/westpa/wiki/man:w_crawl) tool however it required writing and testing custom Python code to get it right. The task here was to simplify the process for users who have already saved their trajectory data with WESTPA's HDF5 Trajectory Storage Framework ("HDF5 Framework"), just by loading their [west.h5](https://github.com/westpa/westpa/wiki/HDF5-File-Organization-of-Simulation-Data#user-content-Overall_structure_of_westh5) file. This also allows for connectivity with MDAnalysis by converting the `west.h5` data into an MDAnalysis [Universe](https://userguide.mdanalysis.org/stable/universe.html). MDAnalysis provides an ecosystem of ready to use tools such as RMSD, RMSF, PCA and more, accessible through a clean API. However MDAnalysis had no way to read the WESTPA HDF5 format so none of these tools were directly usable on WESTPA simulation data. The topology needed to construct a full MDAnalysis Universe was already embedded inside the WESTPA trajectory files. Connecting these two would allow users to use all of MDAnalysis's tools directly on WESTPA simulation data.
<br>




