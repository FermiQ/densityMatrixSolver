# Documentation for NEGF_Main.F90

## Overview

`NEGF_Main.F90` serves as the main driver program for the Non-Equilibrium Green's Function (NEGF) simulation. It orchestrates the overall workflow, which includes:
1.  Reading input parameters and allocating necessary data structures.
2.  Broadcasting initial data to all processes if running in an MPI (Message Passing Interface) environment.
3.  Iteratively calculating physical quantities (like the order parameter) for different points in k-space. This involves:
    *   Setting initial conditions for the order parameter.
    *   Entering a self-consistency loop where the Hamiltonian is built and the order parameter is recalculated until convergence is achieved or the maximum number of iterations is reached.
4.  Calculating final results after the self-consistency loop.
5.  Gathering results from all MPI processes (if applicable).
6.  Dumping the calculated data to output files.
7.  Finalizing the MPI environment.

The program is designed to simulate properties of materials, particularly topological insulators and superconductors, by solving the NEGF equations self-consistently.

## Key Components

*   **PROGRAM `NEGF_main`**:
    *   The main program unit. It controls the entire execution flow as described in the Overview.

## Important Variables/Constants

*   **`xx`, `yy`**: Loop counters for iterating over k-space points (Kx, Ky).
*   **`Kx`, `Ky`**: Current k-space coordinates (momentum components).
*   **`KxAxis`, `KyAxis`**: Arrays holding the discrete values for Kx and Ky.
*   **`NpKx`, `NpKy`**: Number of points along the Kx and Ky axes in k-space.
*   **`NpzSC`**: Number of layers corresponding to the superconducting material.
*   **`DeltaSC`**: Initial value for the s-wave order parameter in the superconductor.
*   **`DeltaAS`, `DeltaBS`**: Arrays holding the A-site and B-site components of the order parameter for each k-point and layer.
*   **`DeltaMat`**: A matrix representing the full pairing potential for each (Kx, Ky, z) point.
*   **`ite`**: Current iteration number in the self-consistency loop.
*   **`iteTot`**: Maximum number of iterations allowed for the self-consistency loop.
*   **`AmtChange`**: The magnitude of change in the order parameter between successive iterations. Used to check for convergence.
*   **`Accuracy`**: The desired precision for `AmtChange` to consider the calculation converged.
*   **`EGnd`, `EGnd0`**: Arrays to store the ground state energy.
*   **`mpisize`, `mpirank`, `mpiroot`**: MPI-related variables for parallel processing (number of processes, rank of current process, rank of the root process).

## Usage Examples

`NEGF_Main.F90` is compiled into an executable. It is typically run from the command line. The specifics of running the program (e.g., command-line arguments for input files or parameters) would depend on how `Read_In_Allocate` is implemented.

Example (conceptual):
```bash
./negf_solver input_parameters.txt
```
The program then performs the calculations and produces output files as specified in the `DumpData` subroutine.

## Dependencies and Interactions

`NEGF_Main.F90` depends on several other modules and subroutines:

*   **`NEGF_Module` (`USE NEGF_variables`)**: This module provides definitions for many variables and constants used in `NEGF_main`, including physical constants, material parameters, k-space grid details, and MPI variables.
*   **`Read_In_Allocate`**: A subroutine (presumably in a separate file) responsible for reading input parameters and allocating memory for arrays.
*   **`BroadcastDataMPI`**: A subroutine (likely in `MPI_routines.F90`) to broadcast initial data from the root MPI process to all other processes.
*   **`Build_Hamiltonian`**: A subroutine (in `Build_Hamiltonian.F90`) that constructs the total Hamiltonian `Htot` for the current `Kx`, `Ky`, and `z` values.
*   **`Calculate_OP`**: A subroutine (in `Calculate_OP.F90`) that calculates the order parameter (`DeltaMat`, `DeltaAS`, `DeltaBS`) based on the current Hamiltonian. It also updates `AmtChange`.
*   **`CalculateFinal`**: A subroutine (in `Calculate_OP.F90`) called after the self-consistency loop to compute final quantities like Local Density of States (LDOS) and band structure.
*   **`GatherDataMPI`**: A subroutine (likely in `MPI_routines.F90`) to collect results from all MPI processes onto the root process.
*   **`FinalizeMPI`**: A subroutine (likely in `MPI_routines.F90`) to properly shut down the MPI environment.
*   **`DumpData`**: A subroutine (in `Dump_Data.F90`) that writes the calculated results to output files. This is typically done only by the root process.

The main program coordinates calls to these routines in a specific sequence to perform the simulation. The self-consistency loop (`DO WHILE ((ite <= iteTot) .AND. (abs(AmtChange)>Accuracy))`) is a critical part where `Build_Hamiltonian` and `Calculate_OP` are called repeatedly.
