---
layout: default
title: "GSoC 2026: Introduction"
date: 2026-05-13
---

<a href="https://summerofcode.withgoogle.com/">
  <img src="{{ '/assets/images/gsoclogo.png' | relative_url }}" alt="GSoC" style="width:500px;">
</a>


Hi! I’m Kunj Sinha

A big thanks to [MDAnalysis](https://www.mdanalysis.org/) and [WESTPA](https://westpa.github.io/westpa/) for accepting my proposal for [Google Summer of Code 2026](https://summerofcode.withgoogle.com/).

I'll be mentored by [Jeremy Leung](https://github.com/jeremyleung521), [Lillian Chong](https://github.com/ltchong) and [Nilay Verma](https://github.com/nilay-v3rma).

I will be working on the project ["Interface for post-simulation analysis (“crawling”) of WESTPA simulations"](https://summerofcode.withgoogle.com/programs/2026/projects/VJMrp2UW).

Summary: <br>
This project aims to implement a WESTPAParser and a WESTPAReader in WESTPA which will expose WESTPA’s [HDF5 Framework](https://westpa.readthedocs.io/en/latest/users_guide/hdf5.html) simulation data as a standard MDAnalysis [Universe](https://userguide.mdanalysis.org/stable/universe.html). Currently, users are required to write extensive boilerplate code to manually navigate the HDF5 simulation data via WESTPA’s [w_crawl](https://westpa.readthedocs.io/en/latest/documentation/cli/w_crawl.html) before any structural observables can be computed. This project replaces that manual process by allowing users to obtain an MDAnalysis Universe directly from a west.h5 simulation file. This native integration will be accessible through both a Python API and a new w_mdacrawl CLI tool, serving as a high performance drop-in replacement for existing workflows. The project will also allow users to use MDAnalysis’s AnalysisBase backend to perform analysis using parallelization. Furthermore, the project will also implement a method to save the resulting analysis results back into the HDF5 framework as auxdata. This will ensure that any computed properties remain compatible with the broader WESTPA ecosystem for use in future simulations or downstream analysis tools like [w_ipa](https://westpa.readthedocs.io/en/latest/users_guide/command_line_tools/w_ipa.html).

<br>

### Rough Timeline

**May 1 - May 24**
* Interact with mentors and setup proper communication times.
* Discuss and make a detailed roadmap for project outcomes.
* Get involved with the community.

**May 25 - June 1**
* Setup developer environment and start basic `h5py` `traj_segs`/reads.
* Register format ‘WESTPA’ and add entry points to `pyproject.toml`.

**June 1 - June 7**
* Build `WESTPAParser` skeleton, implement `__init__`.
* Build `parse()` with configuration detection.

**June 8 - June 14**
* Return fully populated Topology object after detecting config.
* Add parser test cases and verify that it is passing without issues.

**June 15 - June 21**
* Build `WESTPAReader` skeleton and flat frame index.
* Implement `__init__` using `westpa.analysis` API to build flat `frame_index` list.
* Implement `n_frames` and `n_atoms` properties.

**June 22 - June 28**
* Create the `Timestep` object and verify that `len(u.trajectory)` returns the correct frame count.
* Implement `_read_frame()`. Add filter to verify the sequence of trajectory frames using the pointer dataset.
* Resolve `frame_index[i]` and open `traj_segs` file with iteration level caching.
* Read coordinates via `h5py` hyperslab, set `ts.positions`.

**June 29 - July 5**
* Write coordinate correctness tests via direct `h5py` reads for multiple frames across multiple iterations.
* `ts.metadata` and reader completion. Populate weight, `pcoord`, iteration, walker, `parent_id`, `endpoint_type` on every `_read_frame` call.
* Implement `_reopen()` and `close()`.

**July 6 - July 12**
* Write metadata tests by asserting against `west.h5` `seg_index` fields directly.
* Expose any existing `auxdata` datasets in `ts.data`.
* Run RMSD and `RadiusOfGyration` directly on the constructed Universe.
* Submit work for Midterm Evaluation.

**July 13 - July 19**
* Verify RMSD and `RadiusOfGyration` results against Tutorial 7.5 reference outputs.
* Setup skeleton for parallelization support.
* Document completed work so far.

**July 20 - July 26**
* Implement parallelization by adding `__getstate__` and `__setstate__` for `h5py` handle management.
* Test that pickling the reader works without `TypeError`.
* Run analysis and check whether results are identical to a serial run. Test with `n_workers=4`.
* Implement `save_to_west_h5()` completely and handle overwrite cases.

**July 27 - August 2**
* Verify `w_ipa` compatibility.
* Verify written `auxdata` is readable without errors.

**August 3 - August 9**
* Implement `w_mdacrawl` CLI tool via `argparse` and setup the entire pipeline from Universe construction to `save_to_west_h5()`.
* Register as console script entry point.
* Verify the entire project works as intended with all the test cases passing.

**August 10 - August 16**
* Create Jupyter notebook tutorial by reproducing the entire Tutorial 7.5 `w_crawl` workflow.
* Write documentation for all three public APIs - `WESTPAParser`, `WESTPAReader`, `save_to_west_h5()`.
* Project Completion and Submission
* Work on future enhancements if time permits.


I will be posting updates of my progress here.