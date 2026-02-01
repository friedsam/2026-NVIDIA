# PRD: Quantum-Enhanced Optimization for LABS (QE-MTS)

## 1. Problem Statement
The **Low Autocorrelation Binary Sequences (LABS)** problem is characterized by a "rugged" energy landscape where classical heuristics often converge to high-energy local minima. State-of-the-art **Memetic Tabu Search (MTS)** is powerful but remains sensitive to its initial state. There is a critical need for a high-quality "warm-start" mechanism to bias the search toward promising regions of the solution space.

## 2. Solution Objectives
* **Hybrid Integration:** Develop a seamless data-offloading pipeline between **CUDA-Q** (Quantum Heuristic) and **NumPy-based MTS** (Classical Refinement).
* **Quantum Warm-Start:** Utilize a Digitized Counterdiabatic (DC) circuit to produce initial populations with lower average energy than uniform random distributions.
* **Scalability Path:** Design the workflow to graduate from small-$N$ CPU simulation to GPU-accelerated large-scale optimization.

## 3. Technical Approach
* **Heuristic Generation:** Implement a Trotterized CUDA-Q kernel using $R_{ZZ}$ and $R_{YZZZ}$ gates to approximate the LABS Hamiltonian.
* **Stochastic Seeding:** Sample the QPU/Simulator to generate a diversified population of bitstrings, mapped from $\{0,1\}^N$ to the LABS spin-representation $\{-1,1\}^N$.
* **Classical Refinement:** Execute an MTS loop consisting of tournament selection, uniform crossover, and a local Tabu Search to refine quantum-generated candidates.



## 4. Phase 1: Prototype Validation (Current)
* **Functional Goal:** Validate the end-to-end toolchain on qBraid for $N=20$.
* **Key KPI:** Achieving a lower "Initial Mean Energy" in the quantum-seeded population compared to a random baseline.
* **Constraint Handling:** Prioritized kernel executability and toolchain stability over high-depth Trotter circuits to ensure robust data hand-off.

## 5. Verification & Insights
* **Empirical Success:** Demonstrated a **12.5% reduction** in initial average energy (**157.6 vs. 180.2**), confirming the quantum circuit successfully biases the search.
* **Convergence Stability:** Both Random and Quantum branches successfully reached a local minimum of $E=26.0$, verifying the integrity of the MTS logic.
* **Diversity Trade-off:** Identified that excessive Trotter steps ($n=5$) without parameter tuning can lead to premature convergence (**$E=34.0$**), highlighting the need for maintaining "Quantum Diversity" in hybrid populations.

## 6. Phase 2: Scalability & GPU Acceleration
To transition from a prototype to a large-scale discovery engine (N > 40), the following GPU-centric optimizations are required:

### 6.1 Parallel Energy Kernel
The current bottleneck is the $O(N^2)$ energy evaluation. Phase 2 will port this to a **CUDA-Q kernel**, utilizing parallel reduction to evaluate autocorrelation across thousands of threads simultaneously.



### 6.2 Massively Parallel Tabu Search (MPTS)
We will implement a GPU-accelerated neighborhood search. By launching $N$ threads per sequence, we can evaluate an entire 1-bit flip neighborhood in $O(\log N)$ time, allowing for significantly higher `tabu_iters` and deeper exploration of the energy landscape.

### 6.3 Symmetry-Aware Pre-processing
A GPU-based "Symmetry Filter" will normalize quantum seeds (handling spin-flip and bit-reversal symmetries) before the classical search begins. This ensures the search budget is spent only on unique, non-redundant regions of the Hilbert space.

---

## 7. Expected Impact
By combining the **Quantum Seed** (Phase 1) with **Massively Parallel Refinement** (Phase 2), this architecture is designed to maintain a 10-15% energy advantage over classical-only approaches while scaling to sequence lengths that are currently computationally prohibitive.
