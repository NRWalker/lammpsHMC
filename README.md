

     ███▄    █ ▓█████  █    ██  ██▀███   ▄▄▄       ██▓        ███▄ ▄███▓▓█████  ██▓  ▄▄▄█████▓ ██▓ ███▄    █   ▄████
     ██ ▀█   █ ▓█   ▀  ██  ▓██▒▓██ ▒ ██▒▒████▄    ▓██▒       ▓██▒▀█▀ ██▒▓█   ▀ ▓██▒  ▓  ██▒ ▓▒▓██▒ ██ ▀█   █  ██▒ ▀█▒
    ▓██  ▀█ ██▒▒███   ▓██  ▒██░▓██ ░▄█ ▒▒██  ▀█▄  ▒██░       ▓██    ▓██░▒███   ▒██░  ▒ ▓██░ ▒░▒██▒▓██  ▀█ ██▒▒██░▄▄▄░
    ▓██▒  ▐▌██▒▒▓█  ▄ ▓▓█  ░██░▒██▀▀█▄  ░██▄▄▄▄██ ▒██░       ▒██    ▒██ ▒▓█  ▄ ▒██░  ░ ▓██▓ ░ ░██░▓██▒  ▐▌██▒░▓█  ██▓
    ▒██░   ▓██░░▒████▒▒▒█████▓ ░██▓ ▒██▒ ▓█   ▓██▒░██████▒   ▒██▒   ░██▒░▒████▒░██████▒▒██▒ ░ ░██░▒██░   ▓██░░▒▓███▀▒
    ░ ▒░   ▒ ▒ ░░ ▒░ ░░▒▓▒ ▒ ▒ ░ ▒▓ ░▒▓░ ▒▒   ▓▒█░░ ▒░▓  ░   ░ ▒░   ░  ░░░ ▒░ ░░ ▒░▓  ░▒ ░░   ░▓  ░ ▒░   ▒ ▒  ░▒   ▒
    ░ ░░   ░ ▒░ ░ ░  ░░░▒░ ░ ░   ░▒ ░ ▒░  ▒   ▒▒ ░░ ░ ▒  ░   ░  ░      ░ ░ ░  ░░ ░ ▒  ░  ░     ▒ ░░ ░░   ░ ▒░  ░   ░
       ░   ░ ░    ░    ░░░ ░ ░   ░░   ░   ░   ▒     ░ ░      ░      ░      ░     ░ ░   ░       ▒ ░   ░   ░ ░ ░ ░   ░
             ░    ░  ░   ░        ░           ░  ░    ░  ░          ░      ░  ░    ░  ░        ░           ░       ░


Description
===========

This is a set of scripts used for estimating the melting point of a material. This is done by classifying a multitude of sample configurations of a material generated by Monte Carlo simulations as either solid or liquid phase using machine learning methods and predicting the transition with the classification results. The Monte Carlo simulations are done in the isobaric-isothermal ensemble with available Monte Carlo moves including atomic position displacement, simulation box dimension displacement, Hamiltonion Monte Carlo, and replica exchange Markov chain Monte Carlo. The molecular dynamics and thermodynamic calculations are done using the Python library interface to LAMMPS while the Monte Carlo moves themselves are done with Python.

Requirements
============

Python
------

I recommend using Anaconda since the only additional libraries needed are LAMMPS, Multicore-TSNE, and Keras.

- LAMMPS
- NumPy
- SciPy
- Dask
- Numba
- Scikit-Learn
- Multicore-TSNE
- Keras
- TensorFlow/Theano
- tqdm
- MatPlotLib

LAMMPS Information
------------------

In order to use LAMMPS in Python, it must be compiled as a shared library. The instructions for doing so are presented on the LAMMPS webpage. Furthermore, if the user intends to use the parallel modes available with the replica exchange Markov chain Monte Carlo scripts, the serial version of the LAMMPS shared lirary must be used. Note that this then prevents instances of LAMMPS themselves taking advantage of parallelism, which should only be particularly costly for Hamiltonian Monte Carlo.

File Descriptions
=================

lammps_remcmc.py
----------------------------------

This program interfaces with LAMMPS to produce thermodynamic information and trajectories from NPT-HMC simulations that sweep through a range of temperatures and pressures simultaneously. LAMMPS is used to constuct the system, run the dynamics, and calculate the physical properties. The Monte Carlo moves, however, are performed in Python. Three types of Monte Carlo moves are defined and can be performed at each Monte Carlo sweep: The standard atom-wise position move (PMC), the NPT volume move (VMC), and the Hamiltonian Monte Carlo move (HMC). Different probabilities can be chosen for each type of MC move (specified for PMC and VMC while HMC takes the remaining probability). At the end of each data collection cycle, replica exchange Markov chain Monte Carlo is performed between all sample pairs.

The simulation can be run in multiple modes for debugging and implementing parallelism. There are boolean values for controlling verbosity, whether to run in parallel, if the parallelism is on a local or distributed cluster, and if the parallel method is multiprocessing or multithreading. The parallel implementation is done with the Dask Distributed library and only runs the Monte Carlo simulations in parallel (the LAMMPS instances are serial). Since Dask uses Bokeh, multiprocessing runs may be monitored at localhost:8787/status assuming the default Bokeh TCP port is used.

General user-controlled parameters include the verbosity, parallel modes, number of workers/threads, simulation name, the element, the system size (in unit cells), the pressure and temperature ranges to be simulated, the cutoff for data collection, the number of samples to be collected, the frequency of data collection, the probability of the PMC and VMC moves, and the number of HMC timesteps. Material specific parameters are stored dictionaries with the atomic symbols serving as the keys ('LJ' for Lennard-Jones). The default is Lennard-Jones since it is useful for testing and is included with LAMMPS with no extra libraries needed. These dictionaries control the define the lattice type and parameter, the atomic mass, and the parameters for each MC move (position adjustment, box size adjustment, and timestep). The MC parameters are adaptively adjusted during the simulations for each sample independently.

Two output files are written to during the data collection cycles, one containing general thermodynamic properties and simulation details, and another containing the atom trajectories.

lammps_parse.py
---------------

This program parses the output from Monte Carlo simulations and pickles the data. The pickled data includes the temperatures, potential energies, kinetic energies, virial pressures, volumes, acceptation ratios for each MC move, and trajectories for each sample.

lammps_rdf.py
-------------------------

This program calculates the radial distributions, structure factors, and entropic fingerprints, and densities alongside the domains for each structural function for each sample using the pickled trajectory information from the parsing script. The calculations can be run in parallel using a multiprocessing or multithreading approach on local or distributed clusters. The parallelism is implemented with the Dask Distributed library. Since Dask uses Bokeh, multiprocessing runs may be monitored at localhost:8787/status assuming the default Bokeh TCP port is used. The rdf calculations are also optimized using a combination of vectorized code by way of NumPy alongside a JIT compiler by way of Numba.

lammps_neural.py
----------------

This program classifies samples as either solids or liquids by passing structural information through a multi-layer perceptron neural network. There are many options available with regards to which structural information, scaler, reducer, classification neural network, and fitting function to use.

### Structure Features
- Radial distribution
- Structure factor
- Entropic fingerprint

### Feature Scalers
- Standard: very common, vulnerable to outliers, does not guarantee a map to a common numerical range
- MinMax: also common, vulnerable to outliers, guarantees a map to a common numerical range
- Robust: resilient to outliers, does not guarantee a map to a common numerical range
- Tanh: resilient to outliers, guarantees a map to a common numerical range

### Feature Dimensionality Reducers
- None: use the raw scaled data
- PCA: common and fast, orthogonal linear transformation into new basis that maximizes variance of data along new projections
- Kernal PCA: slower than PCA, nonlinear reduction in the sample space rather than feature space
- Isomap: slower than PCA, a nonlinear reduction considered to be an extension of the Kernel PCA algorithm
- Locally Linear Embedding: slower than PCA, can be considered as a series of local PCA reductions that are globally stiched together

### Neural Networks
- Dense Classifier  -- NOT CURRENTLY WORKING --
- 1-D Convolutional Neural Network Classifier

### Fitting Functions
- Logistic: well-behaved and easily extracted transition temperature estimate, symmetric
- Gompertz: well-behaved and easily extracted transition temperature estimate, faster uptake than saturation -- ERROR ANALYSIS UNDERGOING REWRITE --

### Future Plans
- Refine neural network structures and hyperparameters with grid searching
- Refine data preparation techniques (more appropriate scaling, data reorientation, etc.)
- Refine data analysis techniques (better curve fitting, alternate approaches to transition etimation, etc.)

Since the structural feautres, feature scalers, feature dimensionality reducers, neural networks, and fitting functions are all embedded in dictionaries, the user may feel free to add their own.

lammps_cluster.py
-----------------

-- CURRENTLY UNDERGOING REWRITE --

This program classifies samples as either solids or liquid by passing either the radial distributions or the structure factors through an unsupervised clustering algorithm following data dimensionality reduction. Dimensionality reduction is limited to either PCA or PCA followed by t-SNE.

### Feature Scalers
- Standard: very common, vulnerable to outliers, does not guarantee a map to a common numerical range
- MinMax: also common, vulnerable to outliers, guarantees a map to a common numerical range
- Robust: resilient to outliers, does not guarantee a map to a common numerical range
- Tanh: resilient to outliers, guarantees a map to a common numerical range

### Clustering Methods
- K-Means: Good for globular data, struggles on elongated data sets and irregular cluster boundaries (including concentric clusters)
- Agglomerative: Good for globular data, struggles with low density clusters and concentric clusters
- Spectral: Good for connected data (including concentric), struggles with edges of globular data

### Future Plans
This will take second priority to the supervised method with respect to data analysis
- Allow for tuning of clustering method parameters (choice of metric, similarity measure, affinity, etc.)
- Perhaps support for other nonlinear dimensionality reduction algorithms will be added

Since the scalers and clustering methods are embedded in libraries, the user may feel free to add their own features.

lammps_post.py
--------------

This program does post-processing on the transition predictions at multiple different pressures to extract the melting curve in addition to plots of the potential energies and pressures as functions of temperature.

TanhScaler.py
-------------

This program implements a tanh-estimator as introduced by Hampel et al. with usage emulating the scalers found in the preprocessing library of Scikit-Learn. However, this implementation does not use the Hampel estimators and instead uses the means and standard deviations of the scores directly by way of the StandardScaler class from Scikit-Learn.

Further Development
===================

Some more advanced Monte Carlo methods may be implemented to improve sampling in addition to more robust machine learning analysis. GPU support for LAMMPS may also be added as well as support for more atomic simulation environments by way of ASE.