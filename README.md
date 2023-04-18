# PepeBridge RIDE security audit

- [PepeBridge RIDE security audit](#pepebridge-ride-security-audit)
  - [Project summary](#project-summary)
  - [Survey scope](#survey-scope)
  - [Overall](#overall)
  - [Administration policy](#administration-policy)
  - [Severity levels](#severity-levels)
  - [Vulnerability summary](#vulnerability-summary)
  - [Vulnerabilities appendix](#vulnerabilities-appendix)
    - [Reinitialization](#reinitialization)
    - [Unauthorized permissions](#unauthorized-permissions)
    - [Overflows \& underflows](#overflows--underflows)
    - [Reentrancy](#reentrancy)
    - [Unauthorized fund transfer](#unauthorized-fund-transfer)
    - [Token lockout](#token-lockout)
    - [Unauthorized contract termination](#unauthorized-contract-termination)
    - [Unexpected halt](#unexpected-halt)
    - [External call dependency](#external-call-dependency)
    - [Deterministic randomness](#deterministic-randomness)
    - [Conflicting state updates](#conflicting-state-updates)
    - [Complexity overflow](#complexity-overflow)
    - [Complexity optimization](#complexity-optimization)
    - [Incorrect payment](#incorrect-payment)
    - [Zero payment](#zero-payment)
    - [Input validation](#input-validation)
    - [Serialization](#serialization)
    - [Logic flaws](#logic-flaws)
    - [Front-running](#front-running)
    - [Timestamp manipulation](#timestamp-manipulation)
    - [Insider attack](#insider-attack)
    - [Upgradeability](#upgradeability)
    - [Litepaper compliance](#litepaper-compliance)

## Project summary

The litepaper of the project located here: https://github.com/crypto-pepe/research-and-development/blob/master/bridge/lite_paper.ipynb

PepeBridge project is a cross-chain interaction protocol which consists of a set of smart contracts and software modules that enable cross-platform interaction. Peripheral smart contracts or scripts are used to lock transferred tokens and confirm transaction execution. Witness smart contracts store information about confirmed transactions, validate evidence from witnesses, and determine their roles and responsibilities. Signers are responsible for generating signatures to confirm cross-chain transfers, and different groups of signers can use different signature algorithms. The protocol also includes components such as a tax provider smart contract, vault for storing and managing protocol provisioning, witness proxy for transaction confirmation, and a relayer for simplifying token delivery to target users. The goal of the protocol is to simplify user scenarios for cross-chain transfers and enable future upgrades for new types of interactions.

## Survey scope

PepeBridge project code is investigated up to and including April 14, 2023.

The object of the study is the source code of 2 group of RIDE contracts located at:
- Crosschain iteraction protocol (CIP): https://github.com/crypto-pepe/cip-waves-contracts/tree/94f9765a602cd1294db5dca1d080bda086d0de69
- Waves Bridge: https://github.com/crypto-pepe/bridige-waves-contracts/tree/0b3607f93f9ada03ed03424ab006a534d9554648

The following assumes that two independent subsystems of the protocol are being considered.

## Overall

The structure of the contracts makes it easy to conduct audits even without a comprehensive understanding of the overall scheme. The functions in the contracts are well-structured, with pre-actions executed upon entering each public function that immediately provide an understanding of the level of trust and coherence of the function with other system functions. This significantly simplifies the audit process. While some user functions had validation data errors that could have been exploited to increase trust in subsequent calls, the contract structure allowed for corrections to be made in one place, resulting in fixes across all contracts simultaneously.

When considering the two systems separately, the Waves Bridge adheres to the administration policies mentioned below, operates according to the description, and does not contain any hidden pitfalls.

The CIP subsystem, in addition to administration policies, includes operations of two entities of privileged users, Witnesses and Signers. The functionality considered at the level of the CIP is included in the audit, but the actual operation and administration of these privileged users are not part of this audit, although the overall protocol's functioning directly depends on them carrying out their functions. For example, incorrect operation of both Witnesses and Signers can allow for the confirmation of non-existent transfers in both directions at the protocol level. So maybe these users should be in a some way decentralized contracts too.

Here and throughout, it is assumed that privileged users, Witnesses and Signers, behave honestly for the most part, which is why they cannot achieve the required quorum through collusion, similar to the administration policy outlined below. Otherwise, we would also need to consider the change of any contract by administrators in case of collusion.

But it is worth noting that the reward and slashing mechanisms are reduced in the current contracts, which does not sufficiently punish "bad" privileged users.

Additionally, due to the logical gap between the CIP and Bridge systems, privileged users known as Signers are capable of executing any transactions at the Bridge level through off-network collusion. It is recommended to improve the Bridge level with an additional mechanism of preliminary registration of future approved actions to block the possiblity of instant withdrawals of all funds.

The team has received vulnerability reports and recommendations for protocol improvements.

As of now and our assumptions, no critical errors have been identified in the RIDE contract code, and fixing the remaining errors may improve the protocol.

## Administration policy

The administration of each group of contracts is done uniformly. Contracts are connected to a common `multisig.ride` contract that provides functionality for transaction approval upon reaching a quorum by administrators. This multisig contract is also connected to itself. This means that all contracts are initially connected to the common system, followed by initialization, which is then approved by administrators to achieve the quorum. This ensures that all contracts belong to the same administration system and prevents any changes to the entire system from being made outside of the quorum.

This can be represented as policy properties:
- All contracts are connected to a multisig contract: The policy states that all contracts are initially connected to a common multisig contract. In a smart-contract system, this means that each individual contract would have a reference or link to the multisig contract, which serves as a central point of control.
- Multisig contract provides functionality for transaction approval upon reaching a quorum: The multisig contract is designed to have functionality that allows transactions to be approved or executed only when a certain number of administrators, known as a quorum, agree to it. This means that a predefined number of administrators need to provide their approval for a transaction to be valid.
- All contracts are initialized and approved by administrators: Before a contract can be fully functional within the system, it needs to be initialized and approved by the administrators. This likely involves a set of predefined steps or checks that need to be completed before the contract is considered valid. Once the administrators provide their approval, the contract is considered to have achieved the quorum and can operate within the system.
- All contracts belong to the same administration system: The policy ensures that all contracts within the system are connected to the same administration system, which is governed by the multisig contract and the quorum of administrators. This means that any changes or updates to the system can only be made through the multisig contract and with the approval of the required number of administrators.
- Prevents changes outside of the quorum: The policy also aims to prevent any changes to the system from being made without the approval of the quorum of administrators. This means that any modifications or updates to the system, including initializing new contracts, need to go through the multisig contract and obtain the necessary approvals from the administrators to maintain the integrity and consistency of the administration process.

In summary, the smart-contract administration policy outlined in the provided statement ensures that all contracts are connected to a multisig contract, initialized and approved by administrators, and operate within the same administration system. It also requires approvals from a quorum of administrators for transactions to be executed, preventing unauthorized changes to the system.


## Severity levels

It's important to note that severity levels may vary depending on the specific context and system being assessed, and different organizations may have their own definitions or classifications for severity levels. It's crucial to thoroughly understand and follow the severity levels specified in the specific technical report or security assessment being referenced.

- TRIVIAL: Refers to findings that have minimal impact on the system or application's security and are of negligible concern. These findings are typically inconsequential and do not pose any significant risk to the overall security of the system.

- LOW: Refers to findings that have minor impact on the system or application's security and may not require immediate action. These findings are typically of minor concern and do not pose a significant risk to the overall security of the system.

- MEDIUM: Refers to findings that have a moderate impact on the system or application's security and may require attention in the near future. These findings may have some potential risk associated with them, but they are not critical or urgent.

- HIGH: Refers to findings that have a significant impact on the system or application's security and may have indirect or not publicly available security impact. These findings pose a substantial risk to the overall security of the system and need to be addressed urgently to mitigate any potential security breaches.

- CRITICAL: Refers to findings that have a severe impact on the system or application's security and can be directly exploited by anyone. These findings pose a critical risk to the overall security of the system and may result in severe consequences if not addressed promptly.

## Vulnerability summary

Github issues:
- https://github.com/crypto-pepe/cip-waves-contracts/issues/1
- https://github.com/crypto-pepe/cip-waves-contracts/issues/2
- https://github.com/crypto-pepe/cip-waves-contracts/issues/3
- https://github.com/crypto-pepe/bridige-waves-contracts/issues/1


| Severity   | Count | Fixed |
|------------|-------|-------|
| TRIVIAL    | 0     | 0     |
| LOW        | 4     | 1     |
| MEDIUM     | 2     | 1     |
| HIGH       | 3     | 2     |
| CRITICAL   | 1     | 1     |

- Total found: 10
- Total fixed: 5

## Vulnerabilities appendix

### Reinitialization
 - Description: Whether a smart contract's constructor can be called multiple times, allowing an attacker to reset the contract's state or parameters, potentially leading to unexpected behavior or loss of funds.
 - Severity: CRITICAL
 - Result: Not found

### Unauthorized permissions
 - Description: Whether an unauthorized user can gain administrative or safeguard permissions in a smart contract, potentially allowing them to manipulate contract parameters, modify critical state variables, or execute privileged functions without proper authorization.
 - Severity: CRITICAL
 - Result: Not found

### Overflows & underflows
 - Description: Whether a smart contract to be susceptible to general overflow or underflow vulnerabilities, where arithmetic operations on contract variables may result in unexpected behavior or errors due to integer overflow or underflow.
 - Severity: LOW
 - Result: Fixed

### Reentrancy
 - Description: Whether a code of a smart contract can call back into the same contract and modify its state, potentially leading to unexpected changes in contract behavior or manipulation of critical state variables.
 - Severity: CRITICAL
 - Result: Not found

### Unauthorized fund transfer
 - Description: Whether a smart contract returning funds to an arbitrary or unverified address, potentially allowing an attacker to redirect funds to an unauthorized address.
 - Severity: CRITICAL
 - Result: Not found

### Token lockout
 - Description: Whether a smart contract locking tokens indefinitely, preventing their transfer or use beyond a certain point in time, which may result in loss of access or utility for token holders.
 - Severity: MEDIUM
 - Result: Found
 - Status: Only for privileged users known as Signers. Will be resolved in next versions.

### Unauthorized contract termination
 - Description: Whether a smart contract being susceptible to being killed or terminated by an arbitrary party, resulting in the contract being rendered inactive or non-functional.
 - Severity: HIGH
 - Result: Not found

### Unexpected halt
 - Description: Whether a smart contract is vulnerable to a Denial-of-Service (DoS) attack due to unexpected or erroneous usage, which can result in contract halting.
 - Severity: MEDIUM
 - Result: Not found

### External call dependency
 - Description: Whether a smart contract relying on the return value of an external call, which can be manipulated by an attacker, potentially leading to unexpected or malicious behavior.
 - Severity: MEDIUM
 - Result: Fixed

### Deterministic randomness
 - Description: Whether a smart contract containing a predictable randomness source, which can be exploited by an attacker to predict the outcome of randomness-dependent operations.
 - Severity: MEDIUM
 - Result: Not found

### Conflicting state updates
 - Description: Whether a smart contract function's data writes having overlaps instead of aggregations, resulting in potential conflicts or inconsistent state updates.
 - Severity: MEDIUM
 - Result: Not found

### Complexity overflow
 - Description: Whether smart contracts having scenarios that are not possible under certain conditions due to complexity limits, potentially leading to unexpected behavior or limitations in contract functionality.
 - Severity: MEDIUM
 - Result: Not found

### Complexity optimization
 - Description: Whether a smart contract have an excess complexity usage, which can lead to scalability issues, or limitations in contract functionality in future.
 - Severity: LOW
 - Result: Fixed (partly)
 - Status: Not every recommendation currently applied but there is no lack of RIDE complexity in general.

### Incorrect payment
 - Description: Whether a smart contract can accept a different payment asset than the one required, potentially resulting in incorrect value assessment or improper handling of payments.
 - Severity: HIGH
 - Result: Not found

### Zero payment
 - Description: Whether invoking a smart contract with an attached amount of 0 for an asset, rendering the invocation impossible or leading to unexpected behavior.
 - Severity: MEDIUM
 - Result: Not found

### Input validation
 - Description: Whether user inputs are not properly validated, resulting in potential unexpected behavior or exploitation of the system.
 - Severity: CRITICAL
 - Result: Fixed

### Serialization
 - Description: Whether a smart contract uses serialization data that can be encoded non-deterministically, resulting in the movement of fields inside the serialized data.
 - Severity: HIGH
 - Result: Fixed

### Logic flaws
 - Description: Whether issues with the logic or flow of the contract may lead to unintended behavior or exploitation.
 - Severity: CRITICAL
 - Result: Not found

### Front-running
 - Description: Whether smart contracts being susceptible to front-running attacks, where an attacker can manipulate the order of transactions or exploit the order of execution in the Waves blockchain to gain an unfair advantage or manipulate outcomes.
 - Severity: CRITICAL
 - Result: Not found

### Timestamp manipulation
 - Description: Whether timestamps or block numbers are used for time-based logic and can be manipulated, leading to potential exploits.
 - Severity: MEDIUM
 - Result: Not found

### Insider attack
 - Description: Whether individuals or entities involved in the development, deployment, or maintenance of the contract may exploit their access and permissions.
 - Severity: CRITICAL
 - Result: Not found

### Upgradeability
 - Description: Whether smart contracts are immutable making it challenging to fix discovered issues or vulnerabilities.
 - Severity: HIGH
 - Result: Not found

### Litepaper compliance
 - Description: Whether smart contracts accurately implement the full semantics of their corresponding litepaper, enabling them to realize the complete scenarios as described.
 - Severity: HIGH
 - Result: Found
 - Status:
   - EVM-like signatures are not verified at the blockchain level. Will be resolved in next versions.
   - Threshold signatures are verified only at the final step. Needs additional functionality at the blockchain level.