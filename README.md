## What is it?
Full Monte is a python program that finds the low-energy conformations of flexible molecular systems. 


## How does it work?
Find low energy conformations of a given input structure. The searching method is a modified version of the Monte Carlo Multiple Minimum (MCMM) algorithm of Chang, Guida and Still. Since Full Monte can interface with MOPAC, it can be used to find transition state conformations, and to study systems for which there are no force-field parameters.

Starting from a single input geometry with specified atomic connectivity, guess structures are generated, optimized and then the energy is evaluated. Provided the energy is not too high and the optimized structure is different to those found already, the structure is saved along with the other low-energy conformations. 

Guess structures are generated by making random alterations to a random number of torsion angles in the lowest-energy structures found already during the search, so the MCMM algorithm is not entirely random. A previously saved structure is more likely to be selected for alteration if it has been used less frequently than another saved structure. Previously saved structures that have been used equally to generate new guess structures have an equal likelihood of being used next. This was termed "Uniform-Usage" by Chang, Guida and Still.

Only torsions are altered since bond lengths and bond angles are unlikely to differ much between different conformations of the same molecule(s). The variable torsions, which define then conformational space, are calculated automatically within Full Monte. Only single bonds are rotated, and only those are chosen for which rotation results in a discernible difference in conformation, so that groups such as methyl and tert-butyl are excluded. 

When the specified number of steps has been exhausted, or if the search stops finding new conformers, the search will end. The conformers found are then reoptimized with a higher convergence criterion to ensure that structures are fully optimized, and the final results are output.


##  File Formats
The required input format is Gaussian Cartesian input (*com) or PDB. The atomic connectivity must be specified so that Full Monte can calculate the variable torsions.

The low energy conformations are written to an (*_fm.out) outfile that can be opened by GaussView (check Read Intermediate Geometries to see all structures) and a (*_fm.pdb) outfile that can be opened by various programs such as PyMol. The search log is written to a (*_fm.log) file for inspection, and output from the calculations performed is compressed to a (*_fm.tgz) file.


##  Usage
First of all a functioning version of MOPAC is required. Eventually I will get round to interfacing with TINKER, but for the time being, only MOPAC is supported. If not already installed, download MOPAC from http://openmopac.net and obtain a licence key. There is a test file in the Full_Monte/TEST directory (test.mop) to check that MOPAC is working correctly:  copy test.mop to the current working location and execute with the following command, where mopac_location/MOPAC_2009.exe is the full path of the executable:

mopac_location/MOPAC_2009.exe test.mop

Upon successful execution the file test.out should appear, in which the following line can be found:

TOTAL ENERGY            =       -183.23058 EV

If this is not the case, MOPAC is probably not installed correctly, and Full Monte will not be able to work.

If everything is OK at this stage, then Full Monte can be tested. Copy the folder Full_Monte_Carlo to any location on your computer. Then set the variable that point to the location where you just put this folder in the .bashrc file by adding to the following two statements (changing file_location as appropriate):

export FULL_MONTE_DIR='file_location/Full_Monte_Search'
alias FullMonte='$FULL_MONTE_DIR/FullMonte'

Now copy the test files testfm.com and testparamfile found in the Full_Monte/TEST directory to the current working directory and execute Full Monte with the following command:

FullMonte testfm.com testparamfile

(If this is the first time Full Monte has been used, the program may need to be told where to find the MOPAC executable. Run the following FullMonte with one of the arguments as Ðsetup and respond to the two questions, so that the location can be updated)

Respond to any prompts or just press return and you should see the search begin. The search parameters are output at the beginning, followed by real time updates of new conformers found. When the search is complete, the program should exit and you can view the logfile, and visualize the conformers found in the _fm.out and _fm.pdb files.


## Adjustable Parameters
Parameters may be specified for the Monte Carlo search algorithm in a plain text file.

FullMonte structure.com params

Each adjustable parameter is indicated by a four letter keyword, which can be set with a statement such as: COMP = 10.0. Each of these statements must occupy a new line. 

COMP = 10.0
EWIN = 20.0
LEVL = AM1

If a parameter is not adjusted in this way (either because it is not specified in the plain text file or no file is specified) then the default parameter setting will be used in Full Monte.

The adjustable parameters are described below. Some of the keywords are inherited from the orginal definitions of Chang, Guida and Still, while new keywords have also been developed.

### COMP (new)
Conformers are deemed to be identical if they have very similar energies and geometries. For each optimization performed, the root mean squared difference in heavy atom torsion angles is compared with previously saved conformers. If this value falls below COMP (degrees) and the difference in energy is smaller than 0.5 kJ/mol the two structures are deemed identical. 

The default value for COMP is 10

### DEMX
Conformers are saved if they lie within DEMX kJ/mol of the global energy minimum conformation. Those above this energy are discarded.

The default value for DEMX is 41.84 kJ/mol (corresponding to 10 kcal/mol)

### EWIN
In the Monte Carlo Multiple Minimum (MCMM) search method, existing low-energy conformers are used to generate new guess structures for geometry optimization. EWIN sets the energy window above the global energy minimum from which structures are taken to create new guess structures.

The default value of EWIN is 20 kJ/mol (around 5 kcal/mol)

### EQUI (new)
It is possible to specify two or more atoms that are equivalent, such as the two oxygen atoms of a carboxylate anion. In general, Full Monte is able to understand that conformations which differ by rotation about a methyl group or tert-butyl group are identical, however, users may find more complex cases where symmetrical groups cause the same conformation to be saved more than once. Atom number are used to specify equivalent atoms, and any number of equivalent groups may be specified on separate lines, for example:

EQUI = 2 5 7 
EQUI = 14 18

### FIXT (new) 
Full Monte automatically decides which torsion angles will be altered throughout the course of the conformational search, based on the connectivity and the equivalent atoms in the molecule. One way to remove a particular torsion from the search is to set the bond order between the two central atoms to a number greater than one, as only single bonds are considered for rotation. The other way is to use FIXT to specify two atom numbers about which no rotations are performed.

### HSWP (new)
Semi-empirical optimizations sometimes lead to structural isomerization from the input geometry. Usually this will be undesirable and so these isomers are rejected by Full Monte. However, sometimes NH/OH tautomerization will be relevant to locate low-energy conformations, say in conformational studies of peptides, and so HSWP = 1 allows these isomers to be saved in the same search. This can therefore be an advantage over force-field based conformational searching in such cases.

HSWP is OFF by default

### LEVL (new)
The model chemistry used for energy evaluation and geometry optimizations. At present, three semi-emiprical levels of theory may be specified: AM1, PM3 and PM6, all of which are implemented through MOPAC. Future implementations will support the use of force-fields (AMBER, OPLS, AMOEBA) through TINKER.

Notes of caution for semi-empirical calculations: 
Bear in mind that AM1 parameters do not exist for all atoms, causing jobs to fail. 
If nitrogen atoms are present, users will be asked if they wish to use a molecular mechanics correction since nitrogen planarity is a known issue in semi-empirical calculations.

The default for LEVL is PM3

If PM6 is used, the option of a correction for dispersion and hydrogen-bonding interactions is available during the setup of the Full Monte search. This corresponds to the PM6-DH method proposed by Hobza and co-workers.
PROC (new)
The submission of optimizations simultaneously on multi-core machines will accelerate the search time, although during the early stages of the search this makes the search slightly more random than traditional implementations of the MCMM method. PROC sets the number of processors available for optimizations.

The default is PROC = 1, corresponding to traditional MCMM searches.

### RJCT (new)
Guess structures are generated by applying a random number of changes to selected torsion angles in previously saved conformers. This may cause awkward atomic arrangements likely to cause structural isomerization to occur. For force-field calculations this is not a problem, since the inherent connectivity defined by this level of theory prevents isomerization. However, for semi-empirical calculations this can be a problem in that too much time of the conformational search is wasted on irrelevant structures. Guess structures are rejected prior to optimization if any of the non-bonded distances lies below the value set by RJCT, which is a fraction of the sum of the two non-bonded atoms Bondi radii. If this happens, the a new guess geometry is created applying a different set of random torsion rotations. For example RJCT = 0.0 turns off this filtering (not recommended) while RJCT = 1.0 would reject guess structures with any non-bonded distances smaller than the two atoms summed Bondi radii.

The default of RJCT = 0.5

### STEP
The number of conformations generated during the search, which corresponds to the number of search steps in a traditional MCMM run (NPROC = 1). For searches with simultaneous optimizations, the number of conformations generated is still equal to STEP, although the number of search steps will be smaller, since multiple conformers are optimized each time. 

A note of caution: 
If STEP is not specified, it is calculated as 3N where N is the number of rotatable torsions. This may well be a very large number, resulting in a long search time.
Searches terminate when all the steps have finished, or when the search stops finding new conformers Ð if 100 geometries are optimized without finding a new conformer, Full Monte will exit.

## TERMINOLOGY

- NREJECT: The number of optimizations which result in a structure that is rejected for whatever reason: high energy, structural isomerization, or is the same as another previously saved.
- NFAILED: The number of optimizations which do not terminate normally.
- AERATE: The percentage of optimizations which result in an accepted conformer
- DMIN: The number of times that the least-found conformer has been located. 
- MCNV: The number of 	rotatable torsions which will be altered during the conformational search. Each time a new guess geometry is generated, a random number between 2 and MCNV-1 of torsion angles are varied by random amounts.

## REFERENCES
Monte Carlo Multiple Minimum: Chang, G.; Guida, W. C.; Still, W. C. ÒAn Internal Coordinate Monte Carlo Method for Searching Conformational SpaceÓ J. Am. Chem. Soc. 1989, 111, 4379-4386.

MOPAC: MOPAC2009, James J. P. Stewart, Stewart Computational Chemistry, Colorado Springs, CO, USA, http://openmopac.net (2008).

AM1: Dewar, M. J. S.; Zoebisch, E. G.; Healy, E. F.; Stewart, J. J. P. ÒAM1: A New General Purpose Quantum Mechanical Model.Ó J. Am. Chem. Soc. 1985, 107, 3902-3909.

PM3: Stewart, J. J. P. Optimization of Parameters for Semi-Empirical Methods I. Method. J. Comp. Chem. 1989, 10, 209-220.

PM6: Stewart J. J. P. Optimization of Parameters for Semiempirical Methods V: Modification of NDDO Approximations and Application to 70 Elements J. Mol. Model. 2007, 13, 1173-1213.

PM6-DH: Korth, M.; Pitonak, M.; Rezac, J.; Hobza, P. A Transferable H-bonding Correction For Semiempirical Quantum-Chemical Methods,  J. Chem. Theory Comp. 2010, 6, 344Ð352.

Bondi radii: van der Waals radii for all atoms from: Bondi, A. J. Phys. Chem. 1964, 68, 441-452, except hydrogen, which is taken from Rowland, R. S.; Taylor, R. J. Phys. Chem. 1996, 100, 7384-7391.

