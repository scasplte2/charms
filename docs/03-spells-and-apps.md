# Spells and Apps in Charms

This document explores the concepts of spells and apps in the Charms protocol, explaining how they work together to enable programmable tokens and NFTs on Bitcoin.

## Spells

### What is a Spell?

A spell is a special message embedded in a Bitcoin transaction that creates or transforms Charms. It defines how tokens, NFTs, or arbitrary app state are created, transferred, or modified.

### Spell Structure

A spell consists of several key components:

1. **Version**: The version of the Charms protocol
2. **Apps**: The applications referenced by the spell
3. **Public Inputs**: Public inputs to the apps
4. **Private Inputs**: Private inputs to the apps (not included in the transaction)
5. **Inputs**: UTXOs being spent
6. **References**: UTXOs being referenced but not spent
7. **Outputs**: New UTXOs being created

Here's an example of a spell structure from the codebase:

```rust
pub struct Spell {
    /// Version of the protocol.
    pub version: u32,

    /// Apps used in the spell. Map of `$KEY: App`.
    /// Keys are arbitrary strings. They just need to be unique (inside the spell).
    pub apps: BTreeMap<String, App>,

    /// Public inputs to the apps for this spell. Map of `$KEY: Data`.
    #[serde(skip_serializing_if = "Option::is_none")]
    pub public_inputs: Option<BTreeMap<String, Data>>,

    /// Private inputs to the apps for this spell. Map of `$KEY: Data`.
    #[serde(skip_serializing_if = "Option::is_none")]
    pub private_inputs: Option<BTreeMap<String, Data>>,

    /// Transaction inputs.
    pub ins: Vec<Input>,
    /// Reference inputs.
    #[serde(skip_serializing_if = "Option::is_none")]
    pub refs: Option<Vec<Input>>,
    /// Transaction outputs.
    pub outs: Vec<Output>,
}
```

### Normalized Spells

For processing and verification, spells are normalized into a standardized format called `NormalizedSpell`. This format:

- Uses indices instead of keys for apps
- Ensures consistent ordering of inputs and outputs
- Separates public and private inputs

The normalization process is handled by the `normalized` method in the `Spell` struct:

```rust
pub fn normalized(&self) -> anyhow::Result<(NormalizedSpell, BTreeMap<App, Data>)> {
    // Normalization logic...
}
```

### Spell Verification

Spells are verified using the Charms Spell Checker, which:

1. Checks if the spell is well-formed
2. Verifies that the spell satisfies the constraints of its apps
3. Ensures that the spell's inputs and outputs are valid

The verification process is handled by the `is_correct` function in the Charms Spell Checker:

```rust
pub(crate) fn is_correct(
    spell: &NormalizedSpell,
    prev_txs: &Vec<bitcoin::Transaction>,
    app_contract_vks: &Vec<(App, AppContractVK)>,
    spell_vk: &String,
) -> bool {
    // Verification logic...
}
```

## Apps

### What is an App?

An app is a program that defines the rules for creating and transforming Charms. It specifies how tokens, NFTs, or arbitrary app state can be created, transferred, or modified.

### App Structure

An app consists of several key components:

1. **Tag**: A single character identifier (e.g., 't' for tokens, 'n' for NFTs)
2. **Identity**: A 32-byte hash that uniquely identifies the app
3. **Verification Key**: A 32-byte hash used to verify proofs

Here's the definition of the `App` struct from the codebase:

```rust
pub struct App {
    pub tag: char,
    pub identity: B32,
    pub vk: B32,
}
```

### Special App Tags

The Charms protocol defines two special app tags:

1. **TOKEN ('t')**: For fungible tokens
2. **NFT ('n')**: For non-fungible tokens

These special tags allow for simplified handling of common use cases:

```rust
/// Special `App.tag` value for fungible tokens. See [`App`] for more details.
pub const TOKEN: char = 't';
/// Special `App.tag` value for non-fungible tokens (NFTs). See [`App`] for more details.
pub const NFT: char = 'n';
```

### App Development

Apps are developed using the Charms SDK, which provides:

1. **Entrypoint Macro**: A macro for defining app entrypoints
2. **Data Handling**: Utilities for handling data serialization and deserialization
3. **zkVM Integration**: Integration with the SP1 zkVM

Here's an example of how an app entrypoint is defined using the `main!` macro:

```rust
charms_sdk::main! {
    |app: &App, tx: &Transaction, x: &Data, w: &Data| -> bool {
        // App logic...
        true
    }
}
```

### App Execution

Apps are executed in the SP1 zkVM, which:

1. Loads the app's RISC-V binary
2. Executes the binary with the provided inputs
3. Generates a zero-knowledge proof of correct execution

The execution process is handled by the `run` method in the `Prover` struct:

```rust
pub fn run(
    &self,
    app_binary: &[u8],
    app: &App,
    tx: &Transaction,
    x: &Data,
    w: &Data,
) -> anyhow::Result<()> {
    // Execution logic...
}
```

## Interaction Between Spells and Apps

Spells and apps interact in several ways:

1. **Reference**: Spells reference apps by including them in the `apps` field
2. **Inputs**: Spells provide inputs to apps through the `public_inputs` and `private_inputs` fields
3. **Execution**: Apps are executed with the inputs provided by the spell
4. **Verification**: The correctness of the spell is verified by checking that the apps' constraints are satisfied

This interaction allows for complex programmability while ensuring correctness through zero-knowledge proofs.

## Example: Token Transfer

Here's a simplified example of how a token transfer might work in Charms:

1. **App Definition**:
   - A token app with tag 't' is defined
   - The app ensures that token amounts are balanced (inputs = outputs)

2. **Spell Creation**:
   - A spell is created that references the token app
   - The spell specifies input UTXOs containing tokens
   - The spell specifies output UTXOs receiving tokens

3. **Execution and Verification**:
   - The token app is executed with the spell's inputs
   - The app verifies that token amounts are balanced
   - A zero-knowledge proof is generated

4. **Bitcoin Integration**:
   - The spell and its proof are embedded in a Bitcoin transaction
   - The transaction is broadcast to the Bitcoin network

5. **Verification by Others**:
   - Other participants extract the spell from the transaction
   - They verify the zero-knowledge proof to ensure correctness

This example demonstrates how spells and apps work together to enable programmable tokens on Bitcoin.

In the next document, we'll explore zero-knowledge proofs in Charms in more detail.
