# snarkjs-bellman-adapter

An adapter for snarkjs(generate proof) and bellman (verify proof).

It allows zk devs to compile circuits with circom, generate proofs with SnarkJS, and verify proofs on Bellman.

## Background

As zero knowledge proofs become practical, there will be more zk applications. In order to create a zk application, dev needs to write circuit and build application, which normally involves generating proofs and verifying proofs.

There are popular zk development tools and libraries, such as `Circom` (a DSL for circuit writing), `SnarkJS` (a Javascript library for generating proofs and verifications), `Bellman` and `ArkWorks` (zkSnark library based on Rust).

The goal is to provide a simple tool to build multi-chain zk applications using these libraries, so that Rust based blockchain developers can directly build applications without re-creating libraries.

This tutorial was developed to meet the requirements for verifying zk proofs based on Rust's back-end services.

We know that circom is good for writing circuits, and usually after the circuits are written, we can easily generate the corresponding proof with SnarkJS for verification. However, when we use Rust for application development (e.g. developing applications on Substrate or Solana blockchains), we did not find a suitable Rust library to match the proof produced by SnarkJS, so we developed this adapter to decode the data generated by the SnarkJS to satisfy the data structure requirement of Bellman.

So, the adapter provides the following benefits:

- use `circom` to write circiuit and compile into R1CS easily (circom is an excellent DSL to write circuit, which is similar to the programming paradigms of high-level languages, instead of Bellman or arkworks. So, we can aviod many latent errors).
- use `snarkjs` to generate zk proofs
- use `bellman` to verify the proof generated by the snarkjs (when we need to verify the proof in the backend using the Rust, whether it is blockchain or other services, we need a rust library to verify the snarkjs' proof. We did not find a Rust library can do it. bellman is an advanced zk library developed by Zcash. It has widely used in production environments, so we choose it as the rust library to use).

## Pre-requirements

> we know that SnarkJS supports `bn128` and `bls12_381` curves. Bellman support only `bls12_381` curve. So we choose `bls12_381` curve. So, `if you developed a zk application using bn128 curve in the past, you only need to change the curve to bls12_381`.

### 1. install SnarkJS

```
npm install -g snarkjs@latest
```

### 2. install circom compiler

**Attention: recommend to install circom using the latest source code and compile it into the executable file `circom`. Don't install it by the npm which is not the latest library**.

```
git clone https://github.com/iden3/circom.git
cargo build --release
```

**Also, you need to set `circom` as a global command.**

### 3. prepare circuit and inputs

- write a circuit named `circuit.circom` using circom (in the dir `circuit`, we wrote a simple demo circuit of `a * b = c`. We want to prove that we know two numbers whose product is `33`)
- write an input file named `inputs.json`(This file contains two number which we know but we don't want the verifier to know)
> if you are interested in it, you can change it to `{"a": 3, "b": 11}` to generate a proof, you can find it will be verifed correctly too.

## Use the adapter

### 1. Generate zk proof of BLS12_381 curve

Generate proof and verification key with `circuit.circom` and `inputs.json` in the dir `circuit` of this project by `start.sh`.

```
./start.sh
```


### 2. Decode the proof into uncompressed data

You have generated `proof.json` and `verification_key.json`，now you can go to the directory `prove` and run these command:

```
cd prove && npm install
cd src && node adapter.js
```

After that, you can see the generated uncompressed data files `proof_uncompressed.json` and `vkey_uncompressed.json`.

### 3. Encode the uncompressed data into Affine and Verify

Go to the directory `verify/src/adapter` and run the test:

```
cd ../../verify/src/adapter
cargo test snark_proof_bellman_verify -- --nocapture
```

if you see the below output, which means the verification passed.

```
running 1 test
>>>>start encode the uncompressed data to Affine<<<<<
>>>>end verification<<<<<<<
test adapter::snark_proof_bellman_verify ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.07s
```

## Customize zk circuit and verify

For customized circuits, you need to:
- modify the `circuit.circom` and `inputs.json` in the dir `circuit`
- modify the public parameter of the function `Fr::from_str_vartime("xxxxxx")` in the `verify/src/adapter/mod.rs`


## Bellman-verifier with `no_std`

We have improved `bellman-verifier` to support `no_std` execution environment.

if you want to use bellman to verify the proof in `no_std` environment, you can follow it in your `Cargo.toml`:

```toml
[dependencies]
bellman-verifier = { git = "https://github.com/DoraFactory/snarkjs-bellman-adapter.git", default-features = false, version = "0.1.0"}

[features]
default = ["std"]
std = [
	"bellman-verifier/std",
]
```

> For the Polkadot devs, you need to change the branch of `bellman-verifier` to satisfy your chain's version.