# The Amber MD parameters
NPT
250000 pasos
energia 2500 pasos
coordenadas 2500 pasos
checkpoing 250000 pasos
dt 2 fs
non bonded cutoff pme 8.0
shake in hdrogens
thermostat berendsen weak coupling with time tau 10.1
temperature 300 K
pbc
barostat isotropic position scaling, MC barostat
water TIP3P

# The OpenMM MD parameters
Probably similar with:
Langevin integrator and Amber99SB

# Ixtlilton

## Preparing the system in master

```bash
module purge
module load cmake/gcc-4.8.5_3.13.4
module load CUDA/gcc-4.8.5_10.0
module load openmpi/gcc-4.8.5_CUDA-10.0_4.0.0
module load gromacs/gcc-4.8.5_ownfftw_2019.1
``` 

### Preparing the system
The instructions to create the pdb are in the jupyter notebook.

We run the system with AMBER99SB protein:

```bash
gmx pdb2gmx -f DHFR_23558.pdb -o DHFR_init.gro -water tip3p -ignh
```

The system has a net charge or -11, this charge will not be counterbalanced with ions since we
don't care here about correct conditions but comparing the performance. That's why the flag
`maxwarn 1` is used.

```bash
gmx grompp -f minim.mdp -c DHFR_init.gro -p topol.top -o energy_minim.tpr -maxwarn 1
gmx mdrun -v -deffnm energy_minim
```

We run a first 100 ps equilibration with constant volume and positions restraints.

```bash
gmx grompp -f equil1_nvt.mdp -c energy_minim.gro -r energy_minim.gro -p topol.top -o equil1_nvt.tpr -maxwarn 1
gmx mdrun -gpu_id 0,1 -deffnm equil1_nvt
```

We run a second 200 ps equilibration with constant pressure and positions restraints. Positions and
velocities are read from equil2_nvt.

```bash
gmx grompp -f equil2_npt.mdp -c equil1_nvt.gro -r equil1_nvt.gro -t equil1_nvt.cpt -p topol.top -o equil2_npt.tpr -maxwarn 1
gmx mdrun -gpu_id 0,1 -deffnm equil2_npt
```

Finally, we run a short 1 ns run without restraints and NPT:

```bash
gmx grompp -f md_npt.mdp -c equil2_npt.gro -t equil2_npt.cpt -p topol.top -o md_npt.tpr -maxwarn 1
gmx mdrun -gpu_id 0,1 -deffnm md_npt
```

Finally, what is need to restart or to continue the run is:
- md_npt.gro
- md_npt.cpt
- topol.top

