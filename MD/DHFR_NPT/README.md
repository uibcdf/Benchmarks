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

# Preparing the system
The instructions to create the pdb are in the jupyter notebook.

# Run with gromacs

We run the system with AMBER99SB protein:

```bash
gmx pdb2gmx -f DHFR_23558.pdb -o DHFR_init.gro -water tip3p
```

The system has a net charge or -11, this charge will not be counterbalanced with ions since we
don't care here about correct conditions but comparing the performance. That's why the flag
`maxwarn 1` is used.

```bash
gmx grompp -f minim.mdp -c DHFR_init.gro -p topol.top -o energy_minim.tpr -maxwarn 1
gmx mdrun -v -deffnm energy_minim
```






