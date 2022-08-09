# High-Entropy Alloy Catalysis Simulation
HEACS is a set of Python modules for modeling catalysis on high-entropy alloys (HEAs). <br />
This repository is closely linked to the article <br />
"Ab Initio to activity: Machine learning assisted optimization of high-entropy alloy catalytic activity." <br />
DOI: https://doi.org/10.26434/chemrxiv-2022-vvrrf-v2

If this repository is utilized please cite: <br />
Clausen, C. M., Nielsen, M. L. S., Pedersen, J. K., & Rossmeisl, J. (2022). Ab Initio to activity: Machine learning assisted optimization of high-entropy alloy catalytic activity.

We hope you find this page helpful :)


Requirements
------------
* [ase](https://wiki.fysik.dtu.dk/ase/index.html)
* [gpaw](https://wiki.fysik.dtu.dk/gpaw/)
* [SLURM](https://slurm.schedmd.com)
* [scikit-learn](https://scikit-learn.org/stable/)
* [torch](https://pypi.org/project/torch/)
* [torch-geometric](https://pypi.org/project/torch-geometric/)
* [torch-sparse](https://pypi.org/project/torch-sparse/)
* [torch-scatter](https://pypi.org/project/torch-scatter/)
* [iteround](https://pypi.org/project/iteround/)

Setting up density functional theory geometry optimizations
---------------------
In the 
[data](data)
 directory all calculations of DFT simulated slabs are set up. Each HEA slab can be used to sample multiple adsorption sites saving towards half of computational resources. All simulations are stores in ASE database-files for easy overview and subsequently joined into a single database to construct regression features. Instructions are found in the 
[data](data)
 directory's 
[README.md-file](data/README.md)
.


Initializating regression of  
------------------------------

```python
from someDFTprogram import DFTCalculator
from heacs.calculate import AdsorptionEnergy
energies = DFTCalculator('hea_slabs.db')
slabs = read('hea_slabs.db')
properties = AdsorptionEnergy().get(slabs, energies, ref='Pt111')
slabs.add_property(properties)
```

Extrapolating properties
------------------------

```python
from heacs.extrapolate import LinearRegression
from heacs.features import NeighborCounting
from heacs.io import read
known_data = read('hea_slabs.db')
unknown_data = known_data.get_all_surfaces()
reg = LinearRegression(known_data, NeighborCounting).predict(unknown_data)
reg.parity_plot('parity_plot.png')
reg.save_regressor('regressor.pickle')
```

Optimizing alloy composition
----------------------------

```python
from heacs.optimize import BayesianOptimizer
from heacs.io import read
known_data = read('hea_slabs.db')
reg = read('regressor.pickle')
opt = BayesianOptimizer(known_data, reg)
optimum = opt.find_optimum(acquisition_function='expected improvement')
```
