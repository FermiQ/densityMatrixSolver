# Documentation for Build_Hamiltonian.F90

## Overview

`Build_Hamiltonian.F90` is responsible for constructing the total Hamiltonian matrix (`Htot`) of the system being simulated. It achieves this by calling various specialized subroutines based on the chosen material model (`MatType`). These subroutines define the on-site energies, hopping terms, spin-orbit coupling, Zeeman effects, and superconducting pairing terms for different types of materials, including topological insulators (TI), topological crystalline insulators (TCI), and s-wave superconductors.

The main subroutine, `Build_Hamiltonian`, first calls a model-specific routine (e.g., `BuildTICells`, `BuildLiangTCICells`) to set up the basic Hamiltonian blocks for the chosen material. It then assembles the full `Htot` matrix by arranging these blocks and incorporating terms for interactions between layers and superconducting pairing potentials (Bogoliubov-de Gennes framework).

## Key Components

*   **SUBROUTINE `Build_Hamiltonian`**:
    *   The primary subroutine in this file.
    *   It selects the appropriate material model based on `MatType`.
    *   Calls helper subroutines to build unit cell Hamiltonians (`AATI`, `AASC`) and hopping matrices (`HopZTI`, `HopZSC`, `HopZZTI`).
    *   Constructs the full `Htot` matrix by arranging these components, considering different properties for TI and SC layers and the interface between them.
    *   Adds attractive Hubbard-type interactions for the Bogoliubov-de Gennes (BdG) off-diagonal blocks, incorporating the order parameter `DeltaMat`.

*   **SUBROUTINE `BuildLiangTCICells`**:
    *   Constructs Hamiltonian components for a Liang Fu Topological Crystalline Insulator (TCI) model.
    *   Sets up on-site terms (`HOnSite`), spin-orbit coupling (`HSOC`), Zeeman terms (`HZeeman`), and hopping terms (`HHop`, `HHopz`).
    *   Builds the TI part of the BdG Hamiltonian, including both electron (`AATI(1:12,1:12)`, `HopZTI(1:12,1:12)`) and hole (`AATI(13:24,13:24)`, `HopZTI(13:24,13:24)`) components.
    *   Calls `BuildTCIHopping` to get k-dependent hopping matrices.

*   **SUBROUTINE `BuildTCIHopping(kxval, kyval)`**:
    *   Calculates k-dependent nearest-neighbor (`HNN`, `HNNz`) and next-nearest-neighbor (`HNNN`, `HNNNz`) hopping matrices for TCI models.
    *   These are then combined with coefficients (`LFt11`, `LFt12`, etc.) to form `HHop` and `HHopz`.
    *   **Input**: `kxval`, `kyval` (k-space coordinates).

*   **SUBROUTINE `BuildChenTCICells`**:
    *   Constructs Hamiltonian components for Chen's TCI model (assumes `Norb=8`).
    *   Sets up on-site terms and Zeeman terms for the TI part.
    *   Calculates z-direction hopping (`HopZTI`) and next-nearest-neighbor z-hopping (`HopZZTI`).
    *   Includes both electron and hole components for the BdG Hamiltonian.

*   **SUBROUTINE `BuildTICells`**:
    *   Constructs Hamiltonian components for a model Topological Insulator (assumes `Norb=8`).
    *   Includes on-site terms (Dirac mass `MM`, k-dependent terms), Zeeman terms, and z-direction hopping.
    *   Forms both electron and hole parts of the BdG Hamiltonian.

*   **SUBROUTINE `BuildBiSeCells`**:
    *   Constructs Hamiltonian components specific to Bi2Se3 (assumes `Norb=8`).
    *   Uses material-specific parameters (e.g., `mpA1`, `mpA2`, `mpC`, `mpMTI`).
    *   Sets up on-site terms, Zeeman terms, and z-direction hopping.
    *   Includes both electron and hole components for the BdG Hamiltonian.

*   **SUBROUTINE `BuildSCCells`**:
    *   Constructs Hamiltonian components for a model s-wave superconductor (assumes `Norb=8`).
    *   Sets up on-site terms (`AASC`) and z-direction hopping (`HopZSC`).
    *   Can optionally include Zeeman terms in the SC part if `fullZeeman == 1`.
    *   Forms both electron and hole parts of the BdG Hamiltonian.
    *   Defines `HopZSCTI` for hopping between SC and TI layers.

## Important Variables/Constants

*   **`Htot`**: The total Hamiltonian matrix for the system at a given (Kx, Ky). This is the main output of the `Build_Hamiltonian` subroutine.
*   **`MatType`**: Integer flag selecting the material model (0: Model TI, 1: Liang Fu TCI, 2: Chen TCI, else: Bi2Se3).
*   **`MM`**: Dirac mass, used in `BuildTICells`.
*   **`AATI`, `HopZTI`, `HopZZTI`**: Matrices representing the on-site Hamiltonian, z-direction nearest-neighbor hopping, and z-direction next-nearest-neighbor hopping for the TI part, respectively. These are populated by the material-specific cell-building subroutines.
*   **`AASC`, `HopZSC`**: Matrices representing the on-site Hamiltonian and z-direction hopping for the SC part, populated by `BuildSCCells`.
*   **`HopZSCTI`**: Hopping matrix between SC and TI layers.
*   **`Npz`, `NpzSC`, `Norb`**: System dimensions (total z-layers, SC z-layers, number of orbitals).
*   **`Kx`, `Ky`**: Current k-space coordinates, used by cell-building subroutines to compute k-dependent terms.
*   **`muTI`, `muSC`**: Chemical potentials for TI and SC regions.
*   **`UIntTI`, `UIntSC`**: Mean-field interaction strengths.
*   **`DeltaMat`, `DeltaAS`, `DeltaBS`**: Order parameter matrices/arrays, used to add pairing terms to `Htot`.
*   **`gDAS`, `gDBS`**: Gamma matrices for s-wave pairing.
*   **`HOnSite`, `HSOC`, `HZeeman`, `HHop`, `HHopz`**: Component matrices used within `BuildLiangTCICells` and `BuildTCIHopping`.
*   Tight-binding parameters (e.g., `LFm`, `LFt11`, `CFm`, `CFt1`) and Bi2Se3 specific parameters (e.g., `mpA1`, `mpMTI`).
*   Gamma matrices (`g0`, `g1`, `g2`, `g3`, `id`, `gMx`, `gMy`, `gMz`) and TCI specific matrices (`SigZ0`, `SigXX`, etc.) are used from `NEGF_Module`.

## Usage Examples

The `Build_Hamiltonian` subroutine is called within the main loop of `NEGF_Main.F90` for each (Kx, Ky) point before the order parameter is calculated.

```fortran
! In NEGF_Main.F90
! ...
DO xx = 1, NpKx
    DO yy = 1, NpKy
        ! ... (set Kx, Ky, initial DeltaMat, etc.)
        DO WHILE ((ite <= iteTot) .AND. (abs(AmtChange)>Accuracy))
            ite = ite+1
            CALL Build_Hamiltonian ! Constructs Htot based on current Kx, Ky, DeltaMat
            CALL Calculate_OP      ! Uses Htot
        ENDDO
        ! ...
    ENDDO
ENDDO
! ...
```
The choice of `MatType` (read from input) determines which internal path `Build_Hamiltonian` takes:
*   If `MatType == 0`, `BuildTICells` is called.
*   If `MatType == 1`, `BuildLiangTCICells` (which calls `BuildTCIHopping`) is called.
*   And so on for other `MatType` values.

## Dependencies and Interactions

*   **`NEGF_Module` (`USE NEGF_variables`)**: This file heavily relies on `NEGF_Module` for almost all its variables, constants, matrix dimensions, and predefined matrices (like gamma matrices).
*   **Called by**: `NEGF_Main.F90` (specifically, the `NEGF_main` program).
*   **Input Data**:
    *   `Kx`, `Ky` (from `NEGF_Main`).
    *   `MatType`, `MM`, `muTI`, `muSC`, `UIntTI`, `UIntSC`, `DeltaMat`, `DeltaAS`, `DeltaBS`, `Npz`, `NpzSC`, `Norb`, `a0`, `Mx`, `My`, `Mz`, `fullZeeman`, tight-binding parameters (all from `NEGF_Module`, ultimately from input or defaults).
*   **Output Data**:
    *   Modifies `Htot` (in `NEGF_Module`), which is then used by `Calculate_OP.F90`.
    *   Modifies helper matrices like `AATI`, `HopZTI`, `AASC`, `HopZSC` (in `NEGF_Module`).
*   The various `Build...Cells` subroutines are internal to this file and call each other as needed (e.g., `BuildLiangTCICells` calls `BuildTCIHopping`).
