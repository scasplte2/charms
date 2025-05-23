# Charms Documentation Index

This documentation provides a comprehensive overview of the Charms protocol, a system for programmable tokens and NFTs on Bitcoin using zero-knowledge proofs.

## Core Documentation

These documents cover the production-ready features of Charms that developers can use today:

### 1. [Introduction to Charms](01-introduction-to-charms.md)
   - What are Charms and how they work
   - Key concepts: Apps, Spells, and UTXOs
   - Cross-chain capabilities overview
   - Benefits and practical use cases
   - **Diagrams**: Asset construction flow, UTXO lifecycle, cross-chain examples

### 2. [Charms Architecture](02-charms-architecture.md)
   - System overview and component architecture
   - App contract execution and data structures
   - Transaction structure and spell processing
   - Recursive proof system architecture
   - Cross-chain (C3T) protocol design
   - Security model and performance characteristics
   - **Diagrams**: Recursive proof system, app execution flow, cross-chain transfer process

### 3. [Spells and Apps in Charms](03-spells-and-apps.md)
   - Understanding spells: structure, normalization, and verification
   - Understanding apps: development, compilation, and execution
   - Interaction between spells and apps
   - Practical examples: tokens, NFTs, cross-chain operations
   - Best practices for development
   - **Diagrams**: App development sequence, app execution flow

### 4. [Zero-Knowledge Proofs in Charms](04-zero-knowledge-proofs.md)
   - Introduction to zero-knowledge proofs in Charms
   - SP1 zkVM and proof generation pipeline
   - Recursive proof system and verification process
   - Special case optimizations for tokens and NFTs
   - Performance characteristics and benefits
   - Groth16 proof system details
   - **Diagrams**: Recursive proof system architecture

## Advanced Research Topics

These documents explore theoretical concepts and future research directions that are not part of the current implementation:

### 5. [Advanced Research: Topos Framework Using Charms](05-topos-framework-using-charms.md)
   - **Research Topic**: Theoretical topos-based state channel frameworks
   - Topos theory applications to computational systems
   - Potential integration with existing Charms protocol
   - Research challenges and future directions

### 6. [Advanced Research: Cross-Chain State Channels Feasibility](06-cross-chain-state-channels.md)
   - **Research Topic**: Advanced cross-chain state channel concepts
   - Current C3T protocol vs. theoretical extensions
   - Abstract cell model and research challenges
   - Integration possibilities with current Charms infrastructure

## Interactive Examples

Explore these sequence diagrams to understand Charms workflows:

- **[App Development Sequence](diagrams/img/app-development-sequence.svg)**: Complete workflow from writing code to deploying a Charms app
- **[Cross-Chain Journey Example](diagrams/img/cross-chain-journey-sequence.svg)**: Assets moving from Bitcoin to Cardano and back, with operations on both chains
- **[Multi-User Trade Example](diagrams/img/multi-user-trade-sequence.svg)**: Atomic cross-chain trade between users on different blockchains

## Technical Diagrams

Detailed technical diagrams illustrating Charms internals:

- **[Asset Construction Flow](diagrams/img/asset-construction-flow.svg)**: From app development to on-chain deployment
- **[UTXO Charm Lifecycle](diagrams/img/utxo-charm-lifecycle.svg)**: Creation, transformation, and composability of charmed UTXOs
- **[Recursive Proof System](diagrams/img/recursive-proof-system.svg)**: How O(1) verification works without transaction graph traversal
- **[App Execution Flow](diagrams/img/app-execution-flow.svg)**: SP1 zkVM execution and validation process
- **[Cross-Chain Transfer (C3T)](diagrams/img/cross-chain-transfer.svg)**: Complete C3T protocol for moving assets between blockchains

## Getting Started

### For Developers
1. Start with the [Introduction](01-introduction-to-charms.md) to understand core concepts
2. Review the [Architecture](02-charms-architecture.md) to understand system design
3. Learn about [Spells and Apps](03-spells-and-apps.md) for practical development
4. Understand [Zero-Knowledge Proofs](04-zero-knowledge-proofs.md) for the security model

### For Researchers
1. Review the core documentation first (documents 1-4)
2. Explore the [Topos Framework](05-topos-framework-using-charms.md) research
3. Investigate [Cross-Chain State Channels](06-cross-chain-state-channels.md) feasibility

### For Cross-Chain Users
1. Understand basic concepts in the [Introduction](01-introduction-to-charms.md)
2. Focus on C3T protocol details in [Architecture](02-charms-architecture.md)
3. See practical examples in the [Cross-Chain Journey](diagrams/img/cross-chain-journey-sequence.svg) and [Multi-User Trade](diagrams/img/multi-user-trade-sequence.svg) sequences

## Documentation Goals

This documentation aims to:

- **Provide Clear Distinction**: Separate production-ready features from research concepts
- **Enable Practical Development**: Give developers concrete guidance for building on Charms
- **Explain Complex Concepts**: Make zero-knowledge proofs and recursive verification accessible
- **Show Real Examples**: Use concrete scenarios like cross-chain asset movement and trading
- **Maintain Consistency**: Avoid repetition while ensuring comprehensive coverage

## About This Documentation

This documentation was created to provide a comprehensive understanding of the Charms protocol. It covers both the current implementation that developers can use today, as well as theoretical research directions that may be explored in the future.

The documentation emphasizes practical usage while providing sufficient technical depth for developers, researchers, and advanced users who wish to understand the protocol thoroughly.

For questions or contributions to this documentation, please refer to the main Charms repository.
