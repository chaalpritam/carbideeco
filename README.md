# CarbideEco

Meta-repository for the [Carbide Network](https://carbide.network) ecosystem — a decentralized storage marketplace where anyone can run a provider node and earn, and any client can store files across the network with end-to-end encryption.

This repo aggregates every component of the stack as a git submodule, so you can clone the entire ecosystem in one shot.

## Getting started

```sh
git clone --recurse-submodules git@github.com:chaalpritam/CarbideEco.git
cd CarbideEco
```

If you already cloned without `--recurse-submodules`:

```sh
git submodule update --init --recursive
```

To pull the latest commit on every submodule's tracked branch:

```sh
git submodule update --remote --merge
```

## Components

| Submodule | Description |
| --- | --- |
| [`carbide-node`](./carbide-node) | Storage provider node (Rust + Tauri GUI). Earns by contributing capacity to the marketplace. |
| [`carbide-contracts`](./carbide-contracts) | On-chain Solana programs for the provider registry and payment escrow. |
| [`carbide-discovery-service`](./carbide-discovery-service) | Node.js/TypeScript microservice that indexes the on-chain registry and serves provider discovery + quotes. |
| [`carbide-ios-sdk`](./carbide-ios-sdk) | Swift SDK (iOS/macOS) for provider discovery, client-side encryption, and file upload/download. |
| [`Carbide`](./Carbide) | iOS app for decentralized file storage, built on `carbide-ios-sdk`. |
| [`CarbideDrive`](./CarbideDrive) | macOS desktop sync client, built on `carbide-ios-sdk`. |
| [`carbide.network`](./carbide.network) | Marketing site / web frontend at carbidenetwork.xyz. |
| [`carbide-dev-docs`](./carbide-dev-docs) | Developer documentation covering architecture, SDK usage, and provider setup. |
| [`homebrew-carbide`](./homebrew-carbide) | Homebrew tap for installing `carbide-node` on macOS. |

## Repository layout

The ecosystem is split across multiple repos so that each component can ship and version independently. CarbideEco pins each submodule to a specific commit; bumping a submodule is an explicit commit in this repo, which gives a single source of truth for "which versions work together."
