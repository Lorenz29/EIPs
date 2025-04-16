---
eip: <to be assigned>
title: Optional Geo-Proof Incentives for Ethereum Validators
author: Lorenzo Hardoy <lorenzohardoy@gmail.com>
status: Draft
type: Standards Track
category: Core
created: 2025-04-16
---

## Abstract

This EIP introduces an optional proof-of-location mechanism for Ethereum validators. Validators who provide cryptographically signed geolocation data are eligible for increased rewards if they are located in regions with low validator density. The purpose is to incentivize decentralization of validator nodes geographically without excluding participants who prefer anonymity or cannot prove their location.

## Motivation

Ethereum's validator set is unevenly distributed geographically, which may pose centralization and network latency risks. By providing an optional mechanism for validators to attest to their physical location securely, and rewarding those in underrepresented regions, this proposal encourages a more globally balanced and resilient network.

## Specification

- Validators may submit a signed attestation containing:
  - Geographic coordinates
  - Timestamp
  - Signature from a secure enclave (e.g., Intel SGX, TPM)

- Smart contracts calculate validator density by region using submitted data.

- Validators in low-density areas receive a reward multiplier.

- Location proof is optional; validators without it continue receiving base rewards.

## Rationale

Making geolocation optional respects privacy and sovereignty while still creating incentives for global node distribution. Using secure enclaves mitigates spoofing risks.

## Implementation

### Attestation Example

A validator device generates a location attestation using a secure enclave:

```json
{
  "latitude": 37.7749,
  "longitude": -122.4194,
  "timestamp": "2025-04-16T12:00:00Z",
  "signature": "0xabc123...",
  "enclave_id": "SGX-XYZ123"
}
```

The attestation is submitted to a smart contract that verifies the signature and records the validator's region.

### Regional Definition

- The world is divided into a grid of fixed-size cells, e.g., 1° latitude × 1° longitude.
- Each cell is assigned a unique `regionId`, e.g.:

```solidity
function getRegionId(int lat, int lon) public pure returns (uint256) {
    return uint256((lat + 90) * 360 + (lon + 180));
}
```

This ensures consistent regional mapping and easy validator density lookup.

### Density-Based Reward Multiplier

A smart contract keeps a map of validator counts per geographic region:

```solidity
function calculateRewardMultiplier(uint regionId) public view returns (uint) {
    uint density = regionValidatorCount[regionId];
    if (density == 0) return 3; // max multiplier for rare zones
    if (density < 10) return 2;
    return 1;
}
```

This reward multiplier is applied to the validator's staking rewards.

## Test Cases

1. **Validator with no location proof**
   - Input: standard block proposal.
   - Output: reward = base amount.

2. **Validator in high-density area with proof**
   - Input: location proof from dense region.
   - Output: reward = base amount × 1.

3. **Validator in low-density region with valid proof**
   - Input: valid enclave-signed location in rare zone.
   - Output: reward = base amount × 2 or 3.

4. **Validator submits forged location**
   - Input: unsigned or unverifiable location data.
   - Output: rejection + optional slashing.

## Backwards Compatibility

No impact on existing validators who choose not to participate.

## Security Considerations

- Geolocation data must be signed by hardware-secure modules to prevent spoofing.
- Attestation mechanisms must ensure tamper resistance.
- Validators submitting fraudulent data may be penalized via slashing mechanisms.

## License

CC0-1.0
