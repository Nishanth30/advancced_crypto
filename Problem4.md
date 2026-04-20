# Problem 4: Zero-Knowledge Proof of Knowledge and Correct Evaluation of a Secret Polynomial

## 4.1 Problem Description
Polynomials play a central role in modern cryptography and form the foundation of many advanced zero-knowledge proof systems. In this assignment, the prover possesses a secret polynomial and must convince the verifier that they know this polynomial and that it evaluates correctly at a given point, without revealing the polynomial itself.

Let $G_q$ be a cyclic group of prime order $q$ with public generators $g$ and $h$, where the discrete logarithm relation between $g$ and $h$ is unknown. All arithmetic operations are performed modulo $q$.

The prover possesses a secret polynomial of degree $d$ defined as:
$$P(x) = a_0 + a_1x + a_2x^2 + \dots + a_dx^d$$
where the coefficients $a_0, a_1, \dots, a_d \in \mathbb{Z}_q$ are known only to the prover.

To commit to the polynomial, the prover generates **Pedersen commitments** to each coefficient:
$$C_i = g^{a_i} h^{r_i}, \text{ for } i = 0, 1, \dots, d$$
where each $r_i \in \mathbb{Z}_q$ is randomly chosen and kept secret.

The prover must convince the verifier that the committed polynomial evaluates to a claimed value at a given public point, without revealing the polynomial coefficients.

## 4.2 Public Inputs
The following information is publicly known to both the prover and the verifier:
- **Group parameters:** $(G_q, q, g, h)$
- **Polynomial degree:** $d$
- **Polynomial coefficient commitments:** $C_0, C_1, \dots, C_d$
- **Evaluation point:** $z \in \mathbb{Z}_q$
- **Claimed evaluation value:** $y = P(z) \in \mathbb{Z}_q$
- **Number of protocol rounds:** $k$

These values are provided as part of the assignment input files (`public.json`).

## 4.3 Private Inputs
The prover possesses the following secret information:
- **Polynomial coefficients:** $a_0, a_1, \dots, a_d$
- **Commitment randomness:** $r_0, r_1, \dots, r_d$

Such that:
- $C_i = g^{a_i} h^{r_i}, \forall i$
- $y = \sum_{i=0}^{d} a_i z^i \pmod q$

## 4.4 Goal
The prover must convince the verifier of the following statement:
> "I know polynomial coefficients corresponding to commitments $C_0, \dots, C_d$, and the polynomial evaluates to $y$ at the point $z$."

while ensuring that:
- The verifier learns **no information** about the polynomial coefficients.
- The verifier cannot reconstruct the polynomial.
- A prover who does not know valid polynomial coefficients cannot convince the verifier, except with negligible probability.

## 4.5 Protocol Requirements
The following components are implemented:
1. **Commitment Generation:** Implementation of the Pedersen commitment scheme.
2. **Interactive Zero-Knowledge Protocol:** A multi-round protocol demonstrating knowledge of coefficients consistent with commitments and evaluation.
3. **Non-Interactive Version:** Implementation using the Fiat–Shamir transform with SHA-256.
4. **Prover Program:** Takes private/public inputs and outputs commitments, $y$, and the proof transcript.
5. **Verifier Program:** Takes public inputs and proof transcript, outputting **ACCEPT** or **REJECT**.

## 4.6 Implementation Constraints
- **From Scratch:** All commitment schemes and proof protocols are implemented without external cryptographic libraries.
- **Standard Hash:** Uses SHA-256 for the Fiat-Shamir transform.
- **Language:** Pure Python using built-ins (`hashlib`, `secrets`, `json`).

## 4.7 Security Requirements
- **Completeness:** An honest prover always convinces the verifier.
- **Soundness:** A dishonest prover cannot convince the verifier of an incorrect evaluation.
- **Zero-Knowledge:** The verifier learns nothing about the polynomial coefficients.
