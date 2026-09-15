# Classical Optimizer Benchmarking for Variational Quantum Eigensolvers

[![Python](https://img.shields.io/badge/Python-3.12.x-blue.svg)](https://www.python.org/)
[![Qiskit](https://img.shields.io/badge/Qiskit-2.3.0-6929C4.svg)](https://qiskit.org/)
[![Qiskit Nature](https://img.shields.io/badge/Qiskit_Nature-0.7.2-6929C4.svg)](https://qiskit-community.github.io/qiskit-nature/)
[![Qiskit Machine Learning](https://img.shields.io/badge/Qiskit_Machine_Learning-0.9.0-6929C4.svg)](https://qiskit-community.github.io/qiskit-machine-learning/)
[![Qiskit Algorithms](https://img.shields.io/badge/Qiskit_Algorithms-0.4.0-6929C4.svg)](https://qiskit-community.github.io/qiskit-algorithms/)
[![Mitiq](https://img.shields.io/badge/Mitiq-1.0.0-6A5ACD.svg)](https://mitiq.readthedocs.io/)

Research code, experimental data, and analysis routines associated with:

> **Carlos H. M. Esteves, Pedro H. S. Girotto, and Daniel Leal Souza**  
> *Classical Optimizers Benchmark for Variational Quantum Eigensolvers: An Empirical and Statistical Approach*

This repository contains the computational workflow used to benchmark classical optimization algorithms within the Variational Quantum Eigensolver (VQE) for molecular ground-state energy estimation.

The study evaluates how optimizer performance changes as a function of:

- parameter initialization;
- objective-function estimation fidelity;
- finite-sampling noise;
- hardware-derived noise;
- optimizer search strategy;
- objective-function evaluation cost;
- and post-optimization error mitigation.

---

## Overview

The Variational Quantum Eigensolver is a hybrid quantum-classical algorithm in which a parameterized quantum circuit prepares a trial state while a classical optimizer updates the variational parameters in order to minimize the expectation value of the molecular Hamiltonian.

In this work, ten classical optimizer implementations are evaluated for the ground-state energy estimation of beryllium hydride, BeH₂.

The benchmark combines:

- **10 classical optimizers**
- **2 parameter-initialization protocols**
- **3 objective-evaluation regimes**
- **50 independent runs per optimizer/scenario**
- **post-optimization ideal reevaluation**
- **Zero-Noise Extrapolation (ZNE)**

The primary goal is not to identify a universal optimizer winner, but to characterize how optimizer behavior changes under different initialization, noise, and resource conditions.

---

## Molecular VQE Configuration

The benchmark uses a fixed molecular and variational configuration throughout all optimizer comparisons.

| Component | Configuration |
|---|---|
| Molecule | BeH₂ |
| H–Be bond length | 1.326 Å |
| Basis set | STO-3G |
| Active space | CAS(4,3) |
| Active electrons | 4 |
| Active spatial orbitals | 3 |
| Qubits | 6 |
| Fermion-to-qubit mapping | Jordan–Wigner |
| Reference state | Hartree–Fock |
| Ansatz | UCCSD |
| CASCI reference energy | −15.5633775664 Ha |
| Hartree–Fock energy | −15.560335 Ha |

The same logical UCCSD circuit is used across all optimizer and objective-evaluation configurations.

---

## Optimizers

The benchmark evaluates ten optimization algorithms representing four different search paradigms.

### Gradient-based methods

- Gradient Descent (GD)
- ADAM
- L-BFGS-B

Gradient information is obtained numerically through finite differences in the adopted implementations.

ADAM and L-BFGS-B use a finite-difference step of:

```text
eps = 1e-5
