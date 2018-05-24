# lammpsHMC

Description
===========
 
This is a set of programs used for classifying samples of materials across a range of temperatures as solid or liquid for the purpose of estimating the melting point. The smaples are generated by performing isobaric-isothermal Hamiltonian Monte Carlo simulations using the LAMMPS simulation package within python.

Requirements
============

Python
------

- LAMMPS
- NumPy
- SciPy
- Jobib
- Scikit-Learn
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

lammps_hmc.py
-------------

This program interfaces with LAMMPS to produce thermodynamic information and trajectories from NPT-HMC simulations. LAMMPS is used to constuct the system and run the dynamics. The Monte Carlo moves are performed in Python.

lammps_parse.py
---------------

This program parses the output from lammps_hmc.py and pickles the data.

lammps_rdf.py
-------------

This program calculates the radial distributions and structure factors for each sample using the pickled trajectory information from lammps_parse.py.

lammps_neural.py
----------------

This program classifies samples as either solids or liquids by passing either the radial distributions or the structure factors through a neural network. There are many options available with regards to which data scaler, classification neural network, and fitting function to use in addition to whether data reduction/transformation should be used.

Further Development
===================

Some more advanced Monte Carlo methods may be implemented to improve sampling in addition to more robust machine learning analysis.
