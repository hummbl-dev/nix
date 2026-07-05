# Canonical Source Contract v0.1

Status: Draft
Posture: Non-canon (derived from `hummbl-dev/packages`)

## Purpose

This contract defines the binding relationship between the canonical source
repository [`hummbl-dev/packages`](https://github.com/hummbl-dev/packages) and
the downstream Nix surface in `hummbl-dev/nix`. It specifies the inputs
consumed, the outputs produced, the field mapping between them, and the
validation rules that enforce correctness.

## Input

The generation step consumes two artifacts from `hummbl-dev/packages`:

1. **Package identity registry** — the canonical list of package IDs, names,
   and metadata owned by `packages`.
2. **Release artifact receipt** — a per-release record emitted by `packages`
   that binds a package identity to a concrete artifact: its version,
   downloadable URL, and cryptographic hash.

Together these constitute the sole authoritative input for producing Nix
flake outputs in this repository. No other source of package identity or
release metadata is permitted.

## Output

From the input, the generation step produces **Nix flake outputs**:

- `packages.*` — one derivable package per package identity in the registry.
- `devShells.*` — reproducible development shells (see
  `devshell-first-strategy-v0.1.md`).
- `overlays.default` — an overlay exposing the same packages for consumption
  via `nixpkgs` overlays.

Every flake output is generated from the receipt; none are authored by hand
against ad-hoc sources.

## Field mapping

Each release artifact receipt entry maps to a flake output as follows:

| Receipt field | Flake output field | Notes |
| --- | --- | --- |
| `packageId` | flake output name | The `packages.<name>` and overlay key. |
| `version` | `version` | Package version string, used in derivation metadata. |
| `artifactUrl` | `src` | Passed to `fetchurl` / `fetchzip` as the source URL. |
| `sha256` | `hash` | The fixed-output hash; must be a valid SRI or sha256 string. |

Additional receipt fields (e.g. `license`, `description`, `homepage`) are
carried through into derivation metadata where applicable, but the four fields
above are the **required** mapping and must always be present.

## Validation

Before a generated flake output is accepted into this repository, a validation
step verifies:

1. **Field presence** — every receipt entry has `packageId`, `version`,
   `artifactUrl`, and `sha256`.
2. **Exact match** — each flake output's name, version, `src` URL, and `hash`
   match the corresponding receipt fields exactly. No renaming, rewriting, or
   defaulting is allowed.
3. **Hash well-formedness** — `sha256` values are valid and consumable by the
   chosen fetcher.
4. **Completeness** — every package identity in the registry has exactly one
   flake output; no extras, no missing entries.
5. **Reproducibility** — building a flake output from a given receipt yields a
   fixed-output match with the declared `hash`.

A flake output that fails any validation check is rejected and must not be
merged. Discrepancies are reported upstream to `hummbl-dev/packages` for
resolution at the canon source.

## Non-canon posture

This contract describes how the downstream Nix surface consumes canon data.
It is **non-canon**: the authoritative definitions of package identity and
release artifacts live in `hummbl-dev/packages`. If this contract and
`packages` conflict, `packages` is correct and this contract is revised to
match.
