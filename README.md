# chignolin-workflows
AdaptiveMD workflows for the chignolin system

Adaptive Sampling requires NO *a priori* knowledge of your system's states!\
It is also an unbiased method, so in principle (unlike almost all other workflow-based advanced
MD methods) you can run this without much preparation beyond what it would take to run
your first MD simulation as a complete beginner. 

You only need to prepare the simulation system, in this case for OpenMM. Knowledge of course
would most likely help your resource use efficiency, along with clever and/or careful choices
of many different parameters required in an adaptive sampling workflow. 

If you want to learn more about (exeucuting) adaptive sampling workflows,
you can use this as a operationally-oriented tutorial that explains what is
happening in the order we do it from experimental design to workflow execution
to final analsys.

### Toolkit
There are a number of high level dependencies for the listed components that should install
by following the instructions. Additionally, you will want a hardware resource with GPUs
to do this work in a reasonable amount of time. If you replace chignolin with your own system,
or desire to scale up, access to dozens of GPUs is desirable.
 - Python from Miniconda
 - AdaptiveMD, PyEMMA, OpenMM

## A reproducible set of workflow instructions for comparative analysis
### `workflows` Directory
The name of folders in the `workflows` directory indicate the layout of MD simulations in the workflow.\
`< system name >-< phase # >-< features >-< workflow layout >`

##### Phases
In each case of a multiple round workflow, the restarting frames are adaptively sampled from existing data.
Here a simple 2-phase formalism is used, where in phase 1 denoted by `p1` the objective is to
explore the conformation space of the protein as rapidly as possible.

The adaptive sampling algorithm "explore macrostates" is used from the AdaptiveMD package
where a large number of macrostates (~25) are estimated from the existing data via this analysis
pipeline implemented with calls to PyEMMA:

**data** --*featurize*-->\
**feature trajectories** --*TICA*-->\
**reduced dims** --*cluster(k-means)*-->\
**microstates** --*estimate transitions*-->\
**MSM** --*cluster(spectral, PCCA+)*-->\
**macrostates**

Inverse count sampling is used to select macrostates for restarting by weighting each macrostate
inverse to its count of existing frames, so that less-visited macrostates are more likely to be
sampled for restarting.

In phase 2 we want to estimate the full system kinetics as efficiently as possible.
An identical analysis is performed up to the estimation of an MSM, however in this phase we seek
to distribute restart states along the slowest process(es) instead of in the least visited states.

##### Features
Features are (a reduced set of) aspects/measurements from the raw data that we actually model. 
It could be the full raw dataset, which would take a prohibitively long time. In our cases we use
either the inverse of all inter-residue **Cα**-**Cα** distances, or these distances
along with further conformational information from (backbone) φ, ψ, and (sidechain) χ1 torsion angles. 

##### Workflow Layouts
 - `chignolin-p1-1x1x10us`   A single 10μs long trajectory of the chignolin protein
 - `chignolin-p1-20x16x10ns` 4μs of data from 25 rounds of 16 10ns trajectories
 - `chignolin-p1-10x16x25ns` 4μs of data from 10 rounds of 16 25ns trajectories
 - `chignolin-p1-10x8x50ns`  4μs of data from 10 rounds of 8  50ns trajectories


