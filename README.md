# Classical Optimizer Benchmarking for Variational Quantum Eigensolvers

[![Python](https://img.shields.io/badge/Python-3.12.x-blue.svg)](https://www.python.org/) [![Qiskit](https://img.shields.io/badge/Qiskit-2.3.0-6929C4.svg)](https://qiskit.org/) [![Qiskit Nature](https://img.shields.io/badge/Qiskit_Nature-0.7.2-6929C4.svg)](https://qiskit-community.github.io/qiskit-nature/) [![Qiskit Machine Learning](https://img.shields.io/badge/Qiskit_Machine_Learning-0.9.0-6929C4.svg)](https://qiskit-community.github.io/qiskit-machine-learning/) [![Qiskit Algorithms](https://img.shields.io/badge/Qiskit_Algorithms-0.4.0-6929C4.svg)](https://qiskit-community.github.io/qiskit-algorithms/) [![Mitiq](https://img.shields.io/badge/Mitiq-1.0.0-6A5ACD.svg)](https://mitiq.readthedocs.io/)

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
```

while Gradient Descent uses its implementation-default perturbation:

```text
epsilon = 0.01
```

### Derivative-free local methods

- COBYLA
- BOBYQA
- IMFIL

These methods construct search information directly from objective-function evaluations without requiring analytical gradients.

### Stochastic-approximation methods

- SPSA
- QNSPSA

SPSA estimates gradient information using simultaneous random perturbations.

QNSPSA additionally estimates the Fubini–Study metric and applies geometry-aware stochastic preconditioning.

### Swarm-based methods

- GPSO
- QPSO

Both swarm implementations use populations of 20 particles.

GPSO uses a global-best topology, while QPSO follows a quantum-behaved particle-swarm formulation based on probabilistic position updates.

---

## Experimental Design

Two initialization protocols are evaluated.

### Hartree–Fock initialization

For single-point optimizers:

```math
\theta_0 = 0
```

which preserves the Hartree–Fock reference state:

```math
U(0)|\psi_{\mathrm{HF}}\rangle = |\psi_{\mathrm{HF}}\rangle.
```

For GPSO and QPSO, one particle is initialized exactly at the Hartree–Fock point while the remaining particles are generated using a Halton low-discrepancy sequence.

### Random initialization

Each component of the designated initial parameter vector is sampled from:

```math
\theta_i \sim U(-1,1).
```

For swarm methods, one particle receives this designated random initialization while the remaining particles are generated from the Halton sequence over the search domain.

---

## Objective-Evaluation Regimes

The benchmark considers three progressively less ideal objective-function evaluation regimes.

### 1. Exact statevector simulation

Expectation values are evaluated deterministically.

This regime excludes:

- finite-sampling fluctuations;
- gate noise;
- readout noise.

It therefore isolates the interaction between the optimization algorithm and the underlying variational landscape.

### 2. Finite-shot simulation

The circuit remains ideal, but expectation values are estimated from:

```text
10,000 shots
```

This introduces statistical sampling noise into the VQE objective function.

The optimizer therefore observes a stochastic estimator:

```math
\hat{C}(\theta)
```

rather than the exact objective:

```math
C(\theta).
```

### 3. Hardware-derived noisy simulation

The third regime combines finite-shot estimation with a static noise model derived from an IBM Quantum calibration snapshot.

The selected target backend is:

```text
fake_boston
```

corresponding to a Heron 3 processor snapshot.

The noise model includes hardware-derived effects such as gate and readout errors.

---

## Six Benchmark Scenarios

Combining the three evaluation regimes with the two initialization protocols produces six experimental scenarios:

| Objective evaluation | HF initialization | Random initialization |
|---|:---:|:---:|
| Exact statevector | ✓ | ✓ |
| Finite-shot | ✓ | ✓ |
| Hardware-derived noise | ✓ | ✓ |

Each optimizer/scenario combination is evaluated across:

```text
50 runs
```

resulting in a replicated experimental benchmark rather than a single-run comparison.

---

## Random Seeds and Reproducibility

The experimental campaign uses predetermined seed schedules wherever supported by the corresponding implementation.

### Random parameter initialization

```text
127–176
```

### Aer simulator

```text
42–91
```

### GPSO / QPSO stochastic initialization and updates

```text
212–261
```

The exact statevector regime does not require simulator seeds.

Deterministic optimizers starting from the same Hartree–Fock parameter vector therefore reproduce the same trajectory under exact statevector evaluation.

---

## Parameter Bounds

A nominal parameter domain

```math
[-\pi,\pi]^d
```

is adopted throughout the study.

However, bound enforcement is implementation dependent.

Native bounds are used by:

- BOBYQA
- IMFIL
- L-BFGS-B

GPSO and QPSO explicitly clip particle positions to this domain.

The selected implementations of:

- ADAM
- COBYLA
- GD
- SPSA
- QNSPSA

operate without enforced parameter bounds.

This distinction reflects the behavior of the evaluated implementations.

---

## Optimization Budget

A nominal maximum optimization setting of:

```text
maxiter = 1000
```

is used across the benchmark.

However, **this value must not be interpreted as an equal objective-function evaluation budget**.

Its semantics differ between implementations.

For example:

- COBYLA, BOBYQA, and IMFIL primarily interpret the corresponding setting in terms of function-evaluation limits;
- gradient, stochastic, and swarm methods primarily interpret it in terms of iterations or generations.

For this reason, both convergence quality and computational cost are reported independently.

---

## NFEV Accounting

`NFEV` denotes the number of **energy-objective evaluations required by the optimization procedure**.

Evaluations performed exclusively for:

- trajectory logging;
- callbacks;
- post-optimization bookkeeping

are excluded.

For QNSPSA, the reported NFEV includes energy-objective evaluations but excludes additional fidelity evaluations required for stochastic estimation of the quantum metric.

Consequently, NFEV should be interpreted as an implementation-level cost metric rather than as evidence of an equal-resource benchmark.

---

## Hardware-Derived Noise Model

Eight IBM Quantum calibration snapshots were initially considered.

Backend selection was performed using:

- Qiskit's preset pass manager;
- SABRE layout and routing;
- optimization level 3;
- Mapomatic hardware-subgraph scoring.

For each candidate backend, 500 SABRE seeds were evaluated.

The routing configuration minimizing the number of two-qubit gates was retained, with circuit depth used as a tiebreaker.

Mapomatic was subsequently used to identify the physical-qubit subset expected to minimize accumulated hardware error.

The selected backend was:

```text
fake_boston
```

with physical qubits:

```text
[43, 42, 56, 63, 62, 61]
```

The selected transpiled circuit had:

```text
Depth:       765
2Q gates:    255
SABRE seed:  14
```

The backend selection was independently validated by evaluating the Hartree–Fock state 50 times using 40,000 shots.

---

## Post-Optimization Ideal Reevaluation

Raw energies obtained under hardware-derived noise combine two effects:

1. the quality of the optimized variational parameters;
2. the error introduced by the noisy energy estimator.

To separate these effects, every selected final parameter vector

```math
\theta^*
```

is reevaluated using an ideal statevector simulator.

This produces:

```math
E_{\mathrm{ideal}}(\theta^*)
```

which measures the energy associated with the same variational state in the absence of hardware and sampling noise.

This distinction is essential because a poor raw noisy energy does not necessarily imply a poor underlying variational state.

---

## Zero-Noise Extrapolation

Zero-Noise Extrapolation is applied **after optimization** to the parameter vectors obtained from the hardware-derived noisy regime.

It is therefore not part of the iterative optimizer benchmark.

The ZNE stage uses:

```text
40,000 shots per noise-scaled circuit
```

and the following noise scale factors:

```text
λ = {1.0, 1.5, 2.0, 2.5, 3.0}
```

Four extrapolation models are evaluated:

- Linear
- Quadratic
- Exponential
- Richardson

The implementation uses:

```text
Mitiq 1.0.0
```

The resulting energy estimates are compared against the ideal energy of the same optimized state rather than directly against the CASCI reference.

---

## Error Definitions

For an optimized parameter vector `θ*`, the analysis distinguishes three quantities.

### Residual optimization error

```math
\Delta_{\mathrm{opt}} = E_{\mathrm{ideal}}(\theta^*) - E_{\mathrm{CASCI}}
```

This evaluates the intrinsic quality of the optimized variational state.

### Raw noisy-energy deviation

```math
\Delta_{\mathrm{noise}} = E_{\mathrm{raw}}^{40}(\theta^*) - E_{\mathrm{CASCI}}
```

This represents the total deviation of the unmitigated noisy energy from the CASCI reference.

It should **not** be interpreted as estimator-only bias.

### Residual ZNE error

```math
\Delta_{\mathrm{ZNE}} = E_{\mathrm{ZNE}}(\theta^*) - E_{\mathrm{ideal}}(\theta^*)
```

This evaluates the quality of the error-mitigation procedure itself.

---

## Metrics

The benchmark reports:

- mean final energy;
- median final energy;
- standard deviation;
- mean absolute error (MAE);
- minimum energy;
- maximum energy;
- number of iterations (NIT);
- number of objective-function evaluations (NFEV);
- wall-clock execution time;
- optimization trajectories.

Final-energy accuracy is evaluated relative to:

```text
CASCI(4,3) = −15.5633775664 Ha
```

For chemical interpretation, errors below approximately:

```text
1.6 mHa
```

are discussed relative to the conventional chemical-accuracy scale.

---

## Important Interpretation of Finite-Shot Results

Under finite sampling, the measured objective is stochastic.

Therefore, sampled values may occasionally fall below the exact CASCI reference energy.

Such values are **not interpreted as violations of the variational principle**.

The optimizer observes:

```math
\hat{E}(\theta) = E(\theta) + \epsilon(\theta),
```

where `ε(θ)` represents statistical measurement fluctuations.

For this reason, the most negative sampled energy is not necessarily the most accurate result.

Final optimizer rankings are therefore based primarily on the distance from the CASCI reference rather than simply on the lowest observed energy.

---

## Convergence-Trajectory Convention

Convergence plots are displayed as functions of iteration.

For optimizers supporting callbacks, trajectories correspond directly to values generated during optimization.

BOBYQA and IMFIL do not expose equivalent callbacks in the adopted implementation.

Their energy histories are therefore reconstructed from objective-function evaluations.

Because these methods may evaluate exploratory points that do not improve the incumbent solution, their plotted trajectories are converted to a:

```text
best-so-far
```

representation.

This transformation is performed **only for visualization**.

It does not modify:

- final-energy statistics;
- raw objective evaluations;
- NFEV accounting.

---

## Main Findings

The benchmark does not identify a universal optimizer ranking.

Instead, optimizer performance is strongly conditioned by both initialization and objective-evaluation regime.

The most persistent trends observed in this study are:

- BOBYQA and IMFIL exhibit comparatively consistent behavior across the investigated conditions;
- COBYLA is particularly competitive when initialized from the Hartree–Fock reference;
- GPSO and QPSO remain competitive in difficult scenarios but require substantially larger objective-evaluation budgets;
- QNSPSA frequently outperforms SPSA, although its additional fidelity-evaluation cost prevents an equal-cost interpretation;
- the finite-difference implementations of ADAM, GD, and L-BFGS-B perform very well under favorable exact conditions but become substantially more sensitive under random initialization and noisy objectives;
- hardware-derived noise can strongly distort the measured energy scale without implying an equivalently poor underlying variational state;
- ZNE performance is both model dependent and state dependent.

---

## Software Environment

The computational workflow uses the following core software packages:

| Package | Version |
|---|---:|
| Python | 3.12.x |
| Qiskit Nature | 0.7.2 |
| Qiskit Machine Learning | 0.9.0 |
| Qiskit Algorithms | 0.4.0 |
| Qiskit Aer | 0.17.2 |
| Qiskit IBM Runtime | 0.47.0 |
| Mitiq | 1.0.0 |
| PySwarms | see environment file |
| PySCF | see environment file |
| scikit-quant | see environment file |

The original experiments were executed using Google Colab CPU resources.

A TPU v6e-1 accelerator was provisioned in the runtime environment but was **not used** for circuit simulation or classical optimization.

---

## Reproducing the Experiments

The experimental workflow should be reproduced in the following order:

1. Generate the BeH₂ molecular Hamiltonian and CAS(4,3) reference.
2. Construct the Hartree–Fock reference state and UCCSD ansatz.
3. Execute the exact-statevector benchmark.
4. Execute the finite-shot benchmark.
5. Perform hardware-backend and physical-qubit selection.
6. Execute the hardware-derived noisy benchmark.
7. Reevaluate final hardware-noise parameter vectors under ideal statevector simulation.
8. Apply Zero-Noise Extrapolation to the hardware-derived parameter vectors.
9. Generate statistical summaries, tables, and convergence plots.

The predefined seed schedules should be preserved when reproducing the original experimental campaign.

---

## Data Availability

Each run records, where applicable:

```text
final energy
optimized parameters
energy trajectory
iteration count
objective-function evaluation count
simulator seed
initialization seed
swarm seed
execution time
```

Experimental outputs are stored in structured tabular form and are used directly by the statistical-analysis routines.

No successful or unsuccessful run should be removed solely on the basis of final performance.

---

## Reproducibility Notes

Several methodological details are important when reproducing or extending this benchmark.

First, the common nominal `maxiter = 1000` setting does not imply a common quantum-resource or objective-evaluation budget.

Second, optimizer bounds differ according to implementation support.

Third, GD, ADAM, and L-BFGS-B use finite-difference gradient estimates rather than parameter-shift gradients.

Fourth, hardware-derived simulations use a static calibration snapshot and therefore do not reproduce temporal calibration drift present on a live QPU.

Finally, execution time depends on the software stack, transpiled circuit, simulator, and host CPU and should not be interpreted as an architecture-independent algorithmic metric.

---

## Scope and Limitations

The current benchmark is intentionally deep rather than chemically broad.

It considers:

```text
one molecule
one geometry
one active space
one ansatz family
```

Therefore, the reported optimizer rankings should not be generalized to VQE as a whole.

The study also does not impose equal objective- or circuit-evaluation budgets across all algorithms, and optimizer hyperparameters are fixed rather than exhaustively tuned through a symmetric optimization procedure.

Future extensions may include:

- additional molecular systems;
- potential-energy-surface geometries;
- alternative active spaces;
- different variational ansätze;
- matched objective-evaluation budgets;
- parameter-shift versus finite-difference gradient comparisons;
- hyperparameter sensitivity studies;
- repeated experiments on live quantum hardware.

---

## Associated Paper

**Carlos H. M. Esteves, Pedro H. S. Girotto, and Daniel Leal Souza**

*Classical Optimizers Benchmark for Variational Quantum Eigensolvers: An Empirical and Statistical Approach.*

If you use this repository or build upon this benchmark, please cite the associated publication once its final bibliographic information is available.

```bibtex
@article{esteves_vqe_optimizer_benchmark,
  author  = {Esteves, Carlos H. M. and Girotto, Pedro H. S. and Souza, Daniel Leal},
  title   = {Classical Optimizers Benchmark for Variational Quantum Eigensolvers: An Empirical and Statistical Approach},
  journal = {To be defined},
  year    = {2026}
}
```

The citation entry above should be updated after publication with the final journal, volume, article number, and DOI.

---

## Authors

**Carlos H. M. Esteves**  
University Centre of the State of Pará (CESUPA)  
Belém, Pará, Brazil

**Pedro H. S. Girotto**  
University Centre of the State of Pará (CESUPA)  
Belém, Pará, Brazil

**Daniel Leal Souza — Member, IEEE**  
University Centre of the State of Pará (CESUPA)  
Federal University of Pará (UFPA)  
Laboratory of Bioinspired Computing (LCBIO/UFPA)  
Belém, Pará, Brazil

---

## Funding

This research was supported by the **Amazon Foundation for the Support of Studies and Research (FAPESPA), Brazil**, through the undergraduate research activities associated with the University Centre of the State of Pará (CESUPA).

---

## Contact

For questions regarding the benchmark, reproducibility, or associated research:

**Carlos H. M. Esteves**  
📧 `estevescarlos089@gmail.com`
