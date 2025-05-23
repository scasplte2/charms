# Charms Research Strategy: Cross-Chain State Channels via Topos Framework

## Research Goals

1. Create updated versions of IOTX diagrams using Graphviz/DOT for better visualization
   - Preserve DAG construction nature
   - Only propagate arrows left to right or top to bottom
   
2. Produce documentation that:
   - Maps how Charms can be defined using these updated diagrams
   - Shows how Charms can be extended to support other UTXO systems (Ergo, Nervos, Alephium)
   - Develops a topos-inspired state-channel framework on top of Charms

3. Focus on practical and cross-chain interoperability with:
   - Primary targets: BTC, ETH, and Cardano
   - Secondary targets: Ergo, Nervos, Alephium

## Research Strategy

### 1. Updated IOTX Visualization and Formalization

First, we'll create updated Graphviz/DOT diagrams preserving the DAG construction with consistent left-to-right or top-to-bottom flow. These will include:

- Transaction flow diagrams showing input/output relationships
- Lock/signature/evidence verification flows
- State transition diagrams for cross-chain interactions
- Formal representation of the proposition/proof verification model

### 2. Formal Description of Charms via Diagrams

We'll use these updated diagrams to:
- Map Charms' core concepts (Apps, Transactions, Spells, Proofs) to graph structures
- Demonstrate how Charms' zkVM approach relates to the abstract cell model
- Show how the existing Charms architecture can be extended to support the state channel interface (stateSchema, txSchema, combineFns, validateFns)

### 3. Cross-Chain Extension Framework

Our analysis will focus on practical cross-chain interoperability in this order:

**Primary Targets:**
- Bitcoin: Leveraging Charms' native UTXO support
- Ethereum: Mapping account-based to abstract cell model
- Cardano: Extending eUTXO concepts already aligned with Charms

**Secondary Targets:**
- Ergo: Examining data-inputs and multi-stage protocols
- Nervos: Analyzing CKB cell model compatibility
- Alephium: Exploring UTXO with flow-based scripting

### 4. Topos-Inspired State Channel Framework

This framework will:
- Define the lifecycle operations (launch, join, leave, commit, rollup, destroy)
- Model state channels as hash-linked event chains with finite state machine transitions
- Structure cross-chain communication through parallel composable state machines
- Provide practical implementation patterns for BTC/ETH/Cardano interoperability

### 5. Key Research Questions

1. How can we generalize Charms' recursive zkVM proof system to verify state transitions across heterogeneous chain models?

2. What patterns are needed to manage the varying complexities of event chains and sub-event chains?

3. How can we efficiently map the "client-side validation" model of Charms to chains with different validation paradigms?

4. What security properties must be preserved when operating across chains with different security and finality guarantees?

5. How can the topos concept of "internal logic" be best applied to create verifiable cross-chain protocols?

## Research Materials

Key documents and resources we've examined so far:

1. Charms whitepaper - Programmable assets on Bitcoin and beyond
2. Charms documentation - Focus on sections 5 (Topos Framework) and 6 (Cross-Chain State Channels)
3. IOTX model diagrams - DAG-based transaction model with proposition/proof pairs
4. Research papers on UTXO models - High Level Design Patterns in Extended UTXO Systems and Unlocking The Potential Of The UTXO Model
