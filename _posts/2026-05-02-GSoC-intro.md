---
layout: default
title: "GSoC 2026: Introduction"
date: 2026-05-02
---

<a href="https://summerofcode.withgoogle.com/">
  <img src="{{ '/assets/images/gsoclogo.png' | relative_url }}" alt="GSoC" style="width:500px;">
</a>


Hi! I’m Kunj Sinha

A big thanks to [MDAnalysis](https://www.mdanalysis.org/) and [WESTPA](https://westpa.github.io/westpa/) for accepting my proposal for [GSoC 2026](https://summerofcode.withgoogle.com/programs/2026/projects/VJMrp2UW).

I'll be mentored by [Jeremy Leung](https://github.com/jeremyleung521), [Lilian Chong](https://github.com/ltchong) and [Nilay Verma](https://github.com/nilay-v3rma).

I will be working on the project "Interface for post-simulation analysis (“crawling”) of WESTPA simulations".

Summary: <br>
This project aims to implement a WESTPAParser and a WESTPAReader in WESTPA which will expose WESTPA’s HDF5 Framework simulation data as a standard MDAnalysis Universe. Currently, users are required to write extensive boilerplate code to manually navigate the HDF5 simulation data via WESTPA’s w_crawl before any structural observables can be computed. This project replaces that manual process by allowing users to obtain an MDAnalysis Universe directly from a west.h5 simulation file. This native integration will be accessible through both a Python API and a new w_mdacrawl CLI tool, serving as a high performance drop-in replacement for existing workflows. The project will also allow users to use MDAnalysis’s AnalysisBase backend to perform analysis using parallelization. Furthermore, the project will also implement a method to save the resulting analysis results back into the HDF5 framework as auxdata. This will ensure that any computed properties remain compatible with the broader WESTPA ecosystem for use in future simulations or downstream analysis tools like w_ipa.

I will be posting updates of my progress here.