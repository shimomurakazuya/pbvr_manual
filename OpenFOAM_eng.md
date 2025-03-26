# Integrating pbvr into OpenFOAM

By inserting the FoamToPbvr adapter that comes with pbvr into OpenFOAM, the OpenFOAM solver can generate particles for visualization, which is stored in the Filter directory and can be used by referencing and linking to the particle generation library like the particle sampler. The adapter converts the variable array of computed physical values into the vtkUnstructuredGrid class using the vtk library and passes it to the particle sampler.
The following shows the operation procedure of the OpenFOAM sample code attached to PBVR.

## Preferences

In this case, the sample code is run on a supercomputer (SGI8600) owned by JAEA. When operating on a separate server, the module, etc. should be read separately.

[Module] GNU/9.5.0 openMPI/4.1.4 

[OpenFOAM] 10 (Foundation version) (pre-installed ''/home/app/OpenFOAM/OpenFOAM-10'')

【サンプルコード名】 cavity flow 

【vtk】 v9.2　(pre-installed ``/home/center/viztools/VTK-9.2.2``)

【shell】 bash

The module loading command is as follows.

```
module purge; module load gnu/9.5.0　openmpi/4.1.4 
```

## Build & Install ISPBVR

Here, the path of the installation directory is ($install_dir_pbvr). 

In the installation directory, run git clone. 

```
git clone git@github.com:CCSEPBVR/CS-IS-PBVR.git -b release_v3.3.0
```


In order to create a library for OpenFOAM, change the compile settings of pbvr as shown in the figure as shown below. Change PBVR_MACHINE to gcc and VTK_PTAH to true. Also, if necessary, set the path to the pre-installed VTK library. (Default)

 <p align="center">
 <img src="https://github.com/user-attachments/assets/b7dbf750-5179-400f-87f9-6ca20a1383a6" alt="workload" width=60%>
 </p>




As soon as the configuration is complete, run the compilation.

```
make -j 40
```


## Edit OpenFOAM & Solver Compilation Settings

In order to use the pbvr library in OpenFOAM, the compilation settings are matched with pbvr. In SGI8600, the pre-installed OpenFOAM is copied locally and the settings are changed.

### Enabling OpenFOAM Commands

If the path of the OpenFOAM installation directory is ($install_dir_foam), the copy command is as follows.

```
cp -r /home/app/OpenFOAM/OpenFOAM-10  ($install_dir_foam)
cp -r /home/app/OpenFOAM/ThirdParty-10  ($install_dir_foam)
```

Once copied, set the FOAM_INST_DIR of ($install_dir_foam)/OpenFOAM-10/etc/bashrc to ($install_dir_foam) and change the environment variable WM_COMPIKER to Gcc as shown in the red line in the figure. 

 <p align="center">
 <img src="https://github.com/user-attachments/assets/ce5a7ff5-3d5e-4929-94f3-d60255144f41" alt="workload" width=40%>
 </p>


From here, enable the OpenFOAM command. First, run /etc/bashrc once. 
```
source ($install_dir_foam)/OpenFOAM-10/etc/bashrc
```

Run ./allwmake in the OpenFOAM directory.

```
cd OpenFOAM-10
./Allwmake
```

Ready to enable by running /etc/bashrc again. When the help log appears with the following command, it is enabled.

```
source ($install_dir_foam)/OpenFOAM-10/etc/bashrc
icoFOAM –help 
```

### Changing OpenFOAM compilation settings

Compilation command part ( CC = ... ) in the compile configuration file (''$install_dir_foam)/OpenFOAM-10/wmake/rules/linux64Gcc/c++'') as shown in the figure. 

 <p align="center">
 <img src="https://github.com/user-attachments/assets/465930e1-b080-41e9-82e7-afddfe8604fb" alt="workload" width=60%>
 </p>

### Changed OpenFOAM compile options

Go to the sample code directory for OpenFOAM (here, IS_DaemonAndSampler/Example/C/gcc_mpi_omp/cavity_flow_vtk) and add the path in Make/options to use the pbvr library as follows.
(Already added in the sample code)

 <p align="center">
 <img src="https://github.com/user-attachments/assets/9060cb6b-e808-4670-b677-9e2574bccb95" alt="workload" width=150%>
 </p>

This completes the configuration on the OpenFOAM side. 

## Build the sample code

Build the sample code. After compilation is complete, binaries are generated in $FOAM_APPBIN. 

```
wmake -j
```

As a pre-process, execute the mesh generation command.

```
blockMesh
decomposePar
```
## Code Execution

Set the environment variables as per the in-situ setup.

```
export VIS_PARAM_DIR=($install_dir_pbvr)/CS-IS-PBVR/IS_DaemonAndSampler/Example/C/gcc_mpi_omp/cavity_flow_vtk
export  PARTICLE_DIR=($install_dir_pbvr)/CS-IS-PBVR/IS_DaemonAndSampler/Example/C/gcc_mpi_omp/cavity_flow_vtk/particle_out
```

Create a particle_out as a directory for particle data.
```
mkdir particle_out
```

Now that the solver is ready to run, run the OpenFOAM solver. Since icoFoam.C is used as a sample code, the execution command is as follows. 

```
mpirun –n 2 icoFoam –parallel
```

## Executing Daemon Code

The procedure for running the daemon on a remote server is shown here. Note that the solver and daemon are expected to run concurrently, so it is recommended to work in a new tab (don't forget to load the module). 

The construction work of the daemon code is omitted because it is performed at the same time as the solver is built. Set the following two environment variables before execution.

```
export VIS_PARAM_DIR=($install_dir_pbvr)/CS-IS-PBVR/IS_DaemonAndSampler/Example/C/gcc_mpi_omp/cavity_flow_vtk
export  PARTICLE_DIR=($install_dir_pbvr)/CS-IS-PBVR/IS_DaemonAndSampler/Example/C/gcc_mpi_omp/cavity_flow_vtk/particle_out
```

Navigate to the Daemon directory and run the pbvr_daemon.

```
cd ($install_dir_pbvr)/CS-IS-PBVR/IS_DaemonAndSampler/Daemon
./pbvr_daemon
```
“waiting connection… If the log appears, it is successful. Since it is in a waiting state for connection, move on to connection operation from the client side without stopping. 

You have been given a tutorial on how to port forward, run a client, and connect to a remote server.

## Visualization results of sample code
The visualization result of the sample code is shown in the figure below. The visualization parameters at this time are described below.

 <p align="center">
 <img src="https://github.com/user-attachments/assets/9f9eb7b6-2bd8-4217-8a31-3d12e057a8da" alt="workload" width=80%>
 </p>

* Time step: 300 (steady state)

Transfer function
* Color Function Synthesizer : C1
* Opacity Function Synthesizer : O1
* Color Map Function [C1] : f(q1)
* Opacity Map Function[O1] : f(q1)
* minmax mode: server size Min Max

Glyph
* scale factor : 0.1
* Distribution mode: AllPoints
* Size: constant
* Color : Variables Array
  * Number of variables: 3
    * q1
    * q2
    * q3

plot over line 
* Resolution:256
* Target: q1
* Start point: (0.5, 0, 0.05)
* End point: (0.5, 1, 0.05) 

## About the sample code

### Simulation conditions

The calculation content of this sample code is a two-dimensional cavity flow. The z direction is uniform by imposing periodic boundary conditions.

The calculation parameters are as follows.
* MPI Process: 2MPI
* Wall velocity : 1(m/s)
* Number of cells (n_x, n_y, n_z): 400(20, 20, 1)
- Talk:Hexahedra
* Computing field (L_x, L_y, L_z): (1, 1, 0.5)
* Reynolds number (kinematic viscosity): 100


### About Include Files

 <p align="center">
 <img src="https://github.com/user-attachments/assets/75e10e61-750c-4304-b1db-850a092a0fd4" alt="workload" width=40%>
 </p>

The above figure is just an example, and the required files vary depending on the solver.

### PBVR function embedded & output interval control in icoFoam.C

Here, an example of embedding PBVR function in icoFoam.C and output interval control is described.

Like OpenFOAM, the adapter uses a method of inserting include files. As shown in the figure, the PBVR function is called by expanding the specified filter file (red line). (Details will be described later)

 <p align="center">
 <img src="https://github.com/user-attachments/assets/f7e5f588-3dcd-436d-88cf-1f6acd4c3c66" alt="workload" width=40%>
 </p>

There are two types of adapters, the polyhedron type and the mixed cell type. For the mixed cell type, only HEX, WEDGE, PRISM, and PYR type cells included in the cellModel Class of OpenFOAM can be visualized, but the processing speed is fast. On the other hand, the Polyhedron type can handle cells as they are, so it has a wide range of support.

There is no difference between the two usage methods described below, but I was warned to use only one of them, as mixing them can cause errors.

| Supported Types | File Name |
|-----------------|------------------------------------------------------------------|
| mixed cell type   | Conversion_from_OpenFOAM_to_vtk_***.h    |
| polyhedron type　  | Conversion_from_OpenFOAM_to_vtkpolyhedron_***.h      |

The adapter uses a .h file with (**initial_step** ) on the first use, and an .h file with (**later_step**) on the second and subsequent uses. 

## About FoamToPbvr Adapter

Here, a FoamToPbvr adapter that performs information refilling processing from OpenFOAM format to VTK format will be described.
This adapter uses vtk library functions to convert coordinates, connection information, and variable data in OpenFOAM format to the vtk format supported by the particle sampler.
Conversion processing is performed for variable data at each output step, but conversion processing is performed only for the first output step for coordinate and connection information data, reducing processing costs.
In addition, conversion processing from cell data to point data is also performed during the conversion process.


### Steps to increase variables for visualization

If you want to change or add the calculation parameters you want to visualize, you need to edit the adapter.

The adapter is stored in the ''($install_dir_pbvr)/CS-IS-PBVR/IS_DaemonAndSampler /insitulib/unstruct/filter directory.

You can increase the number of variables by copying and pasting the black frame part of the figure below and inserting the variable array to be visualized into InsertNextValue(). Be careful not to duplicate variable names. 

 <p align="center">
 <img src="https://github.com/user-attachments/assets/e9f6d60b-bf3b-4e53-a9e6-83f74d8cc815" alt="workload" width=40%>
 </p>

