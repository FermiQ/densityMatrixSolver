# Documentation for Calculate_OP.F90

## Overview

`Calculate_OP.F90` contains subroutines crucial for the self-consistent calculation of the superconducting order parameter and for computing final physical quantities once convergence is achieved. The primary subroutine, `Calculate_OP`, determines the order parameter matrix (`DeltaMat`) by summing over contributions from eigenvectors and eigenvalues of the Hamiltonian. It also calculates the change in the order parameter (`AmtChange`) from the previous iteration, which is used by the main program to check for self-consistency. The second subroutine, `CalculateFinal`, is called after the self-consistency loop to compute observable quantities like the Local Density of States (LDOS), band structure, and ground state energy.

## Key Components

*   **SUBROUTINE `Calculate_OP`**:
    *   Resets order parameter arrays (`DeltaAS`, `DeltaBS`, `DeltaMat`, etc.) for the current (xx, yy) k-point.
    *   Calls `get_evalsevecs` (an external MKL routine, presumably) to diagonalize the total Hamiltonian `Htot` (provided as `eigvecs` input, which gets overwritten) and obtain its eigenvalues (`eigvals`) and eigenvectors (`eigvecs`).
    *   Calculates the order parameter matrix `DeltaMat(xx,yy,zz,jj,kk)` by summing over the positive energy eigenstates, using the formula:
        `DeltaMat -= conjg(eigvecs(hole_idx, E_idx)) * eigvecs(particle_idx, E_idx) * tanh(eigvals(E_idx)/(2*kB*T0))`
    *   Derives various components of the order parameter (`DeltaAS`, `DeltaBS`, `DeltaABS`, and their spin-polarized variants like `DeltaAPUp`, `DeltaAPDn`) from `DeltaMat` using predefined pairing gamma matrices (`gDAS`, `gDBS`, etc.).
    *   If `ite >= 10`, calculates `AmtChange` (maximum absolute difference) and `RMSChange` (root mean square difference) between the current `DeltaMat` and `DeltaMatOld` (from the previous iteration) to quantify convergence.
    *   Updates `DeltaMatOld` with the current `DeltaMat` for the next iteration.

*   **SUBROUTINE `CalculateFinal`**:
    *   Calculates spin-resolved and orbital-resolved Local Density of States (LDOS). For each (xx, yy, zz) point and energy `Eaxis(ee)`:
        `LDOS(xx,yy,zz,ee) += exp(-1e3*(Eaxis(ee)-eigvals(ii))^2) * real(conjg(PsiKet(kk))*PsiKet(ll)*id(kk,ll))`
        Similar expressions are used for `LDOSMx`, `LDOSMy`, `LDOSMz` (spin-projected LDOS) and `LDOSPx`, `LDOSPy`, `LDOSPz` (orbital-projected LDOS), using `gMx`, `gMy`, `gMz`, `gPx`, `gPy`, `gPz` matrices respectively. `PsiKet` here represents the particle part of an eigenvector.
    *   Stores the lowest few positive energy eigenvalues (`eigvals(NN/2+1:NN/2+5)`) into `BStruc(xx,yy,1:5)` to represent the band structure.
    *   Calculates the ground state energy `EGnd(xx,yy)` by summing negative energy eigenvalues and subtracting chemical potential contributions.
    *   Calculates `EGnd0` (zeroth order ground state energy before Mean-Field Theory subtraction).
    *   Subtracts Mean-Field Theory (MFT) components related to the calculated order parameters (`DeltaAS`, `DeltaBS`, etc.) from `EGnd(xx,yy)` to get the final ground state energy per layer.

## Important Variables/Constants

*   **`DeltaMat(xx,yy,zz,jj,kk)`**: The primary calculated quantity in `Calculate_OP`. It's the matrix form of the superconducting order parameter for a given k-point (xx,yy) and layer (zz).
*   **`DeltaAS`, `DeltaBS`, `DeltaABS`, `DeltaAPUp`, etc.**: Various projections of the order parameter, derived from `DeltaMat`.
*   **`DeltaMatOld(zz,ii,jj)`**: Stores `DeltaMat` from the previous iteration to calculate `AmtChange`.
*   **`eigvecs`**: Input: `Htot`. Output: Eigenvectors of `Htot`. (Overwritten by `get_evalsevecs`)
*   **`eigvals`**: Eigenvalues of `Htot`.
*   **`AmtChange`, `RMSChange`**: Measures of convergence for the order parameter.
*   **`Htot`**: The total Hamiltonian matrix (input to `get_evalsevecs` via `eigvecs`).
*   **`NN`**: Total size of the Hamiltonian matrix (`Npz * Norb`).
*   **`Norb`**: Number of orbitals.
*   **`Npz`**: Number of layers in z-direction.
*   **`xx`, `yy`**: Indices for the current k-point.
*   **`kB`, `T0`**: Boltzmann constant and temperature, used in the `tanh` factor.
*   **`LDOS`, `LDOSMx`, `LDOSMy`, `LDOSMz`, `LDOSPx`, `LDOSPy`, `LDOSPz`**: Arrays to store calculated Local Density of States and its projections.
*   **`Eaxis`, `NpE`**: Array of energy points and number of energy points for LDOS calculation.
*   **`BStruc`**: Array to store band structure information (lowest positive eigenvalues).
*   **`EGnd`, `EGnd0`**: Arrays for ground state energy.
*   **`muTI`, `muSC`**: Chemical potentials for TI and SC.
*   **`UIntTI`, `UIntSC`**: Mean-field interaction strengths for MFT subtraction from `EGnd`.
*   Pairing gamma matrices (`gDAS`, `gDBS`, `gDABS`, `gDAPUp`, etc.) and projection gamma matrices (`gMx`, `gMy`, `gMz`, `gPx`, `gPy`, `gPz`, `id`) are used from `NEGF_Module`.

## Usage Examples

### `Calculate_OP`
This subroutine is called inside the self-consistency loop in `NEGF_Main.F90`:
```fortran
! In NEGF_Main.F90
! ...
DO WHILE ((ite <= iteTot) .AND. (abs(AmtChange)>Accuracy))
    ite = ite+1
    CALL Build_Hamiltonian
    CALL Calculate_OP  ! Calculates DeltaMat, AmtChange using Htot
ENDDO
! ...
```

### `CalculateFinal`
This subroutine is called after the self-consistency loop in `NEGF_Main.F90` for each k-point that the current MPI process is responsible for:
```fortran
! In NEGF_Main.F90
! ...
! After the DO WHILE loop for self-consistency:
CALL CalculateFinal ! Calculates LDOS, BStruc, EGnd using converged eigenvectors/values
! ...
```

## Dependencies and Interactions

*   **`NEGF_Module` (`USE NEGF_variables`)**: This file heavily relies on `NEGF_Module` for most of its input variables (like `xx`, `yy`, `Htot` via `eigvecs`, `kB`, `T0`, `Norb`, `NN`, `Npz`, various gamma matrices, `Eaxis`, `NpE`, `muTI`, `muSC`, `UIntTI`, `UIntSC`) and for storing its results (`DeltaMat`, `DeltaAS`, etc., `AmtChange`, `LDOS` arrays, `BStruc`, `EGnd`).
*   **`get_evalsevecs` (External Subroutine)**: This is a crucial dependency, presumably an MKL (Math Kernel Library) routine or a similar library function, used for eigenvalue decomposition of the Hamiltonian `Htot`. It takes `Htot` (via `eigvecs` which is dimensioned `NN x NN`) and returns `eigvals` (dimension `NN`) and `eigvecs` (overwritten with eigenvectors as columns).
*   **Called by**: `NEGF_Main.F90`.
*   **Input Data**:
    *   For `Calculate_OP`: `Htot` (passed as `eigvecs` which is then overwritten), `xx`, `yy`, `Norb`, `NN`, `Npz`, `kB`, `T0`, `ite`, `DeltaMatOld`, various gamma matrices from `NEGF_Module`.
    *   For `CalculateFinal`: Converged `eigvecs` and `eigvals`, `xx`, `yy`, `Norb`, `NN`, `Npz`, `Eaxis`, `NpE`, `muTI`, `muSC`, `UIntTI`, `UIntSC`, various gamma matrices from `NEGF_Module`.
*   **Output Data**:
    *   `Calculate_OP` modifies: `DeltaMat`, `DeltaAS`, `DeltaBS`, etc., `AmtChange`, `RMSChange`, `DeltaMatOld` (all in `NEGF_Module`). It also overwrites the input `eigvecs` with eigenvectors and fills `eigvals`.
    *   `CalculateFinal` modifies: `LDOS` arrays, `BStruc`, `EGnd`, `EGnd0` (all in `NEGF_Module`).
