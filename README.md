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
 
This is a set of programs used for estimating the melting point of a material by using machine learning methods to classify samples accross a large range of temperatures as solid or liquid and predicting the transition. The samples are generated by performing isobaric-isothermal Monte Carlo simulations across the temperature range using the LAMMPS simulation package within python. The Monte Carlo methods employed involve random position moves, random volume moves, and Hamiltonian Monte Carlo moves.

Requirements
============

Python
------

- LAMMPS
- NumPy
- SciPy
- Joblib
- Dask (distributed)
- Scikit-Learn
- MulticoreTSNE
- Lasagne
- Scikit-NeuralNetwork
- MatPlotLib
- PIL

LAMMPS Libraries
----------------

- USER-MEAMC (for MEAM potentials)
- PYTHON

File Descriptions
=================

lammps_mc.py
-------------

This program interfaces with LAMMPS to produce thermodynamic information and trajectories from NPT-HMC simulations that sweep through a range of temperatures at fixed pressure. LAMMPS is used to constuct the system and run the dynamics. The Monte Carlo moves are performed in Python. Three type of Monte Carlo moves are defined: the NPT volume move (VMC), the classic atom-wise position move (PMC), and the Hamiltonian Monte Carlo move (HMC). Different probabilities can be chosen for each type of MC move (specified for HMC and PMC while VMC takes the remaining probability). Other general user-controlled parameters include the number of total MC move attempts, the number of timesteps in HMC moves, the number of data sets, and the name of the simulation. Material specific parameters are stored dictionaries with the atomic symbols serving as the keys ('LJ' for Lennard-Jones). The default is Lennard-Jones since it is useful for testing and is included with LAMMPS with no extra libraries needed. These dictionaries control the simulation pressure, the temperature array (length determined by the number of data sets), the lattice type and parameter, the various MC parameters (box adjustment for VMC, position adjustment for PMC, and timestep for HMC). The MC parameters are adaptively adjusted during the simulation. Two output files are written to, one containing general thermodynamic properties and simulation details, and another containing the atom trajectories. Currently, there is no support for running LAMMPS in parallel within the Python script.

lammps_remcmc_local.py
----------------------

This program implements replica exchange Markov chain Monte Carlo alongside the Monte carlo methods described in lammps_mc.py. This involves running NPT-HMC Monte Carlo simulations at multiple different pressures and temperatures and attempting to swap the atomic configurations between said simulations at regular intervals. The simulations at different temperatures and pressures can be run in parallel. Note that for the parallel mode to work, the serial version of the LAMMPS shared library must be used or else there will be memory management errors.

lammps_remcmc_distributed.py
----------------------------------

This is essentially the same as lammps_remcmc_local.py with both serial and parallel modes, but this script is intended to be used on distributed networks. The program works fine on a single machine, but proper setup on distributed networks has been difficult to implement.

lammps_parse.py
---------------

This program parses the output from Monte Carlo simulations and pickles the data.

lammps_rdf.py
-------------

This program calculates the radial distributions and structure factors for each sample using the pickled trajectory information from the parsing script. The radial distributions are performed in parallel and the structure factors are calculated as Fourier transitions of the radial distributions. The script pickles the data, which also includes density information.

lammps_neural.py
----------------

This program classifies samples as either solids or liquids by passing either the radial distributions or the structure factors through a multi-layer perceptron neural network. There are many options available with regards to which data scaler, classification neural network, and fitting function to use in addition to whether data reduction/transformation should be used.

### Available scalers
- Standard: very common, vulnerable to outliers, does not guarantee a map to a common numerical range
- MinMax: also common, vulnerable to outliers, guarantees a map to a common numerical range
- Robust: resilient to outliers, does not guarantee a map to a common numerical range
- Tanh: resilient to outliers, guarantees a map to a common numerical range

### Available networks
- Dense Classifier: fully-connected classifier MLP (ReLu -> SoftMax)
- 1-D CNN: not currently working due to incompatibilities between Scikit-NeuralNetwork and Lasagne convolution layers
- 2-D CNN: 5-layer convolutional neural network classifier

           1. convolution (ReLu, 4 channels, (8,1) kernel, (1,1) stride)
           2. convolution (ReLu, 4 channels, (4,1) kernel, (1,1) stride)
           3. convolution (ReLu, 4 channels, (2,1) kernel, (1,1) stride)
           4. convolution (ReLu, 4 channels, (1,1) kernel, (1,1) stride)
           5. output (SoftMax)
          
### Available fitting functions
- Logistic: well-behaved and easily extracted transition temperature estimate, symmetric
- Gompertz: well-behaved and easily extracted transition temperature estimate, faster uptake than saturation
- Richard: generalized logistic function, not recommended (too many fitting parameters)

### Plans for the future
- Refine neural network structures and parameters (learning rate, weights, iterations, etc.)
- Refine data preparation techniques (more appropriate scaling, data reorientation, etc.)
- Refine data analysis techniques (better curve fitting, alternate approaches to transition etimation, etc.)

lammps_cluster.py
-----------------

This program classifies samples as either solids or liquid by passing either the radial distributions or the structure factors through an unsupervised clustering algorithm following data dimensionality reduction. Dimensionality reduction is limited to either PCA or PCA followed by t-SNE.

### Available scalers
- Standard: very common, vulnerable to outliers, does not guarantee a map to a common numerical range
- MinMax: also common, vulnerable to outliers, guarantees a map to a common numerical range
- Robust: resilient to outliers, does not guarantee a map to a common numerical range
- Tanh: resilient to outliers, guarantees a map to a common numerical range

### Available clustering methods
- K-Means: Good for globular data, struggles on elongated data sets and irregular cluster boundaries (including concentric clusters)
- Agglomerative: Good for globular data, struggles with low density clusters and concentric clusters
- Spectral: Good for connected data (including concentric), struggles with edges of globular data

### Plans for the future
This will take second priority to the supervised method with respect to data analysis
- Allow for tuning of clustering method parameters (choice of metric, similarity measure, affinity, etc.) 
- Perhaps support for other nonlinear dimensionality reduction algorithms will be added

TanhScaler.py
-------------

This program implements a tanh-estimator as introduced by Hampel et al. with usage emulating the scalers found in the preprocessing sub-library of Scikit-Learn. However, this implementation does not use the Hampel estimators and instead uses the means and standard deviations of the scores directly by way of the StandardScaler class from Scikit-Learn.

### Plans for the future
- Perhaps Hampel estimators can be used

Further Development
===================

Some more advanced Monte Carlo methods may be implemented to improve sampling in addition to more robust machine learning analysis.
