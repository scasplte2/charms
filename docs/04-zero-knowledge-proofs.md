# Zero-Knowledge Proofs in Charms

This document explores how zero-knowledge proofs are used in the Charms protocol to enable secure and private programmability on Bitcoin.

## Introduction to Zero-Knowledge Proofs

Zero-knowledge proofs (ZKPs) are cryptographic methods that allow one party (the prover) to prove to another party (the verifier) that a statement is true, without revealing any information beyond the validity of the statement itself.

In the context of Charms, ZKPs are used to prove that a spell (a transformation of tokens, NFTs, or app state) was executed correctly according to the rules defined by its apps, without revealing the details of the execution.

## SP1 zkVM

Charms uses the SP1 zkVM (Zero-Knowledge Virtual Machine) to execute RISC-V programs and generate zero-knowledge proofs of their correct execution. SP1 is:

- A zero-knowledge virtual machine that proves the correct execution of programs compiled for the RISC-V architecture
- Capable of running programs written in Rust, C++, C, or any language for which a RISC-V compiler backend exists
- Optimized for performance and efficiency

## Proof Generation Process

The proof generation process in Charms involves several steps:

### 1. App Execution

First, the app is executed in the SP1 zkVM:

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

This execution generates a trace of the program's execution, which is used to create a zero-knowledge proof.

### 2. Proof Generation

Next, a zero-knowledge proof is generated using the SP1 zkVM:

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

This process involves several steps:

1. **Core Proof**: A proof of the RISC-V execution
2. **Compression**: Reducing the size of the proof
3. **Shrinking**: Further optimization of the proof
4. **Wrapping**: Converting the proof to a BN254 elliptic curve format
5. **Groth16 Conversion**: Final conversion to a Groth16 proof

### 3. Proof Serialization

The proof is then serialized along with the spell:

```rust
let spell_data = util::write(&(&norm_spell, &proof))?;
```

### 4. Bitcoin Integration

The serialized spell and proof are embedded in a Bitcoin transaction using Tapscript:

```rust
pub fn data_script(public_key: XOnlyPublicKey, data: &[u8]) -> ScriptBuf {
    let builder = ScriptBuf::builder();
    push_envelope(builder, data)
        .push_slice(public_key.serialize())
        .push_opcode(OP_CHECKSIG)
        .into_script()
}

fn push_envelope(builder: Builder, data: &[u8]) -> Builder {
    let mut builder = builder
        .push_opcode(OP_FALSE)
        .push_opcode(OP_IF)
        .push_slice(b"spell");
    for chunk in data.chunks(MAX_SCRIPT_ELEMENT_SIZE) {
        builder = builder.push_slice::<&PushBytes>(chunk.try_into().unwrap());
    }
    builder.push_opcode(OP_ENDIF)
}
```

## Proof Verification

When a spell is extracted from a Bitcoin transaction, its proof is verified to ensure correctness:

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

This verification process ensures that the spell was executed correctly according to the rules defined by its apps.

## Benefits of Zero-Knowledge Proofs in Charms

Zero-knowledge proofs provide several benefits in the Charms protocol:

### 1. Privacy

ZKPs allow for private computation while still ensuring correctness. This means that:

- The details of the computation are not revealed
- Only the fact that the computation was performed correctly is verified
- Sensitive information can be kept private

### 2. Scalability

ZKPs allow for complex computations to be performed off-chain, with only a compact proof being included in the Bitcoin transaction. This improves scalability by:

- Reducing the amount of data that needs to be stored on the blockchain
- Allowing for complex computations that would be impractical to perform on-chain
- Enabling efficient verification of complex computations

### 3. Programmability

ZKPs enable complex programmability on Bitcoin without requiring changes to the Bitcoin protocol. This allows for:

- Smart contract-like functionality on Bitcoin
- Complex application logic that can be verified by anyone
- Composable applications that can interact with each other

### 4. Security

ZKPs ensure that computations are performed correctly, providing strong security guarantees:

- Computations cannot be tampered with
- Results cannot be falsified
- The integrity of the system is maintained

## Groth16 Proof System

Charms uses the Groth16 zero-knowledge proof system, which is:

- Highly efficient for verification
- Compact in terms of proof size
- Well-suited for blockchain applications

The Groth16 proof system requires a trusted setup, but once this setup is performed, it provides strong security guarantees and efficient verification.

## Future Directions

The Charms protocol could potentially support other zero-knowledge proof systems in the future, such as:

- Plonk: A universal and updateable trusted setup
- Halo2: A recursive proof system without a trusted setup
- Nova: A proof system optimized for recursive proofs

These systems could provide different trade-offs in terms of setup requirements, proof size, and verification efficiency.

In the next document, we'll explore how Charms can be extended with a topos-based state channel framework.
