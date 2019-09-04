<img src="https://raw.githubusercontent.com/franciscovillaescusa/Quijote-simulations/master/images/LSS.png" alt="LSS" width="275"/> <img src="https://raw.githubusercontent.com/franciscovillaescusa/Quijote-simulations/master/images/logo.gif" alt="DL" width="275"/> <img src="https://raw.githubusercontent.com/franciscovillaescusa/Quijote-simulations/master/images/DL.png" alt="DL" width="275"/>
###### The image on the right have been generated by a neural network (DeepDream) from the image on the left

# Quijote simulations
The Quijote simulations are a set of 43000 full N-body simulations. They are designed for two main tasks
- Quantify the information content on cosmological observables
- Provide enough statistics to train machine learning algorithms

But they can be used for a large variety of problems.

### Features
- Simulations run with the TreePM code Gadget-III
- More than 35 Million cpu hours 
- Boxes of 1 Gpc/h
- 17000 simulations for a fiducial Planck cosmology
- Between 500 and 1000 simulations/cosmology for 17 different cosmologies
- 11000 simulations in different latin-hypercubes 
- More than 8.5 trillions of particles at a single redshift
- Billions of halos and voids identified
- Snapshots at redshifts 0, 0.5, 1, 2, 3 and 127 (initial conditions)
- More than 200000 halo catalogues
- More than 200000 void catalogues
- More than 1 million power spectra
- More than 1 million bispectra
- More than 1 million correlation functions
- More than 1 million marked power spectra
- More than 1 million probability distribution functions
- More than 1 Petabyte of data publicly available

## Data
The data is stored in the Gordon cluster of the San Diego Supercomputer Center. It can be access through [globus](https://www.globus.org/). 

- Log in into [globus](https://www.globus.org/) (create an account if you dont have one)
- In collection type Quijote_simulations

There are different folders: 
- __Snapshots__. This folder contains the snapshots of the simulations
- __Halos__. This folder contains the halo catalogues
- __Voids__. This folder contains the void catalogues
- __Pk__. This folder contains the power spectra
- __Marked_Pk__. This folder contains the marked power spectra
- __Bk__. This folder contains the bispectra 
- __CF__. This folder contains the correlation functions
- __PDF__. This folder contains the pdfs

Inside each of the above folders there is the data for the different cosmologies, e.g. h_p, fiducial, Om_m. A brief description of the different cosmologies is provided in the below table. Further details can be found in the Quijote paper.

![](https://raw.githubusercontent.com/franciscovillaescusa/Quijote-simulations/master/images/Sims.jpg)

### Snapshots
The snapshots are stored in either Gadget-II format or HDF5. They can be read using the [readgadget.py](https://github.com/franciscovillaescusa/Pylians/blob/master/library/readgadget.py) and [readsnap.py](https://github.com/franciscovillaescusa/Pylians/blob/master/library/readsnap.py) scripts. If you have [Pylians](https://github.com/franciscovillaescusa/Pylians) installed you already have them.

The snapshots only contain 4 blocks:
- Header: This block contains general information about the snapshot such as redshift, number of particles, box size, particle masses...etc.
- Positions: This block contains the positions of all particles. Stored as 32-floats
- Velocities: This block contains the velocities of all particles. Stored as 32-floats
- IDs: This block contains the IDs of all particles. Stored as 32-integers. (This block may be removed in the future to reduce the size of the snapshots)

An example on how to read a snapshot is this:

```python
import readgadget

# input files
snapshot = '/home/fvillaescusa/Quijote/Snapshots/h_p/snapdir_002/snap_002'
ptype    = [1] #[1](CDM), [2](neutrinos) or [1,2](CDM+neutrinos)

# read header
header   = readgadget.header(snapshot)
BoxSize  = header.boxsize/1e3  #Mpc/h
Nall     = header.nall         #Total number of particles
Masses   = header.massarr*1e10 #Masses of the particles in Msun/h
Omega_m  = header.omega_m      #value of Omega_m
Omega_l  = header.omega_l      #value of Omega_l
h        = header.hubble       #value of h
redshift = header.redshift     #redshift of the snapshot
Hubble   = 100.0*np.sqrt(Omega_m*(1.0+redshift)**3+Omega_l)#Value of H(z) in km/s/(Mpc/h)

# read positions, velocities and IDs of the particles
pos = readgadget.read_block(snapshot, "POS ", ptype)/1e3 #positions in Mpc/h
vel = readgadget.read_block(snapshot, "VEL ", ptype)     #peculiar velocities in km/s
ids = readgadget.read_block(snapshot, "ID  ", ptype)-1   #IDs starting from 0
```
In the simulations with massive neutrinos it is possible to read the positions, velocities and IDs of the neutrino particles. Notice that the field should contain exactly 4 characters, that can be blank: ```"POS ", "VEL ", "ID  "```. The number in the name of the snapshot represents its redshift: 
- 000 ------> z=3
- 001 ------> z=2
- 002 ------> z=1
- 003 ------> z=0.5
- 004 ------> z=0

### Halo catalogues
The halo catalogues can be read through the [readfof.py](https://github.com/franciscovillaescusa/Pylians/blob/master/library/readfof.py) script. If you have [Pylians](https://github.com/franciscovillaescusa/Pylians) installed you already have it. An example on how to read a halo catalogue is this:

```python
import readfof 

# input files
snapdir = '/home/fvillaescusa/Quijote/Halos/s8_p/145/' #folder hosting the catalogue
snapnum = 4                                            #redshift 0

# determine the redshift of the catalogue
z_dict = {4:0.0, 3:0.5, 2:1.0, 1:2.0, 0:3.0}
redshift = z_dict[snapnum]

# read the halo catalogue
FoF = readfof.FoF_catalog(snapdir, snapnum, long_ids=False,
                          swap=False, SFR=False, read_IDs=False)
										
# get the properties of the halos
pos_h = FoF.GroupPos/1e3            #Halo positions in Mpc/h                                                                                                                                                                       
mass  = FoF.GroupMass*1e10          #Halo masses in Msun/h                                                                                                                                                                      
vel_h = FoF.GroupVel*(1.0+redshift) #Halo peculiar velocities in km/s                                                                                                                                                                        
Npart = FoF.GroupLen                #Number of CDM particles in the halo
```
The number in the name of the halo catalogue represents its redshift: 
- 000 ------> z=3
- 001 ------> z=2
- 002 ------> z=1
- 003 ------> z=0.5
- 004 ------> z=0

### Void catalogues

The void catalogues are stored as hdf5 files. They contain the following blocks:
- pos:    the positions of the void centers in Mpc/h
- radius: the sizes of the voids in in Mpc/h
- VSF: the void size function
- VSF_Rbins: the radii bins of the void size function
- parameters: the values of the void finder parameters used to generate the void catalogue

In python, the files can be read as
```python
import h5py

f = h5py.File('/home/fvillaescusa/Quijote/Voids/fiducial/0/void_catalogue_m_z=0.hdf5', 'r')
pos        = f['pos'][:]        #void center positions in Mpc/h
radius     = f['radius'][:]     #void radii in Mpc/h
VSF        = f['VSF'][:]        #VSF (#voids/dR/Volume)
VSF_Rbins  = f['VSF_Rbins'][:]  #VSF radii in Mpc/h
parameters = f['parameters'][:] #parameters used to run the void finder

f.close()
```

### Power spectra

The format of the power spectra are:
- k | P(k) for power spectra in real-space
- k | P0(k) | P2(k) | P4(k) for power spectra in redshift-space

where P0(k), P2(k) and P4(k) are the monopole, quadrupole and hexadecapole, respectively.
The units of k are h/Mpc, while for the power spectra are (Mpc/h)^3.

In redshift-space there are three different files for each realization/redshift. These have been
computed by placing the redshift-space distortions along the three different axes.

In python, the files can be read as 

```python
import numpy as np

k, Pk = np.loadtxt('/home/fvillaescusa/Quijote/Pk/matter/fiducial/3/Pk_m_z=0.txt', unpack=True)
k, Pk0, Pk2, Pk4 = np.loadtxt('/home/fvillaescusa/Quijote/Pk/matter/fiducial/3/Pk_m_RS1_z=0.txt', unpack=True)
```

### Marked power spectra

The format of the marked power spectra files  are:
- k | M(k)  

where M(k) is the marked power spectrum. 
The unit of k is h/Mpc, while the one of M(k) is (Mpc/h)^3.

In python, the files can be read as 

```python
import numpy as np

k, Mk = np.loadtxt(FILENAME, unpack=True)
```

### Correlation functions

The format of the correlation functions are:
- R | xi(R)  for correlation functions in real-space
- R | xi0(R) | xi2(R) | xi4(R) for correlation functions in redshift-space

where xi0(R), xi2(R) and xi4(R) are the monopole, quadrupole and hexadecapole, respectively.
The units of R are Mpc/h, while the different xi are dimensionless.

In redshift-space there are three different files for each realization/redshift. These have been
computed by placing the redshift-space distortions along the three different axes.


In python, the files can be read as 

```python
import numpy as np

R, xi = np.loadtxt('/home/fvillaescusa/Quijote/CF/matter/fiducial/0/CF_m_1024_z=0.txt', unpack=True)
R, xi0, xi2, xi4 = np.loadtxt('/home/fvillaescusa/Quijote/CF/matter/fiducial/0/CF_m_RS0_1024_z=0.txt', unpack=True)
```

### Bispectra

The format of the individual bispectra files are: 
- k1/kf, k2/kf, k3/kf, P0(k1), P0(k2), P0(k3), B0(k1,k2,k3), Q(k1,k2,k3), B_SN(k1,k2,k3), counts 

where k1, k2, k3 specify the length of the triangle sides, P0(k) is the power spectrum monopole, 
B0(k1,k2,k3) is the bispectrum monopole, Q(k1,k2,k3) is the reduced bipsectrum, B_SN is the 
bispectrum shot noise correction, and counts is the number of triangles in the bin. 
_B0 is already shot noise corrected_. The header specifies kf, the fundamental mode, and Nhalo, 
the number of halos. 

The individual bispectra files can be read in python as follows, 

```python
import numpy as np 

k1, k2, k3, p0k1, p0k2, p0k3, b123, q123, b_sn, cnts = np.loadtxt(FILENAME, skiprows=1, unpack=True, usecols=range(10))

# read header to get Nhalo 
hdr = open(FILENAME).readline().rstrip()
Nhalo = int(hdr.split('Nhalo=')[-1])
```

Altneratively, sets of bispectra files for a specific redshift and cosmology can easily be accessed

```python 
import h5py 

fbk = h5py.File(FILENAME, 'r') 
k1 = fbk['k1'][...]
k2 = fbk['k2'][...]
k3 = fbk['k3'][...]
p0k1 = fbk['p0k1'][...]
p0k2 = fbk['p0k2'][...]
p0k3 = fbk['p0k3'][...]
b123 = fbk['b123'][...] 
q123 = fbk['q123'][...]
b_sn = fbk['b_sn'][...]
cnts = fbk['counts'][...] # triange counts
Nhalos = fbk['Nhalos'][...] # number of halos 
files = fbk['files'][...] # names of individual files. 
```

### PDFs

The format of the PDF files are:
- delta | pdf

where delta is the density contrast (rho/< rho > - 1).
	
In python, the files can be read as

```python
import numpy as np

delta, pdf = np.loadtxt('/home/fvillaescusa/Quijote/PDF/matter/latin_hypercube/0/PDF_m_5.0_z=0.txt', unpack=True)
```

## Team
- Francisco Villaescusa-Navarro (CCA)
- ChangHoon Hahn (Berkeley)
- Emanuele Castorina (Berkeley)
- Arka Banerjee (Stanford)
- Elena Massara (CCA)
- Elena Giusarma (CCA)
- Chi-Ting Chiang (BNL)
- Ana Maria Delgado (CUNY)
- Andrej Obuljen (Waterloo)
- David Spergel (CCA/Princeton)
- Ben Wandelt (IAP Paris)
- Shirley Ho (CCA/Princeton)
- Licia Verde (Barcelona)
- Matteo Viel (SISSA)
- Roman Scoccimarro (NYU)
