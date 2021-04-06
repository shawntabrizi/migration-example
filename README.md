# Substrate Migration Example

This repo demonstrates a simple runtime migration.
The intention is to migrate from the state of the `pre-migration` branch to `post-migration`.

The repo contains contains a FRAME-based [Substrate](https://www.substrate.io/) node based on the
Node Template that includes a modified version of the Nicks pallet (the pallet was changed to store
a first and last name `post-migration`).

## Getting Started

In order to try out the migration you need to follow the usual Substrate Template node setup:

This project contains some configuration files to help get started :hammer_and_wrench:

### Rust Setup

Follow the [Rust setup instructions](./doc/rust-setup.md) before using the included Makefile to
build the Node Template.

### Makefile

This project uses a [Makefile](Makefile) to document helpful commands and make it easier to execute
them. Get started by running these [`make`](https://www.gnu.org/software/make/manual/make.html)
targets:

1. `make init` - Run the [init script](scripts/init.sh) to configure the Rust toolchain for
   [WebAssembly compilation](https://substrate.dev/docs/en/knowledgebase/getting-started/#webassembly-compilation).
1. `make run` - Build and launch this project in development mode.

The init script and Makefile both specify the version of the
[Rust nightly compiler](https://substrate.dev/docs/en/knowledgebase/getting-started/#rust-nightly-toolchain)
that this project depends on.

### Build

The `make run` command will perform an initial build. Use the following command to build the node
without launching it:

```sh
make build
```

### Embedded Docs

Once the project has been built, the following command can be used to explore all parameters and
subcommands:

```sh
./target/release/node-template -h
```

## Run

In order to make the process easier the repo contains the `nicks-pre.chainspec.json` which
includes the runtime of the `pre-migration` branch.

The `make run-pre` command will launch a temporary node (its state will be discarded after you
terminate the process) with the above-mentioned chainspec.

### Single-Node Development Chain

After the project has been built, there are other ways to launch the node.
This command will start the single-node development chain with persistent state:

```bash
./target/release/node-template --dev
```

Purge the development chain's state:

```bash
./target/release/node-template purge-chain --dev
```

Start the development chain with detailed logging:

```bash
RUST_LOG=debug RUST_BACKTRACE=1 ./target/release/node-template -lruntime=debug --dev
```

Start the development chain with the `pre-migration` chainspec (with Alice as validator and runtime
logging enabled):
```bash
./target/release/node-template --chain='nicks-pre.chainspec.json' -lruntime=debug --alice
```

## Migrate

Now that we have the node up and running with the old chainspec we need to trigger the actual
migration.
For this we will use [Polkadot-js apps](https://polkadot.js.org/apps).

Navigate to `Settings > Developer` and add the contents of `types.json` to tell Polkadot-js about
types that will be introduced by the `post-migration` code.

Navigate to `Developer > Sudo` where we will use the root account (Alice) to trigger a sudo
call to `system.setCode`.

> Note: Make sure that you compiled the new runtime (calling either `cargo run --release` or
`cargo build --release`, should be the case if you ran `make run-pre`).

Enable file upload for the code and then grab the runtime from
`./target/release/wbuild/node-template-runtime/node_template_runtime.compact.wasm`.
Click the "with weight override" toggle to ignore block weights (code updates are heavy) and set it
to some arbitrary value.
You are now ready to trigger the upgrade and migration via "Submit Sudo Unchecked".

You should see the version in the top left switch from `#100` to `#101` and you should see something
like the following in the logs:
```
Apr 06 17:19:54.410  INFO  >>> Updating MyNicks storage. Migrating 3 nicknames...
Apr 06 17:19:54.410  INFO      Migrated nickname for d43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d (5GrwvaEF...)...
Apr 06 17:19:54.410  INFO      Migrated nickname for 8eaf04151687736326c9fea17e25fc5287613693c912909cb226aa4794f26a48 (5FHneW46...)...
Apr 06 17:19:54.410  INFO      Migrated nickname for 90b5ab205c6974c9ea841be688864633dc9ca8a357843eeacf2314649965fe22 (5FLSigC9...)...
Apr 06 17:19:54.410  INFO  <<< MyNicks storage updated! Migrated 3 nicknames ✅
```