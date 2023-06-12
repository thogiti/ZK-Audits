# yAcademy - Rate Limiting Nullifier Review

**Review Resources:**

- The code repository at [github.com/Rate-Limiting-Nullifier](https://github.com/Rate-Limiting-Nullifier/circom-rln)
- The RLN V1 specification document at [rfc.vac.dev](https://rfc.vac.dev/spec/32)
- The RLN V2 specification document at [rfc.vac.dev](https://rfc.vac.dev/spec/58)

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

**Rate Limiting Nullifier**

Rate limiting nullifier (RLN) is a construct based on zero-knowledge proofs (sometimes called ZKP gadget) that provides an anonymous rate-limited signaling/messaging framework suitable for decentralized (and centralized) environments using Secret Shamir Sharing (SSS) scheme.

**Motivation**
Some applications of rate limiting nullifier in anonymous decentralized networks include:
1. Decentralized voting applications: RLN helps prevent voting outcomes from being manipulated by spam or sybil attacks, ensuring the integrity of the voting process. 
2. Anonymous group chat applications: By preventing users from spamming or polluting group chats, RLN enhances the user experience in these applications.
3. Direct anonymous attestation: RLN can be used in combination with Direct Anonymous Attestation (DAA) to implement service rate-limiting in a scenario where messages between users and the service are sent anonymously while preserving message unlinkability
4. Blockchain-based social networks: RLN can be applied to decentralized social media networks to prevent spam, sybil attacks, and other types of abuse targeting APIs and applications, thus enhancing the overall security and reliability of these networks.
5. Rate limiting in web applications: RLN can be integrated with Web Application Firewalls (WAF) to protect against denial-of-service attacks, brute-force login attempts, and API traffic surges, providing a more secure and reliable web application experience. 

**RLN V2**
The RLN V2 protocol is a more general construct, that allows to set various limits for an epoch (it’s 1 message per epoch in RLN-V1) while remaining almost as simple as it predecessor. Moreover, it allows to set different rate-limits for different RLN app users based on some public data, e.g. stake.

The RLN Circom circuits were reviewed over 13 days. The code review was performed between May 31 and June 12, 2023. The RLN repository was under active development during the review, but the review was limited to the latest commit, [37073131b9](https://github.com/Rate-Limiting-Nullifier/circom-rln/tree/37073131b9c5910228ad6bdf0fc50080e507166a) at the start of the review. 

The official documentation for the RLN circuits was located at [rate-limiting-nullifier.github.io](https://rate-limiting-nullifier.github.io/rln-docs/).

## Scope

The scope of the review consisted of the following circuits at the specific commit:

- [rln.circom](https://github.com/Rate-Limiting-Nullifier/circom-rln/blob/37073131b9c5910228ad6bdf0fc50080e507166a/circuits/rln.circom)
- [utils.circom](https://github.com/Rate-Limiting-Nullifier/circom-rln/blob/37073131b9c5910228ad6bdf0fc50080e507166a/circuits/utils.circom)
- [withdraw.circom](https://github.com/Rate-Limiting-Nullifier/circom-rln/blob/37073131b9c5910228ad6bdf0fc50080e507166a/circuits/withdraw.circom)

After the findings were presented to the RLN team, fixes were made and included in several PRs.

This review is a code review to identify potential vulnerabilities in the code. The reviewers did not investigate security practices or operational security and assumed that privileged accounts could be trusted. The reviewers did not evaluate the security of the code relative to a standard or specification. The review may not have identified all potential attack vectors or areas of vulnerability.

yAcademy and the auditors make no warranties regarding the security of the code and do not warrant that the code is free from defects. yAcademy and the auditors do not represent nor imply to third parties that the code has been audited nor that the code is free from defects. By deploying or using the code, RLN and users of the contracts agree to use the code at their own risk.


Code Evaluation Matrix
---

| Category                 | Mark    | Description |
| ------------------------ | ------- | ----------- |
| Access Control           | Good | TODO |
| Mathematics              | Good | TODO |
| Complexity               | Good | TODO |
| Libraries                | Average | TODO |
| Decentralization         | Good | TODO |
| Code stability           | Good    | TODO |
| Documentation            | Low | TODO |
| Monitoring               | Average | TODO |
| Testing and verification | Average | TODO  |

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

### 1. Low - Incosistency between RLN contract and RLN circuit on the number of bits for userMessageLimit

In RLN.sol, the messageLimit can take upto 2**256 - 1 values whereas messageId & userMessageLimit values in circuits is restricted to 2**16 - 1 .

[rln.circom](https://github.com/Rate-Limiting-Nullifier/circom-rln/blob/37073131b9c5910228ad6bdf0fc50080e507166a/circuits/rln.circom)

```circom
template RLN(DEPTH, LIMIT_BIT_SIZE) {
...
    // messageId range check
    RangeCheck(LIMIT_BIT_SIZE)(messageId, userMessageLimit);
...
}
component main { public [x, externalNullifier] } = RLN(20, 16);
```

[rln.sol](https://github.com/Rate-Limiting-Nullifier/rln-contracts/blob/main/src/RLN.sol)
```solidity
uint256 messageLimit = amount / MINIMAL_DEPOSIT;
```

**Recommended Solution**

Update the relevant code at [rln.sol](https://github.com/Rate-Limiting-Nullifier/rln-contracts/blob/main/src/RLN.sol) with something like below:

```solidity
function register(uint256 identityCommitment, uint256 amount) external {
        ...
        uint256 messageLimit = amount / MINIMAL_DEPOSIT;
        require( messageLimit <= type(uint16).max , "Max length of your message limit is 65535");
        ...
    }
```



### 2. Low - TODO_Title



## Final remarks

