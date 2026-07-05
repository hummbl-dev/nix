# Nix Bootstrap v0.1 Receipt

Repository: hummbl-dev/nix
Branch: feat/devin/bootstrap-policy
Base: main
Date: 2025-07-04
Status: Bootstrap complete

## Summary

Bootstraps the `hummbl-dev/nix` policy surface with the foundational
flake/overlay policy, canonical-source contract, and devshell-first strategy.
No flake outputs are generated in this bootstrap; this commit establishes the
governing documents that subsequent generation and validation steps will
implement.

## Issues addressed

- #1 — Bootstrap Nix flake/overlay policy and canonical-source contract
- #2 — Add devshell-first strategy for reproducible HUMMBL environments

## Files added

| Path | Purpose |
| --- | --- |
| `policy/flake-overlay-policy-v0.1.md` | Flake/overlay policy; nix as downstream generated/policy-validated surface from `hummbl-dev/packages`. |
| `policy/canonical-source-contract-v0.1.md` | Canonical source contract; input/output, field mapping, and validation rules. |
| `policy/devshell-first-strategy-v0.1.md` | Devshell-first strategy; reproducible dev shells and agent runtime environments. |
| `receipts/nix-bootstrap-v0.1-receipt.md` | This receipt. |

## Posture

All documents are **non-canon** with respect to package identity and release
metadata. The canonical source is `hummbl-dev/packages`. These documents
describe how the downstream Nix surface consumes and validates canon data.

## Field mapping (defined, not yet implemented)

| Receipt field (from packages) | Flake output field |
| --- | --- |
| `packageId` | flake output name |
| `version` | `version` |
| `artifactUrl` | `src` |
| `sha256` | `hash` |

Validation rule: flake outputs must match receipt fields exactly. Implementation
of the generator and validator is deferred to a follow-up change.

## Next steps

1. Implement the receipt-driven generator that emits `flake.nix` outputs.
2. Implement the validator enforcing the exact-match rule from
   `canonical-source-contract-v0.1.md`.
3. Produce the first `devShells.default` per `devshell-first-strategy-v0.1.md`.
