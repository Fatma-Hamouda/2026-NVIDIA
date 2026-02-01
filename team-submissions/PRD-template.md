# Product Requirements Document (PRD)

**Project Name:** LABS-QAOA  
**Team Name:** QuantumEgypt  
**GitHub Repository:** https://github.com/Fatma-Hamouda/2026-NVIDIA/tree/main

---

## 2. The Architecture
**Owner:** Project Lead (Solo)

### Choice of Quantum Algorithm
**Algorithm:** QAOA (p = 1–2) with LABS-aware cost Hamiltonian and custom mixer scheduling  

**Quantum:**  
- Cost Hamiltonian encodes the LABS energy function  
- Mixer Hamiltonian uses a problem-informed scheduling strategy to improve convergence  

**Embedding:**  
Reduce effective problem size using correlation-preserving LABS structure (N → M active qubits)  

**Classical:**  
CuPy-accelerated MTS neighbor evaluation, batching up to 1,000 sequences per kernel call  

**Motivation:**  
LABS is a natural fit for QAOA because its energy function maps directly to an Ising-style cost Hamiltonian. QAOA provides a controllable quantum heuristic whose parameters can be optimized classically and evaluated efficiently on GPUs using CUDA-Q. By combining QAOA-generated candidate solutions with a GPU-accelerated classical MTS loop, we aim to reduce time-to-solution while maintaining solution quality. This approach balances judge familiarity (QAOA) with engineering innovation (GPU batching + hybrid workflow).

### Literature Review
**Reference:** R. Shaydulin et al., "Evidence of scaling advantage for the quantum approximate optimization algorithm", [https://www.science.org/doi/10.1126/sciadv.adm6761](https://www.science.org/doi/10.1126/sciadv.adm6761)  

**Relevance:**  
This work studies QAOA performance on combinatorial optimization problems closely related to LABS and shows that shallow QAOA circuits (p = 1–2) can provide competitive approximations. It motivates our choice of low-depth QAOA and supports the feasibility of benchmarking against classical heuristics for moderate problem sizes.

---

## 3. The Acceleration Strategy
**Owner:** GPU Acceleration PIC (Solo)

### Quantum Acceleration (CUDA-Q)
**Strategy:**  
- Use CUDA-Q's `nvidia` backend to execute batched QAOA circuit evaluations  
- Perform grid sweeps over (γ, β) parameters in parallel on the GPU  
- Compare CPU backend vs GPU backend for identical circuits to quantify acceleration  

**Scaling Plan:**  
- Development and validation on CPU backend (qBraid)  
- Migration to Brev L4 GPU for performance benchmarking  
- Optional multi-GPU experiments if time and credits allow  

### Classical Acceleration (MTS)
**Strategy:**  
- Rewrite LABS energy computation from NumPy to CuPy  
- Evaluate hundreds to thousands of MTS neighbor candidates in parallel  
- Minimize host–device transfers by keeping sequences and correlation tensors resident on GPU  

### Hardware Targets
**Development:** qBraid CPU backend  
**GPU Testing:** Brev L4 (primary target)  
**Stretch Goal:** Short A100 benchmark for scaling comparison (if resources permit)

---

## 4. The Verification Plan
**Owner:** Quality Assurance PIC (Solo)

### Unit Testing Strategy
**Framework:** `pytest`  

**AI Hallucination Guardrails:**  
All AI-generated code must pass:  
- Deterministic unit tests  
- Known small-N ground-truth comparisons  
- Physical and symmetry-based property tests  

### Core Correctness Checks
**Check 1 – LABS Symmetry:** `assert energy(S) == energy(-S)`  

**Check 2 – Ground Truth (Small N):**  
For N = 3, the optimal LABS energy is known. The test suite asserts exact agreement.  

**Check 3 – QAOA Sanity:**  
- Expectation values remain within theoretical LABS bounds  
- QAOA results match brute-force evaluation for N ≤ 10  

---

## 5. Execution Strategy & Success Metrics
**Owner:** Technical Marketing PIC (Solo)

### Agentic Workflow
**Plan:**  
- Cursor IDE for coding  
- Local `skills.md` with CUDA-Q API references to reduce hallucinations  
- Iterative loop: write code → run pytest → inspect failures → refine  

### Success Metrics
**Metric 1 (Quality):** QAOA-seeded solutions match or exceed classical MTS quality for N ≤ 25  
**Metric 2 (Performance):** ≥ 10× speedup in parameter sweep evaluation on GPU vs CPU  
**Metric 3 (Scale):** Successful GPU-accelerated runs for LABS instances up to N ≈ 25–30  

### Visualization Plan
**Plot 1:** QAOA energy landscape (γ, β heatmap)  
**Plot 2:** Time-to-solution vs problem size (CPU vs GPU)  
**Plot 3:** Energy convergence (classical MTS vs QAOA-seeded MTS)

---

## 6. Resource Management Plan
**Owner:** GPU Acceleration PIC (Solo)

**Plan:**  
- CPU-only development and full test coverage  
- Short GPU sessions on Brev L4 for benchmarking  
- Manual shutdown of GPU instances after each run  
- Optional short A100 run for final scaling figure (time-boxed)

---
