# Devshell-First Strategy v0.1

Status: Draft
Posture: Non-canon (downstream surface)

## Purpose

This document defines the devshell-first strategy for `hummbl-dev/nix`: how
reproducible development shells are produced and prioritized as the first
class of Nix outputs, before packaged derivations. It complements
`flake-overlay-policy-v0.1.md` and `canonical-source-contract-v0.1.md`.

## Core strategy

### 1. Reproducible dev shells before building packages

The first deliverable of the Nix surface is a set of **reproducible
development shells** that can be entered immediately, without first building
or installing packaged derivations. This means:

- `devShells` are generated and validated ahead of `packages.*` outputs.
- A contributor or agent can `nix develop` into a working environment before
  any packaged tool is built as a closure.
- Dev shells declare their toolchain explicitly; they do not depend on ambient
  system state.

### 2. devShells for HUMMBL development environments

`devShells.*` provide the canonical HUMMBL development environments:

- A default `devShells.default` for general HUMMBL development.
- Named per-component shells (e.g. `devShells.<component>`) where a component
  requires a distinct toolchain or set of inputs.
- Shells include the tools, libraries, and environment variables needed to
  develop, test, and lint HUMMBL code, pinned to exact versions.

### 3. Agent runtime environments via devShells

Automated agents (CI, build agents, AI coding agents) obtain their runtime
environments through `devShells` rather than host-installed tooling. This
ensures an agent runs in the same reproducible environment as a human
contributor. Agent-facing shells:

- Are pure: no reliance on the host's globally installed tools.
- Are pinned: identical inputs produce identical environments.
- Are declarative: the shell is fully described by the flake, not by
  imperative setup scripts.

### 4. Toolchain pinning for reproducibility

Every dev shell pins its toolchain via locked flake inputs:

- `nixpkgs` is pinned to a specific locked revision.
- Compilers, language runtimes, linters, and formatters are pinned to versions
  declared in the release artifact receipt from `hummbl-dev/packages`.
- No mutable refs (`github:owner/repo` without `rev`) are permitted in dev
  shell inputs.
- Toolchain upgrades are deliberate: they require a new receipt from
  `packages` and regeneration of the affected shells.

## Relationship to other outputs

Dev shells are the first outputs produced and validated, but they are not the
only outputs. Once dev shells are reproducible, `packages.*` and
`overlays.default` follow from the same receipt, per
`canonical-source-contract-v0.1.md`. The ordering is a delivery priority, not
a dependency inversion: packaged derivations may still be consumed by dev
shells where appropriate, provided their inputs are pinned.

## Non-canon posture

This strategy governs the downstream Nix surface only. It is **non-canon**:
package identity, release artifacts, and toolchain versions originate in
`hummbl-dev/packages`. Where this strategy and `packages` disagree, `packages`
is authoritative and this document is updated to match.
