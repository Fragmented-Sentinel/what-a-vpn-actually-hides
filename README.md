# What Does a VPN Actually Hide?

**Independent verification of AirVPN, WireGuard, DNS, and kill-switch behaviour**

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.22986653.svg)](https://doi.org/10.5281/zenodo.22986653)

This repository accompanies the public technical paper:

> **Kyle Pakhet / Fragmented Sentinel. (2026). _What Does a VPN Actually Hide? Independent Verification of AirVPN, WireGuard, DNS and Kill-Switch Behaviour_ (Version 1.0). Zenodo.**  
> https://doi.org/10.5281/zenodo.22986653

## Why this project exists

This investigation started after watching NetworkChuck's video on what a VPN actually does:

https://www.youtube.com/watch?v=axfNxZ1R6C4

The video raised two practical questions worth testing directly:

1. Does a VPN actually hide DNS queries and destination traffic from a local observer or ISP?
2. Does a VPN eliminate trust, or simply move that trust from the ISP to the VPN provider?

Rather than relying on a commercial VPN leak-test page, this project tested those questions from inside a Linux virtual machine using routing inspection, `tcpdump`, `dig`, `curl`, `ping`, `systemd-resolved`, WireGuard, and nftables.

## The most important finding

The most useful result was not that the VPN worked. It was discovering that the system's **kill switch did not work when first tested**.

An nftables configuration existed, but the firewall service had failed at boot because its rules referenced the AirVPN interface before that interface existed. When WireGuard was stopped, the VM silently fell back to its normal VMware NAT route and regained direct Internet access.

The firewall was then corrected and tested again.

This is a good example of why a configuration file is not evidence that a security control is operating.

## What was tested

The investigation covered:

- IPv4 policy routing through WireGuard
- IPv6 policy routing through WireGuard
- DNS selection through `systemd-resolved`
- DNS visibility inside and outside the tunnel
- Direct attempts to use the VMware DNS resolver
- Hard-coded public DNS such as `1.1.1.1`
- Applications explicitly binding to the physical interface
- IPv6 escape attempts
- VPN-down kill-switch behaviour
- Reboot persistence
- nftables/WireGuard boot ordering
- AirVPN entry and exit addressing
- Identification of the tested AirVPN server and DNS service
- DNS resolver egress and EDNS Client Subnet behaviour
- AirVPN's published no-logging claims and the remaining trust boundary

## Key results

| Test | Result |
|---|---|
| IPv4 default traffic | Through AirVPN |
| IPv6 default traffic | Through AirVPN |
| DNS visible on physical interface | No |
| DNS visible inside VPN | Yes |
| Direct VMware DNS bypass | Blocked |
| Hard-coded public DNS | Still tunneled |
| Forced IPv4 through physical interface | Blocked |
| Forced IPv6 through physical interface | Blocked |
| VPN stopped | Ordinary Internet blocked after firewall repair |
| Reboot persistence | Passed |
| Firewall before general networking | Confirmed |
| Separate VPN entry/exit addresses | Confirmed |
| DNS resolver identity | AirVPN `Unurgunite` node |
| EDNS Client Subnet observed | No |

## What an ISP can still see

The VPN did **not** make network activity invisible.

An observer outside the tunnel could still see:

- the VPN endpoint
- protocol and port
- packet sizes
- traffic timing
- direction
- volume
- connection duration

What disappeared from that vantage point were the individual DNS queries and destination connections carried inside WireGuard.

The more precise conclusion is therefore:

> **A VPN can hide substantial destination-related metadata from the local/ISP side, but it does not eliminate metadata.**

## The remaining trust problem

The client-side behaviour was directly tested.

The harder question exists at the other end of the tunnel.

AirVPN is technically positioned to observe information such as destination IPs, DNS queries when its resolver is used, connection timing, and traffic volume. AirVPN states that it does not retain identifying traffic logs or inspect customer traffic, and several observable parts of its architecture were consistent with a privacy-focused design.

However, the central no-logging claim has not been independently demonstrated by this lab.

The main unresolved evidence gap is the lack of a published independent infrastructure/no-logs audit.

That leads to the project's overall conclusion:

> **A VPN does not eliminate trust. It relocates and concentrates it. Good engineering, direct testing, open documentation, and independent audits reduce the amount of blind trust required.**

## Paper

The publication PDF is available in this repository:

[`paper/What_A_VPN_Actually_Hides_AirVPN_Verification_v1.0.pdf`](paper/What_A_VPN_Actually_Hides_AirVPN_Verification_v1.0.pdf)

The archival publication is hosted on Zenodo:

**DOI:** https://doi.org/10.5281/zenodo.22986653

## Scope

This is a point-in-time technical investigation of one Linux VM and one AirVPN server configuration.

It is **not** proof that:

- every AirVPN server behaves identically;
- AirVPN can never change its policies or configuration;
- AirVPN keeps no server-side logs;
- a hosting provider has no additional visibility;
- a VPN provides anonymity against every adversary.

The paper deliberately distinguishes direct experimental evidence from provider claims and third-party reporting.

## Independence

This work is independent.

Kyle Pakhet publishes cybersecurity research under the name **Fragmented Sentinel**. This project is not affiliated with, sponsored by, or endorsed by AirVPN, NetworkChuck, tzulo, WireGuard, Zenodo, or any other referenced organization.

## References

The full reference list is included in the paper. Core sources include:

- NetworkChuck video that prompted the investigation:  
  https://www.youtube.com/watch?v=axfNxZ1R6C4
- AirVPN:  
  https://airvpn.org/
- AirVPN Privacy Notice:  
  https://airvpn.org/privacy
- TorrentFreak VPN privacy survey:  
  https://torrentfreak.com/best-vpn-anonymous-no-logging/
- RTINGS AirVPN review:  
  https://www.rtings.com/vpn/reviews/airvpn/airvpn
- Akamai DNS resolver diagnostic:  
  https://www.akamai.com/blog/developers/introducing-new-whoami-tool-dns-resolver-information
- WireGuard Quick Start:  
  https://www.wireguard.com/quickstart/

## Citation

If you reference this work, please cite:

> Pakhet, K. (2026). _What Does a VPN Actually Hide? Independent Verification of AirVPN, WireGuard, DNS and Kill-Switch Behaviour_ (Version 1.0). Zenodo. https://doi.org/10.5281/zenodo.22986653

See [`CITATION.cff`](CITATION.cff) for machine-readable citation metadata.

## License

This work is licensed under the **Creative Commons Attribution 4.0 International License (CC BY 4.0)**.

See [`LICENSE`](LICENSE).
