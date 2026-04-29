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

## Stack

Payments and the provider registry live on **Solana** (devnet today,
mainnet-beta as a default-flippable target). Providers register themselves
via the `carbide_registry` Anchor program; clients fund deals into the
`carbide_escrow` program with USDC SPL tokens, and an authorised verifier
co-signs each periodic release after a valid proof-of-storage. Ed25519
keypairs follow Solana's standard derivation path `m/44'/501'/0'/0'`.

## Repository layout

The ecosystem is split across multiple repos so that each component can ship and version independently. CarbideEco pins each submodule to a specific commit; bumping a submodule is an explicit commit in this repo, which gives a single source of truth for "which versions work together."

## Running the stack locally

You don't need every component to be useful — pick the slice that matches what you're doing. Each submodule has its own README with a "Running locally" section; this is the cross-cutting overview.

| Layer | Component | Local command |
| --- | --- | --- |
| On-chain (optional) | `carbide-contracts` | `anchor test` (localnet validator with cloned devnet USDC) |
| Discovery | `carbide-discovery-service` | `npm install && npm run dev` (`:9090`) |
| Provider | `carbide-node` (`carbide-provider`) | `cargo run --bin carbide-provider -- start --port 8080 --capacity-gb 25` |
| Client CLI | `carbide-node` (`carbide-client`) | `cargo run --bin carbide-client -- upload --provider http://127.0.0.1:8080 --file <path>` |
| iOS/macOS app | `Carbide` / `CarbideDrive` | open the Xcode project, point Settings → *Discovery Service URL* at `http://localhost:9090` |
| Web | `carbide.network` | `npm install && npm run dev` (`:3000`) |

Two common slices:

**Single-laptop end-to-end demo** (no on-chain, no discovery):

```sh
# Terminal 1 — provider
cd carbide-node
cargo run --bin carbide-provider -- start --port 8080 --capacity-gb 1

# Terminal 2 — client (direct upload, skips discovery + Solana)
cargo run --bin carbide-client -- upload \
    --provider http://127.0.0.1:8080 --file ~/file.pdf
```

**Full marketplace slice** (discovery service + provider + iOS/macOS app):

```sh
# Terminal 1 — discovery
cd carbide-discovery-service && npm install && npm run dev

# Terminal 2 — provider (with discovery_endpoint pointed at localhost:9090
# in $(brew --prefix)/etc/carbide/provider.toml or via env)
cd ../carbide-node
CARBIDE_DISCOVERY_ENDPOINT=http://127.0.0.1:9090 \
  cargo run --bin carbide-provider -- start --port 8080 --capacity-gb 25

# Terminal 3 — open CarbideDrive.xcodeproj, set Discovery Service URL to
# http://localhost:9090 in Settings, hit Cmd+R
```

For the **two-laptop demo** (one provider, one client over LAN or Tailscale), see [`carbide-node/Readme.md`](./carbide-node/Readme.md#running-locally) — the only extra step is editing `[network] advertise_address` in `provider.toml` to the provider laptop's reachable IP before starting the service.

## Running in production

Each component has its own deployment story. The pinned submodule SHAs in this repo represent a known-good combination — bump a submodule when you ship that component.

| Component | Production target | Notes |
| --- | --- | --- |
| `carbide-node` | Homebrew tap → launchd on macOS | `brew install --HEAD chaalpritam/carbide/carbide-node`; edit `provider.toml`; `brew services start`. See [`homebrew-carbide`](./homebrew-carbide). |
| `carbide-discovery-service` | Container or Node host | Ships a `Dockerfile`; deploys cleanly to Railway, Render, Fly.io, ECS, GKE. Public default DNS: `discovery.carbidenetwork.xyz`. |
| `carbide-contracts` | Solana mainnet-beta | `anchor deploy --provider.cluster mainnet`; record program IDs and the USDC mint into `provider.toml` and the discovery service env. |
| `carbide-ios-sdk` | Swift Package, tagged release | Consumers add a versioned dependency on the GitHub repo. |
| `Carbide` (iOS) | App Store / TestFlight | Xcode archive + App Store Connect upload; production discovery URL by default. |
| `CarbideDrive` (macOS) | Notarized DMG (Developer ID) or App Store | `notarytool submit … --wait` then `stapler staple`; package as a DMG. |
| `carbide.network` | Container or Vercel/Netlify | Next.js standalone build; pass `NEXT_PUBLIC_API_URL` at build time. |

For each, the relevant submodule README's "Running in production" section is the source of truth. Bump the submodule pointer here whenever you ship a new version, mirroring the existing `Bump <component>: …` commit style.
