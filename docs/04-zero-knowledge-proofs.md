# Zero-Knowledge Proofs in Charms

This document explores how zero-knowledge proofs enable secure and private programmability in the Charms protocol on Bitcoin.

## Introduction to Zero-Knowledge Proofs

Zero-knowledge proofs (ZKPs) are cryptographic methods that allow one party (the prover) to prove to another party (the verifier) that a statement is true, without revealing any information beyond the validity of the statement itself.

In Charms, ZKPs serve a crucial role: they prove that a spell (a transformation of tokens, NFTs, or app state) was executed correctly according to the rules defined by its apps, without revealing the details of the execution or requiring verifiers to re-execute the computation.

## The SP1 zkVM

Charms uses the SP1 zkVM (Zero-Knowledge Virtual Machine) to execute RISC-V programs and generate zero-knowledge proofs of their correct execution. SP1 provides:

- **RISC-V Compatibility**: Runs programs compiled for the widely-supported RISC-V architecture
- **Language Flexibility**: Supports programs written in Rust, C++, C, or any language with a RISC-V compiler
- **Performance Optimization**: Highly optimized for proof generation speed and verification efficiency
- **Groth16 Output**: Produces compact Groth16 proofs for efficient on-chain verification

## Recursive Proof System Architecture

The heart of Charms' efficiency lies in its recursive proof system, which eliminates the need to traverse transaction history for verification.

![Recursive Proof System](diagrams/img/recursive-proof-system.svg)
*Figure 1: How recursive proofs enable O(1) verification without transaction graph traversal*

### Key Innovation: No Transaction Graph Traversal

Traditional blockchain verification requires checking the entire history of an asset to ensure its legitimacy. Charms replaces this with a single proof verification that recursively attests to all prerequisites.

**Traditional Approach:**
- Traverse entire transaction graph
- Verify each transaction individually
- O(n) verification time
- High bandwidth usage

**Charms Recursive Approach:**
- Single proof verification
- O(1) verification time
- Constant bandwidth
- Previous proofs embedded recursively

## Proof Generation Process

The proof generation in Charms involves a sophisticated multi-stage pipeline:

### Stage 1: App Contract Execution

First, app contracts are executed in the SP1 zkVM:

```rust
pub fn run(
    &self,
    app_binary: &[u8],
    app: &App,
    tx: &Transaction,
    x: &Data,
    w: &Data,
) -> anyhow::Result<()> {
    let (_pk, vk) = self.sp1_client.get().setup(app_binary);
    ensure!(app.vk == B32(app_vk(vk)), "app.vk mismatch");

    let mut app_stdin = SP1Stdin::new();
    app_stdin.write_vec(util::write(&(app, tx, x, w))?);
    let (committed_values, _report) = self.sp1_client.get().execute(app_binary, &app_stdin)?;
    let com: (App, Transaction, Data) = util::read(committed_values.to_vec().as_slice())?;
    ensure!(
        (&com.0, &com.1, &com.2) == (app, tx, x),
        "committed data mismatch"
    );
    Ok(())
}
```

This execution generates a trace of the program's execution, which forms the basis for proof generation.

### Stage 2: Multi-Stage Proof Pipeline

The SP1 zkVM then generates proofs through a carefully orchestrated pipeline:

1. **Core Proof**: Initial RISC-V execution trace proof
2. **Compression**: Reduces proof size for efficiency
3. **Shrinking**: Further optimization for minimal bandwidth
4. **Wrapping**: Converts to BN254 elliptic curve format
5. **Groth16 Conversion**: Final compact proof format

```rust
fn prove(
    &self,
    pk: &SP1ProvingKey,
    stdin: &SP1Stdin,
    kind: SP1ProofMode,
) -> Result<SP1ProofWithPublicValues> {
    let cuda_prover = &*self.cuda_prover.lock().unwrap();

    cuda_prover.ready()?;
    cuda_prover.setup(&pk.elf)?;

    // Generate the core proof.
    let proof = cuda_prover.prove_core(stdin)?;
    if kind == SP1ProofMode::Core {
        return Ok(SP1ProofWithPublicValues {
            proof: SP1Proof::Core(proof.proof.0),
            public_values: proof.public_values,
            sp1_version: self.version().to_string(),
            tee_proof: None,
        });
    }

    // Generate the compressed proof.
    let deferred_proofs = stdin
        .proofs
        .iter()
        .map(|(reduce_proof, _)| reduce_proof.clone())
        .collect();
    let public_values = proof.public_values.clone();
    let reduce_proof = cuda_prover.compress(&pk.vk, proof, deferred_proofs)?;
    if kind == SP1ProofMode::Compressed {
        return Ok(SP1ProofWithPublicValues {
            proof: SP1Proof::Compressed(Box::new(reduce_proof)),
            public_values,
            sp1_version: self.version().to_string(),
            tee_proof: None,
        });
    }

    // Generate the shrink proof.
    let compress_proof = cuda_prover.shrink(reduce_proof)?;

    // Genenerate the wrap proof.
    let outer_proof = cuda_prover.wrap_bn254(compress_proof)?;

    if kind == SP1ProofMode::Groth16 {
        let groth16_bn254_artifacts = groth16_circuit_artifacts_dir();

        let proof = self
            .cpu_prover
            .wrap_groth16_bn254(outer_proof, &groth16_bn254_artifacts);
        return Ok(SP1ProofWithPublicValues {
            proof: SP1Proof::Groth16(proof),
            public_values,
            sp1_version: self.version().to_string(),
            tee_proof: None,
        });
    }

    unimplemented!("Unsupported proof mode: {:?}", kind);
}
```

### Stage 3: Spell Proof Generation

The Charms Spell Checker program runs in the zkVM to generate spell proofs:

**Public Inputs:**
- Spell VK (recursive spell verification key)
- NormalizedSpell (the spell being checked)

**Private Inputs:**
- App contract proofs (proving app constraints are satisfied)
- Prerequisite transactions (transactions that created input UTXOs)

The spell checker verifies:
1. **Spell Well-Formed**: Version supported, apps listed correctly, no invalid indexes
2. **App Contracts Satisfied**: All app predicates return true with valid inputs
3. **Prerequisites Valid**: Input UTXOs are legitimate with valid spell proofs

## Proof Verification Process

When a spell is extracted from a Bitcoin transaction, its proof is verified efficiently:

```rust
pub fn extract_and_verify_spell(
    tx: &bitcoin::Transaction,
    spell_vk: &str,
) -> anyhow::Result<NormalizedSpell> {
    let Some((spell_tx_in, tx_ins)) = tx.input.split_last() else {
        bail!("transaction does not have inputs")
    };

    let (spell, proof) = parse_spell_and_proof(spell_tx_in)?;

    ensure!(
        &spell.tx.outs.len() <= &tx.output.len(),
        "spell tx outs mismatch"
    );
    ensure!(
        &spell.tx.ins.is_none(),
        "spell must inherit inputs from the enchanted tx"
    );

    let spell = spell_with_ins(spell, tx_ins);

    let (spell_vk, groth16_vk) = vks(spell.version, spell_vk)?;

    Groth16Verifier::verify(
        &proof,
        to_sp1_pv(spell.version, &(spell_vk, &spell)).as_slice(),
        spell_vk,
        groth16_vk,
    )
    .map_err(|e| anyhow!("could not verify spell proof: {}", e))?;

    Ok(spell)
}
```

This verification process ensures that the spell was executed correctly according to all app constraints.

## Special Case Optimizations

Charms optimizes for common operations that don't require full zkVM execution:

### Simple Token Transfers (tag='t')
- **Condition**: Total input amount equals total output amount
- **Optimization**: No proof generation needed
- **Validation**: Simple arithmetic check

### Simple NFT Transfers (tag='n')
- **Condition**: NFT data remains unchanged during transfer
- **Optimization**: No proof generation needed
- **Validation**: Simple data comparison

### Custom Apps
- **Requirement**: Full zkVM execution with proof generation
- **Use Case**: Any logic beyond simple transfers
- **Flexibility**: Arbitrary programmable behavior

## Benefits of Zero-Knowledge Proofs in Charms

### 1. Privacy

ZKPs enable private computation while maintaining correctness:

- **Execution Details Hidden**: The specific computation steps remain private
- **Input Privacy**: Private inputs (witness data) are never revealed
- **Selective Disclosure**: Only necessary information is made public

### 2. Scalability

ZKPs dramatically improve scalability by eliminating transaction graph traversal:

- **Constant Verification Time**: O(1) regardless of transaction history depth
- **Minimal Bandwidth**: Only spell and proof data required
- **Efficient Storage**: No need to store entire transaction histories

### 3. Programmability

ZKPs enable complex programmability on Bitcoin without protocol changes:

- **Smart Contract Logic**: Arbitrary business logic can be implemented
- **Composable Applications**: Multiple apps can interact within transactions
- **Cross-Chain Compatibility**: Same apps work on different blockchains

### 4. Security

ZKPs provide strong security guarantees:

- **Computational Integrity**: Proofs ensure computations were performed correctly
- **Tamper Resistance**: Proofs cannot be forged or modified
- **Non-Interactive**: Verification doesn't require interaction with the prover

## Performance Characteristics

### Client-Side Verification

Charms is designed to be efficient enough for web and mobile applications:

- **Web Browser Compatible**: Groth16 verification runs efficiently in browsers
- **Mobile Friendly**: Minimal computational requirements for verification
- **Offline Capable**: Verification doesn't require network access once data is available

### Proof Generation

While proof generation is more computationally intensive, it's optimized for practical use:

- **Hardware Acceleration**: Supports GPU acceleration for faster proof generation
- **Parallel Processing**: Multiple app proofs can be generated concurrently
- **Incremental Updates**: Only changed apps need new proofs
- **Caching**: Generated proofs can be cached and reused

### Network Efficiency

The recursive nature of proofs provides excellent network efficiency:

- **Constant Proof Size**: Proof size doesn't grow with transaction depth
- **Bandwidth Optimization**: Only new spell data needs to be transmitted
- **CDN Friendly**: Proofs can be cached and distributed efficiently

## The Groth16 Proof System

Charms uses Groth16 as its final proof format due to several advantages:

### Advantages of Groth16

- **Compact Proofs**: Very small proof size (around 200 bytes)
- **Fast Verification**: Constant-time verification regardless of circuit size
- **Mature Technology**: Well-tested and widely adopted
- **Hardware Support**: Efficient implementation on various platforms

### Trusted Setup

Groth16 requires a trusted setup, but this is handled transparently:

- **Per-Circuit Setup**: Each app circuit has its own trusted setup
- **Open Source Process**: Setup ceremonies are conducted openly
- **Verification**: Setup parameters can be independently verified
- **Future Migration**: Protocol designed to support other proof systems

## Future Directions

The Charms protocol is designed to adapt to advances in zero-knowledge technology:

### Alternative Proof Systems

Future versions could support:

- **PLONK**: Universal and updateable trusted setup
- **Halo2**: Recursive proof system without trusted setup
- **Nova**: Optimized for recursive proofs
- **STARKs**: Post-quantum security without trusted setup

### Performance Improvements

Ongoing optimizations include:

- **Circuit Optimization**: More efficient constraint systems
- **Proof Aggregation**: Combining multiple proofs for better efficiency
- **Hardware Acceleration**: Specialized hardware for proof generation
- **Algorithm Advances**: Leveraging new research in ZK technology

### Integration Enhancements

Future developments may include:

- **Light Client Integration**: Better support for mobile and web clients
- **Cross-Chain Verification**: More efficient cross-chain proof verification
- **Privacy Features**: Enhanced privacy through advanced ZK techniques

The next document explores how Charms can be extended with advanced frameworks for state channels and cross-chain operations.
