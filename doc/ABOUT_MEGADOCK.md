# About MEGADOCK

> version: MEGADOCK 4.0.2

MEGADOCK is a ZDOCK like protein-protein docking program suitable for executing multiple docking jobs on heterogeneous supercomputers.  
It optimizes shape complementarity (rPSC function), electrostatic energies, and desolvation free energies (RDE function) using Fast Fourier Transform algorithm.  

MEGADOCK version 4 assumes running on GPU-based supercomputer like TSUBAME 2.5, Tokyo Institute of Technology, Japan.  
However, you can compile and run the program on other cluster systems or personal computers by modifying appropriate parameters in Makefile.  

## Build

[doc/BUILD.md](./BUILD.md)


## Example

### move to example data directory
```
cd ./data
```

### a) GPU & MPI (e.g. 4 MPI processes)
```sh
mpirun -n 4 ../megadock-gpu-dp -tb SAMPLE.table

# for using `HOSTS` file (HOSTS is node list file)
# mpirun -n 4 -hostfile HOSTS ../megadock-gpu-dp -tb SAMPLE.table
```

### b) CPU & MPI (e.g. 4 MPI processes)
```sh
mpirun -n 4 ../megadock-dp -tb SAMPLE.table

# for using `HOSTS` file (HOSTS is node list file)
# mpirun -n 4 -hostfile HOSTS ../megadock-dp -tb SAMPLE.table
```

### c) GPU single node
```sh
../megadock-gpu -R receptor.pdb -L ligand.pdb
```

### d) CPU single node
```sh
../megadock -R receptor.pdb -L ligand.pdb
```



## Parameters

| Required                     | Optional                                    |
| :----------------------------| :-------------------------------------------|
| `-tb [docking job table] `   | `-lg [log file name]` (default: master.log) |
|                              | `-rt [number of retries]` (defalt: 0) (*)   |

> (*) Number of retry of a docking job if fails


----


## Docking job table

Parameters and docking target list should be written in a text file.  
A table file should have `TITLE` and `PARAM` lines followed by parameters listed by the same order as in the `PARAM` line.  
`SAMPLE.table` file in this package shows an example.



## Docking parameters

You can specify docking parameters loaded by MEGADOCK by setting `PARAM` in the table file.  
Lines follows the `PARAM` lines specifies parameters for each docking job which will be distributed to available nodes by MPI.

### Required
```
 -R [receptor pdb file]
 -L [ligand pdb file]
```

### Optional
```
 -o [filename]    : set the output filename (default to "$R-$L.out")
 -O               : output docking detail files
 -N [integer]     : set the number of output predictions (default to 2000)
 -t [integer]     : set the number of predictions per each rotation (default to 1)
 -F [integer]     : set the number of FFT point (default to none)
 -v [float]       : set the voxel pitch size (default to 1.2 [angstrom])
 -D               : set the 6 deg. (54000 angles) of rotational sampling
                    (default to none, 15 deg. (3600 angles) of rotational sampling)
 -r integer       : set the number of rotational sampling angles
                    (54000: 54000 angles, 1: 1 angles, 24: 24 angles, default to 3600 angles)
 -e [float]       : set the electrostatics term ratio (default to 1.0)
 -d [float]       : set the hydrophobic term ratio (default to 1.0)
 -a [float]       : set the rPSC receptor core penalty (default to -45.0)
 -b [float]       : set the rPSC ligand core penalty (default to 1.0)
 -f [1/2/3]       : set function
                    (1: rPSC only, 2: rPSC+Elec, 3: rPSC+Elec+RDE, default to 3)
 -h               : show this message
```


----


## Docking output

Each docking job generates docking output file in which rotation angles (x, y, z) of ligand from the initial structure, numbers of voxel translation (x, y, z) from receptor center coordinate and docking scores are listed.  

If you want to generate decoy pdbfiles, please use `decoygen`. 
```sh
./decoygen [decoy_filename] [used_ligand.pdb] [.outfile] [decoy no.]
```

ex) `megadock -R rec.pdb -L lig.pdb -o dock.out`, generate 1st ranked decoy
```
./decoygen lig.1.pdb lig.pdb dock.out 1
cat rec.pdb lig.1.pdb > decoy.1.pdb

# lig.1.pdb : rotated and translated lig.pdb
# decoy.1.pdb : complex pdb file
```

ex) `megadock -R rec.pdb -L lig.pdb -o dock.out`, generate all decoys
```
for i in `seq 1 2000`; do ./decoygen lig.${i}.pdb lig.pdb \
  dock.out $i; cat rec.pdb lig.${i}.pdb > decoy.${i}.pdb; done

# Note: this command is oneliner.
```



## Blocking target residues

`block` tool (python script) is to block some residues of a receptor protein from docking.  
If you know that some residues are not in the binding sides, please list their residue numbers and input `block` tool.  
This program changes the name of residues to "BLK" and prints the new pdb on the screen.

```
./block [pdbfile] [chain] [target residue list]
# ex)
# ./block 1gcq_r.pdb B 182-186,189,195-198,204 > blocked.pdb
```

### Note
Target residues list is separated by commas and no spaces.  
You can also use `-` (hyphen): "182-186" means blocking residues of 182, 183, ..., 186.  
Blocked residues are substituted for 'BLK'.  
Updated PDB coordinates are written to "standard output".  
MEGADOCK can only block receptor residues.


----


## Thread parallelization

MEGADOCK can parallelize rotation calculations by using OpenMP.  
You can tell MEGADOCK the number of OpenMP threads you want to use by environmental variable such as `$OMP_NUM_THREADS`.


----


## References

- Masahito Ohue, Takehiro Shimoda, Shuji Suzuki, Yuri Matsuzaki, Takashi Ishida, Yutaka Akiyama. MEGADOCK 4.0: an ultra-high-performance protein-protein docking software for heterogeneous supercomputers. Bioinformatics, 30(22), 3281-3283, 2014.

- Masahito Ohue, Yuri Matsuzaki, Nobuyuki Uchikoga, Takashi Ishida, Yutaka Akiyama. MEGADOCK: An all-to-all protein-protein interaction prediction system using tertiary structure data. Protein and Peptide Letters, 21(8), 766-778, 2014.

- Takehiro Shimoda, Takashi Ishida, Shuji Suzuki, Masahito Ohue, Yutaka Akiyama. MEGADOCK-GPU: An accelerated protein-protein docking calculation on GPUs. In Proc. ACM-BCB 2013 (ParBio Workshop 2013), 884-890, 2013.
 
- Yuri Matsuzaki, Nobuyuki Uchikoga, Masahito Ohue, Takehiro Shimoda, Toshiyuki Sato, Takashi Ishida, Yutaka Akiyama. MEGADOCK 3.0: A high-performance protein-protein interaction prediction software using hybrid parallel computing for petascale supercomputing environments. Source Code for Biology and Medicine, 8(1): 18, 2013.

- Masahito Ohue, Yuri Matsuzaki, Takashi Ishida, Yutaka Akiyama. Improvement of the Protein-Protein Docking Prediction by Introducing a Simple Hydrophobic Interaction Model: an Application to Interaction Pathway Analysis. Lecture Note in Bioinformatics 7632 (In Proc. of PRIB 2012), 178-187, Springer Heidelberg, 2012.


## Contact

| Email                       | URL                                    |
| :---------------------------| :--------------------------------------|
| megadock@bi.cs.titech.ac.jp | http://www.bi.cs.titech.ac.jp/megadock |
