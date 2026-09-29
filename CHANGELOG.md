# Changelog

All notable changes to this project will be documented here.

## [1.1] - 2026-09-28

### Published
- Published Version 1.1 of *What Does a VPN Actually Hide? Independent Verification of AirVPN, WireGuard, DNS and Kill-Switch Behaviour*.
- Archived Version 1.1 on Zenodo.
- DOI: https://doi.org/10.5281/zenodo.23027447
- Added a sanitized reproducibility package containing configuration snapshots, software versions, evidence indexes, hashes, and reviewed packet-capture artifacts.

### Corrections and clarifications
- Corrected the test environment from Ubuntu 24.04 to Linux Mint 22.3.
- Clarified that the tested kill switch was a custom system-level nftables policy used with `wg-quick`, not AirVPN Eddie or Network Lock.
- Clarified that WireGuard itself does not push DNS configuration.
- Documented that `wg-quick` used the system `resolvconf` compatibility interface, backed by `systemd-resolved`, to apply AirVPN DNS settings.
- Tightened the distinction between directly observed results, provider statements, and third-party reporting.
- Clarified that absence of an independent no-logging audit does not demonstrate that logging occurs.

### Additional testing
- Added controlled IPv6 firewall-enforcement testing using a known-working non-VPN VMware IPv6 underlay.
- Demonstrated that IPv6 traffic succeeded when temporarily permitted and was blocked by the hardened nftables policy when the exception was removed.
- Added deliberate nftables startup-failure injection.
- Demonstrated that systemd ordering alone did not guarantee fail-closed startup behaviour.
- Added explicit systemd dependencies requiring nftables before NetworkManager and WireGuard startup.
- Verified that the hardened configuration remained fail-closed during a deliberate firewall startup failure.
- Identified a shutdown leakage condition caused by a broad OUTPUT `ct state established,related accept` rule.
- Removed the broad OUTPUT established-state rule and verified that the reproduced shutdown escape no longer occurred.
- Added packet-level DNS verification showing plaintext DNS on the logical `airvpn` interface while no port-53 DNS traffic appeared on the physical `ens33` interface.

### Reproducibility
- Added `REPRODUCING.md`.
- Added an evidence index mapping major conclusions to preserved artifacts and SHA256 hashes.
- Added sanitized firewall, routing, DNS, systemd, WireGuard, and package-version snapshots.
- Included reviewed public evidence artifacts while retaining privacy-sensitive raw captures privately.

## [1.0] - 2026-09-27

### Published
- Initial public release.
- Published the technical paper: *What Does a VPN Actually Hide? Independent Verification of AirVPN, WireGuard, DNS and Kill-Switch Behaviour*.
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
