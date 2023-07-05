# yAcademy - Spartan Ecdsa Audit Report 

Spartan-ecdsa is known to be the fastest open-source method to verify ECDSA (secp256k1) signatures in zero-knowledge.

**Review Resources:**

- The code repository at [github.com/personaelabs](https://github.com/personaelabs/spartan-ecdsa)
- The PersonaeLabs spartan-ecdsa blog post at [personaelabs.org](https://personaelabs.org/posts/spartan-ecdsa/)
- The PersonaeLabs spartan-ecdsa blog post at [personaelabs.org](https://personaelabs.org/posts/efficient-ecdsa-1/)
- The overview document [hackmd.io](https://hackmd.io/68TlQZFvQOeMZ2mvFl0Abg?view)

**Auditors:**

 - [0xnagu](https://github.com/thogiti/)


## Table of Contents <!-- omit in toc -->

[Executive Summary](#executive-summary)

[Scope](#scope) 

[Explanation of Findings](#explanation-of-findings) 

[Critical Findings](#critical-findings) 

[High Findings](#high-findings) 

[Medium Findings](#medium-findings) 

[Low Findings](#low-findings) 

[Final Remarks](#final-remarks) 

## Executive Summary
- There are no critical or high impact bugs in the code.
- There were some informational items, mainly under constrained input variables between the Circom circuits.
- The goal of Spartan-ECDSA was to create a fast and secure ECDSA implementation with minimal constraints. To achieve this, the Circom circuits used in Spartan-ECDSA had to be carefully checked for inputs and outputs on the client side. The team behind Spartan-ECDSA advised clients to use their typescript functions instead of the Circom circuits, unless the inputs and outputs were fully verified.

**Spartan ECDSA**

[Spartan-ecdsa](https://github.com/personaelabs/spartan-ecdsa) is an open-source implementation for verifying ECDSA (secp256k1) signatures in zero-knowledge. It is significantly faster than previous implementations, such as efficient-zk-ecdsa, and can prove ECDSA group membership 10 times faster.

The goal of Spartan-ECDSA was to create a fast and secure ECDSA with minimal constraints. To achieve this, the Circom circuits had to be carefully checked for inputs and outputs on the client side. Using the Circom circuits directly was not recommended unless the inputs and outputs were fully verified. The Spartan-ECDSA team advised the clients to use their typescript functions instead of the Circom circuits.

The major improvement in Spartan-ecdsa comes from the optimization of constraint breakdown. By using right-field arithmetic with secq and avoiding SNARK-unfriendly range checks and big integer math, Spartan-ecdsa reduces the number of constraints from 1.5 million in circom-ecdsa to 3,039 for efficient ECDSA signature verification. Furthermore, it uses efficient ECDSA signatures instead of standard ECDSA signatures, saving an additional 14,505 constraints.

Efficient ECDSA signatures consist of $s$, $T = r^{-1} * R$, and $U = -r^{-1} * m * G$, which can be computed outside of the SNARK without breaking correctness. Thus, verifying an efficient ECDSA signature requires fewer computations than a standard ECDSA signature, resulting in performance improvements.

The standard ECDSA signature consists of $(r, s)$ for a public key $Q_a$ and message $m$, where $r$ is the x-coordinate of a random elliptic curve point $R$. Standard ECDSA signature verification checks if

```math
R == m s ^{-1} * G + r s ^{-1} * Q_a
```

where $G$ is the generator point of the curve. 


**Benchmarks**

Proving membership to a group of ECDSA public keys:

|          Benchmark           |   #   |
| :--------------------------: | :---: |
|         Constraints          | 8,076 |
|   Proving time in browser    |  4s   |
|   Proving time in Node.js    |  2s   |
| Verification time in browser |  1s   |
| Verification time in Node.js | 300ms |
|          Proof size          | 16kb  |

- Measured on a M1 MacBook Pro with 80Mbps internet speed.
- Both proving and verification time in browser includes the time to download the circuit.

Please note that the performance benchmarks mentioned in the repository include the time to download the circuit in both proving and verification times in the browser.

To use Spartan-ecdsa, you can install it using `yarn add @personaelabs/spartan-ecdsa`. To build the project, follow the instructions provided in the repository's [README](https://github.com/personaelabs/spartan-ecdsa).

**Motivation**
Some applications of Spartan-Ecdsa in anonymous decentralized networks include:
1. **Blockchain Privacy:** In public blockchains like Ethereum, ECDSA can be combined with zero-knowledge proofs to enable confidential transactions, improving privacy while maintaining fast transaction times.
2. **Secure Authentication:** In authentication systems, efficient ECDSA with zero-knowledge proofs can be employed to verify users' identities without sharing their secret information (e.g., passwords). This approach minimizes the risk of password leaks and unauthorized access while maintaining fast authentication times. For example, Spartan ECDSA combined with zk-SNARKs and zk-STARKs can be utilized to facilitate private transactions without revealing sender, receiver, or transaction amount very efficiently.
3. **Private Web Verification:** Companies like Cloudflare use one-out-of-many proofs, a type of zero-knowledge proof, in combination with ECDSA for private web verification using vendor hardware. This allows for secure and efficient verification of web resources without compromising user privacy.
4. **Distributed Ledger Technology:** In distributed ledger systems, efficient ECDSA with zero-knowledge proofs can be utilized to establish secure communication channels between nodes, allowing them to verify each other's transactions and maintain data privacy. This approach ensures data integrity and confidentiality while maintaining fast transaction processing.
5. **Confidential Smart Contracts:** In smart contract platforms like Ethereum, combining efficient ECDSA with zero-knowledge proofs can enable confidential execution of smart contracts. This allows users to interact with smart contracts without revealing sensitive information, ensuring privacy and security while maintaining fast execution times.

The Spartan ECDSA Circom circuits were reviewed over 18 days. The code review was performed between June 17 and July 4 29, 2023. The Spartan ECDSA repository was under active development during the review, but the review was limited to the latest commit, [3386b30](https://github.com/personaelabs/spartan-ecdsa/commit/3386b30d9b5b62d8a60735cbeab42bfe42e80429) at the start of the review. 

The official documentation for the Spartan ECDSA circuits was located at [personaelabs.org](https://personaelabs.org/posts/spartan-ecdsa/).


## Scope

The scope of the review consisted of the following circuits at the specific commit:

- [posedion.circom](https://github.com/personaelabs/spartan-ecdsa/blob/3386b30d9b5b62d8a60735cbeab42bfe42e80429/packages/circuits/poseidon/poseidon.circom)
- [eff_ecdsa.circom](https://github.com/personaelabs/spartan-ecdsa/blob/3386b30d9b5b62d8a60735cbeab42bfe42e80429/packages/circuits/eff_ecdsa_membership/eff_ecdsa.circom)
- [pubkey_membership.circom](https://github.com/personaelabs/spartan-ecdsa/blob/3386b30d9b5b62d8a60735cbeab42bfe42e80429/packages/circuits/eff_ecdsa_membership/pubkey_membership.circom)
- [tree.circom](https://github.com/personaelabs/spartan-ecdsa/blob/3386b30d9b5b62d8a60735cbeab42bfe42e80429/packages/circuits/eff_ecdsa_membership/tree.circom)
- [add.circom](https://github.com/personaelabs/spartan-ecdsa/blob/3386b30d9b5b62d8a60735cbeab42bfe42e80429/packages/circuits/eff_ecdsa_membership/secp256k1/add.circom)
- [mul.circom](https://github.com/personaelabs/spartan-ecdsa/blob/3386b30d9b5b62d8a60735cbeab42bfe42e80429/packages/circuits/eff_ecdsa_membership/secp256k1/mul.circom)
- [double.circom](https://github.com/personaelabs/spartan-ecdsa/blob/3386b30d9b5b62d8a60735cbeab42bfe42e80429/packages/circuits/eff_ecdsa_membership/secp256k1/double.circom)

After the findings were presented to the PersonaeLabs team, fixes were made and included in several PRs.

This review is a code review to identify potential vulnerabilities in the code. The reviewers did not investigate security practices or operational security and assumed that privileged accounts could be trusted. The reviewers did not evaluate the security of the code relative to a standard or specification. The review may not have identified all potential attack vectors or areas of vulnerability.

yAcademy and the auditors make no warranties regarding the security of the code and do not warrant that the code is free from defects. yAcademy and the auditors do not represent nor imply to third parties that the code has been audited nor that the code is free from defects. By deploying or using the code, RLN and users of the contracts agree to use the code at their own risk.


Code Evaluation Matrix
---

| Category                 | Mark    | Description |
| ------------------------ | ------- | ----------- |
| Access Control           | Not applicable | The code does not explicitly implement access control mechanisms. It does not have specific checks or restrictions on who can access or modify the data. |
| Mathematics              | Good | The code includes mathematical operations such as addition, multiplication, and hashing using the Poseidon function. It also includes range checks and bit manipulation operations.  |
| Complexity               | Good | The complexity of the code is relatively low. It consists of basic mathematical operations and includes a Merkle tree inclusion proof and range checks. |
| Libraries                | Average | The code includes the Circomlib library, specifically the Poseidon circuit, which is used for hashing. |
| Decentralization         | Not applicable | The code does not explicitly address decentralization. It does not include mechanisms for distributed consensus or interaction with a decentralized network.  |
| Code stability           | Good    | The code appears to be well-structured and follows the Circom syntax. It does not contain any obvious errors or issues that would affect its stability. |
| Documentation            | Low | The code does not include extensive documentation. There are some comments explaining the purpose of certain components, but more detailed documentation would be beneficial.  |
| Monitoring               | Average | The code does not include specific monitoring mechanisms. It does not have built-in logging or tracking of events or performance metrics. |
| Testing and verification | Average | The code includes some basic range checks and a Merkle tree inclusion proof, which are important for ensuring the correctness of the code. However, it does not include comprehensive testing or verification procedures.  |

## Explanation of Findings

Findings are broken down into sections by their respective impact:
 - Critical, High, Medium, Low impact
     - These are findings that range from attacks that may cause loss of funds, impact control/ownership of the contracts, or cause any unintended consequences/actions that are outside the scope of the requirements
 - Gas savings
     - Findings that can improve the gas efficiency of the contracts
 - Informational
     - Findings including recommendations and best practices

---

## Critical Findings

None.

## High Findings

None.

## Medium Findings

None.

## Low Findings

## Informational Findings

The below are for informational purpose only as the direct use of Circom circuits are not intended. The circuits are underconstrained can lead to citicial bigs if the circom circuits are directly used without completely constraining the inputs and outputs.

**circuits/eff_ecdsa_membership/secp256k1/add.circom**

warning: Using the signal assignment operator `<--` does not constrain the assigned signal.
   ┌─ /Users/nagumba/Documents/Projects/yacademy/spartan-ecdsa/packages/circuits/eff_ecdsa_membership/secp256k1/add.circom:31:5
```circom
31 │     lambda <-- dy / dx;
   │     ^^^^^^^^^^^^^^^^^^ The assigned signal `lambda` is not constrained here.
   = For more details, see https://github.com/trailofbits/circomspect/blob/main/doc/analysis_passes.md#signal-assignment.
```

warning: In signal assignments containing division, the divisor needs to be constrained to be non-zero
   ┌─ /Users/nagumba/Documents/Projects/yacademy/spartan-ecdsa/packages/circuits/eff_ecdsa_membership/secp256k1/add.circom:31:21
```circom
31 │     lambda <-- dy / dx;
   │                     ^^ The divisor `dx` must be constrained to be non-zero.
   = For more details, see https://github.com/trailofbits/circomspect/blob/main/doc/analysis_passes.md#unconstrained-division.
```

circomspect: analyzing template 'Secp256k1AddComplete'
warning: Using the signal assignment operator `<--` does not constrain the assigned signal.
   ┌─ /Users/nagumba/Documents/Projects/yacademy/spartan-ecdsa/packages/circuits/eff_ecdsa_membership/secp256k1/add.circom:75:5
```circom
75 │     signal lambdaA <-- ((yQ - yP) / dx) * (1 - isXEqual.out);
   │     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ The assigned signal `lambdaA` is not constrained here.
   = For more details, see https://github.com/trailofbits/circomspect/blob/main/doc/analysis_passes.md#signal-assignment.
```

warning: Using the signal assignment operator `<--` does not constrain the assigned signal.
   ┌─ /Users/nagumba/Documents/Projects/yacademy/spartan-ecdsa/packages/circuits/eff_ecdsa_membership/secp256k1/add.circom:79:5
```circom
79 │     signal lambdaB <-- ((3 * xPSquared) / (2 * yP));
   │     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ The assigned signal `lambdaB` is not constrained here.
   = For more details, see https://github.com/trailofbits/circomspect/blob/main/doc/analysis_passes.md#signal-assignment.
```

warning: In signal assignments containing division, the divisor needs to be constrained to be non-zero
   ┌─ /Users/nagumba/Documents/Projects/yacademy/spartan-ecdsa/packages/circuits/eff_ecdsa_membership/secp256k1/add.circom:79:44
```circom
79 │     signal lambdaB <-- ((3 * xPSquared) / (2 * yP));
   │                                            ^^^^^^ The divisor `(2 * yP)` must be constrained to be non-zero.
   │
   = For more details, see https://github.com/trailofbits/circomspect/blob/main/doc/analysis_passes.md#unconstrained-division.
```

**circomspect ./eff_ecdsa_membership/secp256k1/mul.circom**

warning: Using the signal assignment operator `<--` does not constrain the assigned signal.
    ┌─ /Users/nagumba/Documents/Projects/yacademy/spartan-ecdsa/packages/circuits/eff_ecdsa_membership/secp256k1/mul.circom:124:5
```circom
124 │     signal shi <-- s >> 128;
    │     ^^^^^^^^^^^^^^^^^^^^^^^ The assigned signal `shi` is not constrained here.
    = For more details, see https://github.com/trailofbits/circomspect/blob/main/doc/analysis_passes.md#signal-assignment.
```

warning: Using the signal assignment operator `<--` does not constrain the assigned signal.
    ┌─ /Users/nagumba/Documents/Projects/yacademy/spartan-ecdsa/packages/circuits/eff_ecdsa_membership/secp256k1/mul.circom:123:5
```circom
123 │     signal slo <-- s & (2 ** (128) - 1);
    │     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ The assigned signal `slo` is not constrained here.
    = For more details, see https://github.com/trailofbits/circomspect/blob/main/doc/analysis_passes.md#signal-assignment.
```

warning: Using `Num2Bits` to convert field elements to bits may lead to aliasing issues.
    ┌─ /Users/nagumba/Documents/Projects/yacademy/spartan-ecdsa/packages/circuits/eff_ecdsa_membership/secp256k1/mul.circom:180:25
```circom
180 │     component kloBits = Num2Bits(256);
    │                         ^^^^^^^^^^^^^ Circomlib template `Num2Bits` instantiated here.
    │
    = Consider using `Num2Bits_strict` if the input size may be >= than the prime size.
    = For more details, see https://github.com/trailofbits/circomspect/blob/main/doc/analysis_passes.md#non-strict-binary-conversion.
```

warning: Using `Num2Bits` to convert field elements to bits may lead to aliasing issues.
    ┌─ /Users/nagumba/Documents/Projects/yacademy/spartan-ecdsa/packages/circuits/eff_ecdsa_membership/secp256k1/mul.circom:183:25
```circom
183 │     component khiBits = Num2Bits(256);
    │                         ^^^^^^^^^^^^^ Circomlib template `Num2Bits` instantiated here.
    │
    = Consider using `Num2Bits_strict` if the input size may be >= than the prime size.
    = For more details, see https://github.com/trailofbits/circomspect/blob/main/doc/analysis_passes.md#non-strict-binary-conversion.
```

To improve error handling in the given Circom code, you can add assertions and constraints to check for invalid inputs and edge cases. Here are a few suggestions:
 1. Check if the input scalar is within the valid range:
 Add a constraint to ensure that the input scalar is within the valid range of the Secp256k1 elliptic curve. You can do this by adding an assertion to check if the scalar is less than the curve's order.
```circom
// Add this line after the signal input scalar declaration
assert(scalar < 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141);
```

2. Check if the input point is on the curve:
 Add a constraint to ensure that the input point $(xP, yP)$ lies on the Secp256k1 curve. You can do this by checking the curve equation: $y^2 = x^3 + 7$.
```circom
// Add these lines after the signal input xP and yP declarations
var a = 0;
var b = 7;
assert(yP * yP === xP * xP * xP + a * xP + b);
```

3. Check for edge cases:
 For the edge cases where the input point is the point at infinity or the scalar is zero, the output point should also be the point at infinity. You can add an assertion to check for these cases.
```circom
// Add these lines after the outX and outY signal output declarations

var isPointAtInfinity = (xP === 0 && yP === 0) || scalar === 0;
assert((isPointAtInfinity && outX === 0 && outY === 0) || (!isPointAtInfinity));
```
These additional checks will help improve the error handling of the given Circom code by ensuring that the inputs are valid and handling edge cases properly.


**circuits/eff_ecdsa_membership/secp256k1/double.circom**

circomspect: analyzing template 'Secp256k1Double'
warning: Using the signal assignment operator `<--` does not constrain the assigned signal.
   ┌─ /Users/nagumba/Documents/Projects/yacademy/spartan-ecdsa/packages/circuits/eff_ecdsa_membership/secp256k1/double.circom:22:5
```circom
22 │     lambda <-- (3 * xPSquared) / (2 * yP);
   │     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ The assigned signal `lambda` is not constrained here.
   = For more details, see https://github.com/trailofbits/circomspect/blob/main/doc/analysis_passes.md#signal-assignment.
```

warning: In signal assignments containing division, the divisor needs to be constrained to be non-zero
   ┌─ /Users/nagumba/Documents/Projects/yacademy/spartan-ecdsa/packages/circuits/eff_ecdsa_membership/secp256k1/double.circom:22:35
```circom
22 │     lambda <-- (3 * xPSquared) / (2 * yP);
   │                                   ^^^^^^ The divisor `(2 * yP)` must be constrained to be non-zero.
   │
   = For more details, see https://github.com/trailofbits/circomspect/blob/main/doc/analysis_passes.md#unconstrained-division.
```
**circuits/eff_ecdsa_membership/eff_ecdsa.circom**

Someone can generate fake witnesses by randomly selecting $s1$, $T1$ and calculate $U1 = Pubkey - s1*T1$. The circuit doesn't check/verify $s1$, $T1$, and $U1$ constraints, it merely accepts and passes through the circuit. 

There should be similar informational warnings to the client implementations for many edge cases like zero point, points at infinity, additions/multiplications with $p & $-p$, etc.


### `POSEIDON` and Some Additional Remarks
- The Spartan-Ecdsa circuits assume that the underlying hash function (`Poseidon`) is:
    * Collision-resistant
    * Resistant to differential, algebraic, and interpolation attacks
    * Behaves as a random oracle
- The Merkle tree used for membership proof is assumed to be secure against second-preimage attacks.
- Social engineering attacks are still a valid way to break the system.
- ECDSA has several nonce based attacks. It is very important that the client side confirguration doesn't leak any nonce data or any app metadata that can reduce the security of guessing nonce for the ECDSA.

## Final remarks
Overall, the code demonstrates good implementation of mathematical operations and basic functionality. However, it could benefit from more extensive documentation and additional testing and verification procedures.
