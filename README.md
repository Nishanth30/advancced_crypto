# ZKP Polynomial Evaluation - Project README

This project implements a Zero-Knowledge Proof (ZKP) system to prove knowledge of a secret polynomial and its correct evaluation at a public point, without revealing the polynomial coefficients.

## Project Structure

- **`main.py`**: Core library containing the ZKP implementation (Pedersen commitments, Sigma protocol) and CLI for non-interactive/simulated proofs.
- **`generate_inputs.py`**: Tool to generate cryptographic parameters ($p, q, g, h$), polynomial coefficients, and commitments.
- **`prover_interactive.py`**: Live interactive prover that communicates with the verifier via the `session/` directory.
- **`verifier_interactive.py`**: Live interactive verifier.
- **`report_problem4.tex`**: LaTeX source for the theoretical report.
- **`Problem4.md`**: The original problem description and requirements.

---

## Prerequisites

- **Python 3.10+** (Uses `secrets` and `int.to_bytes`)
- No external libraries required (uses only `hashlib`, `json`, `secrets`).

---

## Execution Guide

### 1. Setup Parameters
First, generate the public and private inputs (primes, generators, and the secret polynomial):
```bash
python3 generate_inputs.py --size small --degree 3 --rounds 5
```
This creates `public.json` and `private.json`.

### 2. Non-Interactive Proof (Fiat-Shamir)
Generate and verify a single-shot NIZK proof:
```bash
# Prover creates proof.json
python3 main.py prover --mode noninteractive

# Verifier checks proof.json
python3 main.py verifier
```

### 3. Single-Program Interactive (Simulated)
Run a $k$-round protocol where challenges are derived via hashing:
```bash
python3 main.py prover --mode interactive-hash
python3 main.py verifier
```

### 4. Live Two-Program Interactive Protocol
Simulates a real network exchange using the file system as a communication channel. Open two terminal windows:

**Terminal A (Verifier):**
```bash
python3 verifier_interactive.py
```

**Terminal B (Prover):**
```bash
python3 prover_interactive.py
```

### 5. Security Demonstration (Tamper Test)
To verify that the system correctly rejects invalid proofs:
```bash
python3 main.py prover --mode noninteractive --tamper
python3 main.py verifier  # Expected output: REJECT
```

---

## Protocol Details
The implementation uses a **Parallel Per-Coefficient Sigma Protocol** linked by a **Scalar Evaluation Check**.
- **Commitments**: Pedersen Commitments ($C_i = g^{a_i} h^{r_i} \pmod p$).
- **Fiat-Shamir**: Canonical length-prefixed encoding with domain separation (`ZKP-POLY-FS-v1:`).
- **Group**: Safe-prime subgroup $G_q \subset \mathbb{Z}_p^*$ where $p = 2q + 1$.
