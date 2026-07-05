# Flake/Overlay Policy v0.1

Status: Draft
Posture: Non-canon (downstream surface)

## Purpose

This document defines the policy governing how Nix flakes and overlays are
produced and maintained in `hummbl-dev/nix`. The `nix` repository is a
**downstream generated and policy-validated surface** that consumes package
identity and release metadata from the canonical source repository
`hummbl-dev/packages`.

## Principles

### 1. nix is a downstream surface

The `nix` repository is **not** the origin of package identity, versions, or
release artifacts. It is a downstream consumer of
[`hummbl-dev/packages`](https://github.com/hummbl-dev/packages), which is the
canonical source. All package definitions here are generated from, and
validated against, receipts emitted by `packages`.

### 2. Flake/overlay encodes reproducible install paths

Flake outputs and overlays exist to encode **reproducible install paths** for
HUMMBL tooling. They must:

- Pin exact versions and hashes for every artifact.
- Produce identical outputs across machines and over time for a given input
  receipt.
- Avoid impure inputs (e.g. `fetchTarball` without hashes, mutable refs
  without locking).
- Express install paths declaratively; no imperative mutation of the store is
  permitted in flake outputs.

### 3. Package identity and release metadata originate in packages

Package identity (package IDs, names), release versions, artifact URLs, and
cryptographic hashes all **originate in `hummbl-dev/packages`**. The `nix`
repository mirrors these fields into flake outputs and must not invent,
override, or mutate them. If a field is missing or inconsistent, the
discrepancy is reported upstream to `packages` and resolved there before the
flake output is regenerated.

### 4. Non-canon posture

This repository and the policies within it are **non-canon** with respect to
package identity and release metadata. They describe how the downstream Nix
surface is governed, not how packages themselves are defined. The canon lives
in `hummbl-dev/packages`. When this policy and `packages` disagree, `packages`
wins and this policy is updated to match.

## Scope

This policy applies to:

- The top-level flake (`flake.nix`).
- All flake outputs: `packages`, `devShells`, `overlays`, `legacyPackages`.
- Overlay definitions exposed by this repository.
- Any generated Nix expressions derived from `packages` receipts.

It does **not** apply to:

- Package identity assignment (canon: `packages`).
- Release artifact production and signing (canon: `packages`).
- Upstream source code of the packaged tools.

## Generation and validation

Flake outputs in this repository are produced by a generation step that reads
a release artifact receipt from `packages` and emits Nix expressions. A
validation step then checks that the generated flake outputs match the receipt
fields exactly (see `canonical-source-contract-v0.1.md`). A flake output that
fails validation must not be merged.

## Versioning

This policy is versioned alongside the bootstrap it describes. Changes to the
policy require a new versioned document (e.g. `flake-overlay-policy-v0.2.md`)
and a corresponding receipt entry.
