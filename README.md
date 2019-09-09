# chignolin-workflows
AdaptiveMD workflows for the chignolin system

If you want to learn more about (exeucuting) adaptive sampling workflows,
you can use this as a operationally-oriented tutorial that explains what is
happening in the order we do it from experimental design to workflow execution
to final analsys.

### Toolkit
 - Python from Miniconda
 - AdaptiveMD, PyEMMA, OpenMM

## A reproducible set of workflow instructions for comparative analysis using Chignolin
### `workflows`
The name of folders in the `workflows` directory indicate the layout of MD simulations in the workflow.\
`< system name >-< phase # >-< features >-< workflow layout >`

##### Phases
In each case of multiple rounds, the restarting frames are adaptively sampled from existing data.
Here a simple 2-phase formalism is used, where in phase 1 denoted by `p1` the objective is to
explore the conformation space of the protein as rapidly as possible.

The adaptive sampling algorithm "explore macrostates" is used from the AdaptiveMD package
where a large number of macrostates (~25) are estimated from the existing data via this analysis
pipeline implemented with calls to PyEMMA:

data --featurize--> feature trajectories --TICA--> 
reduced dims --cluster(k-means)--> 
microstates --estimate transitions-->
MSM --cluster(spectral, PCCA+)--> macrostates

Inverse count sampling is used to select macrostates for restarting by weighting each macrostate
inverse to its count of existing frames, so that less-visited macrostates are more likely to be
sampled for restarting. 

In phase 2 we want to estimate the full system kinetics as efficiently as possible.
An identical analysis is performed up to the estimation of an MSM, however in this phase we seek
to distribute restart states along the slowest process(es) instead of in the least visited states.

 - `chignolin-p1-1x1x10us` A single long trajectory of the chignolin protein
 - `chignolin-p1-20x16x10ns` 20 rounds of 16 10ns trajectories 
