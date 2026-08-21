---
layout: default
title: "GSoC 2026: Final Submission"
date: 2026-08-21
---

# GSoC 2026: MDACrawl:

As a part of Google Summer of Code 2026, I contributed to [MDAnalysis](https://www.mdanalysis.org/) and [WESTPA](https://westpa.github.io/westpa/) on the project 
["Interface for post-simulation analysis (“crawling”) of WESTPA simulations"](https://summerofcode.withgoogle.com/programs/2026/projects/VJMrp2UW). My mentors for this project were [Jeremy Leung](https://github.com/jeremyleung521), [Lillian Chong](https://github.com/ltchong) and [Nilay Verma](https://github.com/nilay-v3rma).
<br>

## The Original Goal
The original goal for this project was to make post processing of simulation data far easier and be able to analyze and compute observables without much effort. Since WESTPA (unlike conventional [MD](https://en.wikipedia.org/wiki/Molecular_dynamics) engines) runs hundreds of short parallel simulations at once, it splits simulation data in multiple places. The original way to process this data was to use the [w_crawl](https://github.com/westpa/westpa/wiki/man:w_crawl) tool however it required writing and testing custom Python code to get it right. The task here was to simplify the process for users who have already saved their trajectory data with WESTPA's HDF5 Trajectory Storage Framework ("HDF5 Framework"), just by loading their [west.h5](https://github.com/westpa/westpa/wiki/HDF5-File-Organization-of-Simulation-Data#user-content-Overall_structure_of_westh5) file. This also allows for connectivity with MDAnalysis by converting the `west.h5` data into an MDAnalysis [Universe](https://userguide.mdanalysis.org/stable/universe.html). MDAnalysis provides an ecosystem of ready to use tools such as RMSD, RMSF, PCA and more, accessible through a clean API. However MDAnalysis had no way to read the WESTPA HDF5 format so none of these tools were directly usable on WESTPA simulation data. The topology needed to construct a full MDAnalysis Universe was already embedded inside the WESTPA trajectory files. Connecting these two would allow users to use all of MDAnalysis's tools directly on WESTPA simulation data.
<br>


## What I did
I initially split this project into different phases that I would tackle one by one. <br>
First was *WESTPAParser*. *WESTPAParser*'s main job was to read the topology which was already embedded in the `west.h5` files, detect its format, and extract atom names, residues, elements, bond pairs, etc. It created flat arrays for each data and constructed a MDAnalysis [TopologyAttr](https://userguide.mdanalysis.org/1.1.1/topology_system.html) object. This data is parsed exactly once when the Universe is created and then reused for every subsequent frame read. However, this only contained the static parts of the simulation.
<br>

In order to actually read the dynamic data i.e., the actual coordinates of all the atoms and other frame specific auxdata that is saved during the simulation run, I had to find a way to convert WESTPA parallel simulation run data into flat MDAnalysis data. This was the second phase of the project - *WESTPAReader*. *WESTPAReader* was essentially just going through every iteration of the simulation and logging all the data into a "frame index". This frame index would store the trajectory frames in the correct order as they may not be stored sequentially and maybe stored on first come basis. One of the main benefits of this crawler is retaining WESTPA-specific data (like statistical weights, segment IDs, and iteration numbers) directly alongside the physical coordinates. Rather than forcing the researcher to cross-reference two different data structures, I injected the metadata directly into the MDAnalysis [timestep](https://docs.mdanalysis.org/2.0.0/documentation_pages/coordinates/init.html#timesteps) dictionary. However while testing, I noticed all the frames had the same metadata. This was because `ts.data` was retaining its values from the previous frame and was polluting every subsequent frame. Hence it was necessary to clear it for every frame. There was also a need to cache file I/O. If the reader opened and closed an HDF5 file for every single frame, the I/O overhead would bring the analysis to a crawl. With this done, *WESTPAParser* and *WESTPAReader* alone were able to run standard MDAnalysis analysis tools on any `west.h5` file.
<br>

The project was far from over though. MDAnalysis has a way to [run analysis in parallel](https://www.mdanalysis.org/2024/01/18/gsoc2023_marinegor/) by specifying the number of cores. If MDACrawl could utilize this, it would make running analysis on large simulations much faster. For this, I added a `__getstate__` and `__setstate__` function. When MDAnalysis decides to split the workload, it uses Python's `pickle` library to package up your reader and send it to the workers. Right before this happens, it call `__getstate__` which intercepts this process, explicitly closes all open HDF5 file handles (self._current_h5 and self._h5), and sets their variables to None. This strips away the "unpicklable" OS-level pointers, allowing the pure data (like the frame index) to transmit safely. When the worker process receives the package and wakes up, it calls `__setstate__`. The reader uses this moment to cleanly reconstruct itself in the new memory space. It re-opens the main `west.h5` file restores all the attributes that sometimes gets wiped out during multiprocessing. With this, multiprocessing was possible now. This marked the completion of the third phase of the project.
<br>

The fourth phase of the project was all about auxdata. WESTPA simulations usually also contain some user defined auxilary datasets that are important for certain analysis. This needed to be imported into the Universe as well. It was pretty straightforward. If any auxdata exists in the directory, it safely loads it into `ts.data`. This was only importing the data. In order to form a closed loop workflow in which users could transmit data between the two tools efficiently, there also needed a way to send auxdata back into the `west.h5` files. For this, I created a `save_to_west_h5()` tool. When you run an analysis tool like rms.RMSD, MDAnalysis returns a flat, 1D array of results. The saving function's job is to "un-flatten" that list and safely pack it back into the WESTPA HDF5 grid. This was done in 3 parts - mapping, validation and actual writing. To figure out where a flat result belongs in the HDF5 hierarchy, the code uses the Universe's frame_index. It then builds a massive nested dictionary and completely reconstructs the branching tree logic of WESTPA. The validation logic scans the HDF5 file based on the planned writes in `data_map`. If the dataset already exists and you didn't pass `overwrite=True`, it intentionally aborts before any writing begins. This prevents "partial writes" where half your file gets updated before an error occurs. Finally, the code loops through the `data_map` to build the physical datasets. It asks the HDF5 file how many segments exist in the iteration and creates an empty grid of zeros. It loops through the `data_map` and slots the values directly into the correct grid coordinates and then pushes the completed grid into the HDF5 file.
<br>

The fifth and final phase of the project was wrapping everything done so far into a CLI tool since not everyone would want to write code to run their analysis. Using this CLI tool, users can directly import their `west.h5` file, select standard MDAnalysis analysis tools or use custom analysis tools, extract a certain column from their dataset, select atoms and run the analysis and get the results all in the terminal itself. In the backend, *WESTPAParser* create the Universe, and the CLI passes the imported analysis tool to *WESTPAReader* which runs it on the Universe. Saving back uses `save_to_west_h5()`. With all this in place, the entire MDACrawl tool was complete. It sucessfully fulfills all the needs of the original goal along with prospects for future enhancements.
<br>

## Test Cases
I also implemented extensive test cases which test each part of the tool using simulation data from the WESTPA's Tutorial 7.5. <br>
List of test cases added:
- test_parser_atom_count()
- test_parser_residue_names()
- test_parser_masses()
- test_parser_bonds()
- test_parser_segments()
- test_parser_elements()
- test_parser_bond_connectivity()
- test_reader_frame_count()
- test_reader_coordinates()
- test_reader_metadata()
- test_reader_is_picklable()
- test_parallel_rmsd_matches_serial()
- test_reader_pbc_dimensions()
- test_save_auxdata_roundtrip()
- test_saved_auxdata_compatability()
- test_cli_execution()
- test_invalid_module_import()


## Where is the code
The code for this project is done entirely in one Pull Request - [GSoC 2026: Interface for post-simulation analysis (“crawling”) of WESTPA simulations](https://github.com/westpa/westpa/pull/594).
*WESTPAParser*, *WESTPAReader*, `save_to_west_h5()` all live in `src/westpa/core/mdanalysis_core.py`. The w_mdacrawl CLI tool lives in `src/westpa/cli/tools/w_mdacrawl.py`. Test cases for the parser and reader are in `tests/test_mdacrawl.py` and for the CLI tool, `tests/test_tools/test_w_mdacrawl.py`. I also wrote documentation about the tool which will be hosted on the WESTPA docs once the PR is merged.

## Some more words
All in all, this was a very productive summer and I enjoyed working with WESTPA and MDAnalysis very much. I would like to thank my mentors Jeremy Leung, Lillian Chong and Nilay Verma for their unending support. They were very warm and welcoming to me in the beginning when I was just an open source beginner and they helped me grow into more confident developer and a capable contributor to the community.
