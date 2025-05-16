# Charms Architecture

This document provides an overview of the Charms architecture, explaining how the different components work together to enable programmable tokens and NFTs on Bitcoin.

## System Overview

The Charms system consists of several key components:

1. **Charms SDK**: Provides the framework for developing apps
2. **Charms Client**: Handles interaction with the Bitcoin blockchain
3. **Charms Data**: Defines the data structures used in the system
4. **Charms Spell Checker**: Verifies the correctness of spells
5. **SP1 zkVM**: Executes RISC-V programs and generates zero-knowledge proofs

These components work together to enable the creation, execution, and verification of spells on the Bitcoin blockchain.

## Component Architecture

### 1. Charms SDK

The Charms SDK (`charms-sdk`) provides the framework for developing apps. It includes:

- A macro system for defining app entrypoints
- Integration with the SP1 zkVM
- Utilities for handling data serialization and deserialization

The SDK allows developers to write apps in Rust (or any language that compiles to RISC-V) and have them executed in the SP1 zkVM.

### 2. Charms Client

The Charms Client (`charms-client`) handles interaction with the Bitcoin blockchain. It:

- Extracts and verifies spells from Bitcoin transactions
- Normalizes spells for processing
- Provides utilities for working with Bitcoin transactions

### 3. Charms Data

The Charms Data (`charms-data`) component defines the data structures used in the system:

- `App`: Represents an application with a tag, identity, and verification key
- `Transaction`: Represents a transaction involving Charms
- `Charms`: Collections of tokens, NFTs, or app state
- `Data`: Generic data structure for storing app-specific data

### 4. Charms Spell Checker

The Charms Spell Checker (`charms-spell-checker`) verifies the correctness of spells. It:

- Checks if a spell is well-formed
- Verifies that the spell satisfies the constraints of its apps
- Ensures that the spell's inputs and outputs are valid

### 5. SP1 zkVM

The SP1 zkVM is a zero-knowledge virtual machine that:

- Executes RISC-V programs
- Generates zero-knowledge proofs of correct execution
- Supports various proof systems, including Groth16

## Data Flow

The data flow in the Charms system follows these steps:

1. **App Development**:
   - Developers create apps using the Charms SDK
   - Apps are compiled to RISC-V binaries

2. **Spell Creation**:
   - Users create spells that reference apps
   - Spells define transformations of Charms

3. **Proof Generation**:
   - The system executes the app in the SP1 zkVM
   - The zkVM generates a zero-knowledge proof of correct execution

4. **Bitcoin Integration**:
   - The spell and its proof are embedded in a Bitcoin transaction
   - The transaction is broadcast to the Bitcoin network

5. **Verification**:
   - Other participants extract the spell from the transaction
   - They verify the zero-knowledge proof to ensure correctness

## Transaction Structure

Charms uses a two-transaction structure for embedding spells in Bitcoin:

1. **Commit Transaction**:
   - Creates a Taproot output containing the spell
   - Uses a Tapscript to encode the spell data

2. **Spell Transaction**:
   - Spends the commit transaction
   - Executes the spell and transforms Charms

This structure allows for complex programmability while maintaining compatibility with the Bitcoin protocol.

## Proof System

Charms uses the Groth16 zero-knowledge proof system for verifying the correctness of spells. The proof generation process involves:

1. **Core Proof**: A proof of the RISC-V execution
2. **Compression**: Reducing the size of the proof
3. **Shrinking**: Further optimization of the proof
4. **Wrapping**: Converting the proof to a BN254 elliptic curve format
5. **Groth16 Conversion**: Final conversion to a Groth16 proof

This multi-step process ensures that proofs are compact and efficient to verify.

## Security Model

The Charms security model relies on several key properties:

- **Bitcoin Security**: Leverages Bitcoin's security for transaction finality
- **Zero-Knowledge Proofs**: Ensures correctness of spell execution
- **Taproot Privacy**: Uses Taproot to enhance privacy
- **App Verification**: Verifies that apps satisfy their constraints

By combining these properties, Charms provides a secure and private platform for programmable tokens and NFTs on Bitcoin.

In the next document, we'll explore spells and apps in more detail.
