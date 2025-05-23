# Cardano (eUTXO) Protocol Analysis

## Overview

Cardano is a third-generation blockchain platform that pioneered the Extended UTXO (eUTXO) model. This model combines the security and scalability benefits of Bitcoin's UTXO model with the expressiveness needed for smart contracts. Cardano's smart contracts are written in Plutus, a Haskell-based language that ensures formal verification capabilities.

## Key Components

### 1. eUTXO Model

- **Extended UTXO**: A standard UTXO but with additional capabilities:
  - Contains datum (arbitrary data) attached to the output
  - Controlled by validator scripts (smart contracts)
  - Can hold multiple native assets alongside ADA

- **Datum**: Arbitrary data stored in UTXOs
  - Represents state for smart contracts
  - Can be referenced by transactions with CIP-31 (Reference Inputs)

- **Validator Scripts**: Control how UTXOs can be spent
  - Receive datum, redeemer, and context
  - Return boolean (true = valid spend)

### 2. Transaction Structure

- **Inputs**: UTXOs being spent
- **Reference Inputs**: UTXOs referenced but not spent (CIP-31)
- **Outputs**: New UTXOs being created
- **Mint/Burn**: Native asset creation/destruction
- **Certificates**: Staking operations
- **Metadata**: Arbitrary transaction metadata
- **Scripts**: Collection of scripts used in the transaction

### 3. Script Types

- **Spending Scripts**: For spending UTXOs
- **Minting Scripts**: For creating/destroying native assets
- **Staking Scripts**: For delegation/staking operations
- **Governance Scripts**: For protocol governance

### 4. Script Context

Scripts have access to comprehensive transaction context:
- All inputs and outputs in the transaction
- Transaction metadata
- Validity time range
- Signatures provided

## eUTXO Contract Execution Model

The key characteristic of eUTXO is its deterministic validation model:

1. **Validation-Time Execution**: Scripts run at validation time, not during "calls"
2. **Local State**: Each UTXO contains its own state (datum)
3. **Concurrency Model**: No shared state, enabling high concurrency 
4. **Execution Fee Model**: Fee based on script size and execution steps

## Design Patterns in eUTXO

Several important design patterns have emerged for eUTXO smart contracts:

1. **Linear Protocols**: Sequential state transitions
2. **Branching Protocols**: State transitions with multiple paths
3. **Recursive Protocols**: State transitions that can loop back
4. **Parallelized Protocols**: Multiple state machines operating concurrently
5. **Outsourced Computation**: Delegating computation to auxiliary UTXOs

## Double Satisfaction Problem

A challenge specific to eUTXO systems is the "double satisfaction problem":

- Transactions can potentially satisfy spending conditions in unintended ways
- Solution: Validate entire transaction, not just individual inputs
- Necessary to include unique identifiers or constraints in validator logic

## Relation to Charms

Cardano's eUTXO model is closely related to Charms in several ways:

1. **Extended Data Model**: Both allow arbitrary data in UTXOs
2. **Multi-Asset Support**: Both support multiple assets per UTXO
3. **Reference Inputs**: Both support reading UTXOs without spending
4. **Validation Model**: Both validate at the transaction level

Key differences:

1. **Proof System**: Charms uses recursive zkVM proofs instead of on-chain script execution
2. **Language**: Cardano uses Plutus (Haskell-based), while Charms uses Rust compiled to RISC-V
3. **Client Validation**: Charms emphasizes client-side validation 

## Relevance to Our Research

Cardano's eUTXO model provides several insights for our state channel framework:

1. **State Machine Patterns**: Can be adapted for state channel transitions
2. **Reference Inputs**: Enable composable off-chain state channels
3. **Datum Design**: Provides patterns for state channel state representation
4. **Double Satisfaction Prevention**: Important for secure settlement

Cardano is a natural integration target for Charms due to their shared UTXO heritage, making it an excellent candidate for cross-chain state channels.
