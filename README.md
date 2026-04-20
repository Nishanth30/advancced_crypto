# CS6160 Problem 4 — ZKP of Polynomial Evaluation

Zero-knowledge proof of knowledge of secret polynomial coefficients and correct
evaluation at a public point `z`. Parallel per-coefficient Σ-protocol over a
prime-order subgroup of `Z_p*` with Pedersen commitments, a shared challenge,
and a scalar evaluation check. Ships both a Fiat–Shamir non-interactive mode
and two interactive variants.

## Files

| File | Role |
|---|---|
| `generate_inputs.py` | Input generator. Hand-rolled Miller–Rabin, safe-prime search, subgroup generators. Emits `public.json` + `private.json`. |
| `Problem4.py` | Prover and verifier for non-interactive (Fiat–Shamir) and interactive-hash modes. Includes `--tamper` for REJECT demo. |
| `prover_interactive.py` | Live two-program variant — prover side. File-polled message exchange. |
| `verifier_interactive.py` | Live two-program variant — verifier side. Samples fresh `c` each round. |
| `report_problem4.tex` / `.pdf` | Full LaTeX report: protocol, proofs, audit, testing. |
| `samples/` | Pre-run ACCEPT/REJECT transcripts + toy-sized inputs for manual verification. |

## Dependencies

Python 3.8+. Standard library only (`hashlib`, `secrets`, `random`, `json`, `sys`, `os`, `time`). No external crypto libraries.

## Quickstart

```bash
# 1. Generate inputs (toy size reproduces PDF numeric example)
python generate_inputs.py --size toy --h-mode hash --degree 2 --rounds 3

# 2. Prove + verify, non-interactive (Fiat-Shamir)
python Problem4.py prover --mode noninteractive
python Problem4.py verifier               # -> ACCEPT

# 3. Tamper -> REJECT
python Problem4.py prover --mode noninteractive --tamper
python Problem4.py verifier               # -> REJECT
```

## Input Generator — `generate_inputs.py`

```
python generate_inputs.py [--size toy|small|full]
                          [--h-mode hash|discard]
                          [--degree N] [--rounds N]
                          [--z N] [--seed N]
                          [--out-public public.json]
                          [--out-private private.json]
```

| Flag | Values | Meaning |
|---|---|---|
| `--size` | `toy` | `p=607, q=101, g=7`. Matches PDF example. INSECURE — demo only. |
|          | `small` | 128-bit `q`, 129-bit `p`. Safe-prime search via hand-rolled Miller–Rabin. < 1 s. |
|          | `full`  | 256-bit `q`, 257-bit `p`. A few seconds. |
| `--h-mode` | `hash` | `h` derived by hash-to-subgroup (SHA-256 → square into QR). **No trapdoor.** |
|            | `discard` | `h = g^s mod p`, `s` sampled then discarded. Trapdoor-in-principle. |
| `--degree` | N | Polynomial degree `d` (default 3). |
| `--rounds` | N | Interactive rounds `k` (default 4). Ignored by NI mode. |
| `--z` | N | Evaluation point (default: random in `Z_q`). |
| `--seed` | N | Seeds `random` module for reproducibility (non-crypto only). |

Outputs:
- `public.json` — `{p, q, g, h, d, k, commitments, z, y}`
- `private.json` — `{coeffs, randomness}` (witness — keep secret)

## Prover/Verifier — `Problem4.py`

```
python Problem4.py prover   [--mode noninteractive|interactive-hash]
                            [--public public.json] [--private private.json]
                            [--proof proof.json] [--tamper]

python Problem4.py verifier [--public public.json] [--proof proof.json]
```

Modes:
- `noninteractive` (default) — Fiat–Shamir. One round with SHA-256 challenge
  over `(domain, public, T_list, E)`.
- `interactive-hash` — `k` rounds. Per-round challenge
  `c_j = H("ZKP-POLY-INT-v1" ‖ j ‖ public ‖ T_list_j ‖ E_j) mod q`. Prover and
  verifier independently compute the same `c_j`; no extra flow needed.

`--tamper` flips one response scalar to force REJECT.

Verifier checks:
1. Subgroup membership: `pow(x, q, p) == 1` for `g, h, C_i`.
2. Scalar range: `z, y, u_{i,a}, u_{i,r}, E, c ∈ [0, q)`.
3. V1: `g^{u_{i,a}} · h^{u_{i,r}} ≡ T_i · C_i^c (mod p)` for each `i`.
4. V_eval: `Σ u_{i,a} · z^i ≡ E + c·y (mod q)`.

## Live Two-Program Variant

Runs prover and verifier as two separate processes exchanging JSON files.

```bash
# Terminal 1
python verifier_interactive.py --public public.json \
       --transcript transcript_interactive.json

# Terminal 2 (started after verifier, or simultaneously)
python prover_interactive.py --public public.json --private private.json
```

Message files per round `j` (0-indexed):
- `round_j_A.json` — prover → verifier — `{T_list, E}`
- `round_j_B.json` — verifier → prover — `{c}` (fresh `secrets.randbelow(q)`)
- `round_j_C.json` — prover → verifier — `{u_a, u_r}`
- `round_j_V.json` — verifier → prover — `{ok: bool}`

Verifier writes final `transcript_interactive.json` with all rounds logged.
Poll interval 50 ms, timeout 60 s.

## Tamper Demo (REJECT)

```bash
python Problem4.py prover --mode noninteractive --tamper
python Problem4.py verifier
# V1 check fails -> REJECT
```

## Reproduce Paper's Toy Example

```bash
python generate_inputs.py --size toy --degree 2 --rounds 3 --z 9 \
       --out-public samples/public_toy.json \
       --out-private samples/private_toy.json
python Problem4.py prover --mode noninteractive \
       --public samples/public_toy.json \
       --private samples/private_toy.json \
       --proof samples/proof_toy_ni.json
python Problem4.py verifier \
       --public samples/public_toy.json \
       --proof samples/proof_toy_ni.json
```

## Test Matrix

All 6 configs × 3 modes + tamper verified to ACCEPT/REJECT:

| `--size` | `--h-mode` | NI | int-hash | live | tamper |
|---|---|---|---|---|---|
| toy   | hash    | ✓ | ✓ | ✓ | REJECT |
| toy   | discard | ✓ | ✓ | ✓ | REJECT |
| small | hash    | ✓ | ✓ | ✓ | REJECT |
| small | discard | ✓ | ✓ | ✓ | REJECT |
| full  | hash    | ✓ | ✓ | ✓ | REJECT |
| full  | discard | ✓ | ✓ | ✓ | REJECT |

## Report

`report_problem4.pdf` — compile via:

```bash
pdflatex report_problem4.tex
pdflatex report_problem4.tex   # second pass for TOC/refs
```

Contains the protocol spec, Pedersen hiding/binding, parallel Σ-protocol,
Fiat–Shamir ROM security, full proofs of completeness / special soundness
(with extractor) / perfect HVZK, input-generator audit, and test results.

## Security Notes

- `--size toy` is INSECURE — `q=101` is brute-forceable. Use `small` / `full`
  for any real demonstration.
- `--h-mode discard` is trapdoor-in-principle. Prefer `hash`.
- Fiat–Shamir domain tag: `b"ZKP-POLY-FS-v1:"`. Interactive tag:
  `b"ZKP-POLY-INT-v1:"`.
- Soundness error per round: `1/q`.

## Authors

Divyansh Sevta, Nishanth D. — IIT Hyderabad, CS6160 Spring 2026.
