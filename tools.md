# Tooling

## `cargo contract`

For compiling, building and deploying contracts to the network you will need `cargo contract` installed. Each network may be compatible with a different release of it. 

To interact with current version of Aleph Zero Testnet, you will need `cargo contract` version `2.0.0-beta.1`:
```
cargo install cargo-contract --version 2.0.0-beta.1
```

Latest releases of `cargo contract` are [adding support](https://github.com/paritytech/cargo-contract/pull/870) for new arguments in `Contracts::upload_code` which has not been yet integrated into Aleph Zero chain.

## `substrate-contracts-node`

Similarly, if you're using substrate-contracts-node, you need to be using version [`0.22.1`]( https://github.com/paritytech/substrate-contracts-node/releases/tag/v0.22.1).

## Building with Docker

We've prepared a Docker image with required tooling to build contracts compatible with current Aleph Zero Testnet (contracts pallet at version `0.9.32`). If you prefer to use it, take a look at https://github.com/Cardinal-Cryptography/docker-ink-dev.
