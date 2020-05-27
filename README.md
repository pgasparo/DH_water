# Dynamical Heterogeneity in bulk and confined water 
LAMMPS input files to compute DH in water.

The inputs are diveded in directories, by model, confinement type and temperature. The name of the direcotory explains its content. 

Input files are usually in the directory named LAMMPS_inputs, while the system-specifc data can be found in the apposit directory.
To run a simulation use:

```
mpirun -np $NCPUs lmp_mpi -var Timestep $TIMESTEP -var psRunTime $RUNTIME -var psStride $STRIDE -var nRand $RANDOMSEED -var dataIN $DATAFILE -var T0 $TARGET_T -var outfile $OUTPUT_TRAJ -log $LOGFILE < $INPUTFILE
```

To perform isoconfigurational analysis and probe DH in water one has first to sample tens of independent snapshots from a well-equilibrated NVT trajectory. From each independet snapshot at least 50 NVT (or NVE) runs should then be started, initializing every time the velocities randomly according to the Boltzmann distribution. To compute the local diffusivity, or the dynamical propensity, only the displacements of atoms at the time of maximum heterogeneity are needed. This can achieved adding to the input file the following commands:

```
compute disp gOxygen displace/atom
dump d2 gOxygen custom $(v_psRunTime*1000/v_Timestep) ${outfile}.displacements c_disp[1] c_disp[2] c_disp[3]
dump_modify d2 sort id

run $(v_psRunTime*1000/v_Timestep)
```

where in this case $RUNTIME will be the time of maximum heterogeneity.

Once obtained an ensamble of displacements the following script can be used to compute the local diffusivity:

```
#!/bin/sh

c=0
for i in *displ* ; do
  c=$((c+1))
  tail -3072 $i | awk '{print ($1*$1+$2*$2+$3*$3)}' > tmp.$c
done

paste tmp.* | awk '{c=0;for(j=1;j<=NF;j++){c+=$j};print c/NF}'
rm tmp*
```

The script should be placed in the directory containing the displacements and its output should be redirected to a file.

