# Changelog

All notable changes to this project will be documented here.

## [1.0] - 2026-09-27

### Published

- Initial public release.
- Published the technical paper:
  **What Does a VPN Actually Hide? Independent Verification of AirVPN, WireGuard, DNS and Kill-Switch Behaviour**
- Archived Version 1.0 on Zenodo.
- DOI: https://doi.org/10.5281/zenodo.22986653

### Investigation highlights

- Verified IPv4 and IPv6 routing through AirVPN/WireGuard.
- Identified and corrected a non-functional boot-time nftables kill switch.
- Verified DNS containment with packet capture.
- Tested explicit interface-bypass attempts.
- Tested local and hard-coded public DNS behaviour.
- Verified IPv6 failure paths.
- Verified reboot persistence and firewall boot ordering.
- Identified the tested AirVPN server and resolver infrastructure.
- Documented the remaining trust boundary around provider-side no-logging claims.
