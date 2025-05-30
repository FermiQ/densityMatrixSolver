# Documentation for NEGF_Module.F90

## Overview

`NEGF_Module.F90` defines a Fortran module named `NEGF_variables`. This module serves as a central repository for various global variables, physical constants, material parameters, and derived types used across different parts of the NEGF (Non-Equilibrium Green's Function) simulation code. It also contains a `Kron` function for Kronecker products. By encapsulating these definitions, it promotes code organization and avoids redundant declarations in other files.

## Key Components

*   **MODULE `NEGF_variables`**:
    *   The primary component of this file. It groups all the shared data and type definitions.
*   **FUNCTION `Kron(A, B)`**:
    *   A utility function that calculates the Kronecker product of two complex matrices `A` and `B`.
    *   **Input**:
        *   `A`: A 2D complex matrix.
        *   `B`: A 2D complex matrix.
    *   **Output**: A 2D complex matrix representing the Kronecker product of `A` and `B`. The dimensions of the output matrix are `(size(A,1)*size(B,1), size(A,2)*size(B,2))`.

## Important Variables/Constants

The module declares a wide range of variables and constants. These can be broadly categorized as:

### Physical Constants
*   **`ci`**: Imaginary unit (0, 1).
*   **`c0`**: Complex zero (0, 0).
*   **`c1`**: Complex one (1, 0).
*   **`q`**: Elementary charge.
*   **`m0`**: Electron rest mass.
*   **`pi`**: Mathematical constant pi.
*   **`eps0`**: Vacuum permittivity.
*   **`kB`**: Boltzmann constant.
*   **`h`**: Planck constant.
*   **`hbar`**: Reduced Planck constant.

### Device Parameters
*   **`MM`**: Dirac Mass, critical for defining topological phases.
*   **`Norb`**: Number of orbitals (e.g., 4 for a 3D Dirac Hamiltonian).
*   **`muTI`, `muSC`**: Chemical potentials for the Topological Insulator (TI) and Superconductor (SC) regions.
*   **`UIntTI`, `UIntSC`**: Attractive Mean-Field interaction strengths for TI and SC.
*   **`tSC`**: Hopping parameter for the s-wave superconductor.
*   **`T0`**: Temperature (must be non-zero).
*   **`Mx`, `My`, `Mz`**: Components of an external Zeeman field.
*   **`Myz`**: Parameter for yz Mirror-plane breaking perturbation.
*   **`MatType`**: Integer selecting the material model (0: Model TI, 1: Liang Fu TCI, 2: Bi2Se3).
*   **`a0`**: Lattice constant in Angstroms.

### Tight-Binding Parameters
*   Parameters for specific models like Liang Fu (LF) TCI (e.g., `LFm`, `LFt11`) and Chen et al. (CF) TCI (e.g., `CFm`, `CFt1`).

### Input Parameters
*   **`nargs`**: Number of command-line arguments.
*   **`arg`**: Character string to store a command-line argument.
*   **`outputfiledir`**: Directory path for output files.
*   **`NpKx`, `NpKy`**: Number of k-points along Kx and Ky axes.
*   **`Npz`, `NpzSC`**: Number of layers in z-direction for the total system and the SC part.
*   **`NN`**: Total number of grid points (`Npz * Norb`).

### Iteration/Convergence Variables
*   **`ite`, `iteTot`**: Current and total iteration counts for self-consistency loops.
*   **`AmtChange`**: Magnitude of change between iterations (for convergence check).
*   **`RMSChange`**: Root Mean Square change.
*   **`Accuracy`**: Convergence criterion.

### Matrices
*   **Gamma Matrices (`g1`, `g2`, `g3`, `g0`, `id`)**: Standard Dirac gamma matrices and identity matrix.
*   **TCI Matrices (`Sig0`, `Sig1`, etc.)**: Matrices specific to Topological Crystalline Insulator models.
*   **Hamiltonian Component Matrices (`HLdotS`, `HSOC`, `HOnSite`, `HZeeman`, `HNN`, `HNNN`, `HHop`, etc.)**: Matrices representing different terms in the Hamiltonian.
*   **Superconducting Pairing Matrices (`gDAS`, `gDBS`, etc.)**: Gamma matrices related to different superconducting pairing symmetries.
*   **Order Parameter Arrays (`DeltaAS`, `DeltaBS`, `DeltaMat`, `DeltaMatOld`)**: Store the calculated order parameter values.
*   **Hamiltonian Matrices (`Htot`, `AATI`, `HopZTI`, `eigvecs`, `eigvals`)**: Matrices for the total Hamiltonian, its components, eigenvectors, and eigenvalues.
*   **LDOS Arrays (`LDOS`, `LDOSMx`, etc.)**: Arrays to store Local Density of States and its spin/orbital projections.
*   **`BStruc`**: Array to store band structure information.

### MPI Variables
*   **`Nprocs`**: Total number of MPI processes.
*   **`mpirank`, `mpisize`, `mpiroot`, `mpierror`, `mpistatus`**: Standard MPI control variables.

### Momentum and Energy Parameters
*   **`Kx`, `Ky`**: Current k-space coordinates.
*   **`deltaKx`, `deltaKy`**: Step size in k-space.
*   **`KxAxis`, `KyAxis`**: Arrays of k-points.
*   **`NpE`**: Number of energy points.
*   **`deltaE`**: Energy step size.
*   **`Eaxis`**: Array of energy points.

### Temporary/Dummy Variables
*   **`ii`, `jj`, `kk`, `ll`, `ee`, `ctr`, `xx`, `yy`, `zz`, `index`**: Loop counters and indices.

## Usage Examples

### Using the Module
Other Fortran files in the project include `USE NEGF_variables` at the beginning of their program or module units. This makes all public entities (variables, constants, types, functions) from `NEGF_variables` accessible.

```fortran
PROGRAM NEGF_main
    USE NEGF_variables ! Makes entities from the module available
    IMPLICIT NONE

    REAL :: local_variable
    local_variable = Kx + MM ! Kx and MM are from NEGF_variables

    ! ... rest of the program
END PROGRAM NEGF_main
```

### Using the `Kron` function
```fortran
USE NEGF_variables
IMPLICIT NONE
COMPLEX, DIMENSION(2,2) :: matrix_A, matrix_B
COMPLEX, DIMENSION(4,4) :: result_matrix

! Initialize matrix_A and matrix_B
matrix_A(1,1) = c1; matrix_A(1,2) = c0
matrix_A(2,1) = c0; matrix_A(2,2) = c1

matrix_B(1,1) = ci; matrix_B(1,2) = c1
matrix_B(2,1) = c1; matrix_B(2,2) = ci

result_matrix = Kron(matrix_A, matrix_B)

! result_matrix will now hold the Kronecker product
```

## Dependencies and Interactions

*   **Provides To**: Nearly all other Fortran files in the project (`NEGF_Main.F90`, `Build_Hamiltonian.F90`, `Calculate_OP.F90`, `Read_In_Allocate.F90`, etc.) use this module to access shared variables and constants.
*   **Dependencies**: This module itself does not have explicit dependencies on other custom modules within this project, but it relies on intrinsic Fortran capabilities. The variables it defines are populated or used by other parts of the system. For instance, `Read_In_Allocate.F90` would populate many of the input parameters, and `NEGF_Main.F90` would use iteration variables and pass various parameters to computational subroutines.
