# What Does a VPN Actually Hide?

Independent verification of AirVPN, WireGuard, DNS, IPv6, and custom nftables kill-switch behaviour.

## Current release

**Version:** 1.1  
**Published:** 2026-09-28  
**Zenodo DOI:** https://doi.org/10.5281/zenodo.23027447  
**All versions:** https://doi.org/10.5281/zenodo.22986652

This repository accompanies the public technical paper:

**Kyle Pakhet / Fragmented Sentinel. (2026). _What Does a VPN Actually Hide? Independent Verification of AirVPN, WireGuard, DNS and Kill-Switch Behaviour_ (Version 1.1). Zenodo.**  
https://doi.org/10.5281/zenodo.23027447

Version 1.0 remains archived at:

https://doi.org/10.5281/zenodo.22986653

---

## Why this project exists

This investigation started after watching NetworkChuck's video on what a VPN actually does:

https://www.youtube.com/watch?v=axfNxZ1R6C4

The video raised two practical questions worth testing directly:

- Does a VPN actually hide DNS queries and destination traffic from a local observer or ISP?
- Does a VPN eliminate trust, or simply move that trust from the ISP to the VPN provider?

Rather than relying on a commercial VPN leak-test page, this project tested those questions from inside a Linux Mint virtual machine using routing inspection, `tcpdump`, `dig`, `curl`, `ping`, `systemd-resolved`, WireGuard, `wg-quick`, systemd, and nftables.

---

## Test environment

The tested client was:

- Linux Mint 22.3 x86_64
- VMware virtual machine
- AirVPN-generated WireGuard configuration
- `wg-quick`
- `systemd-resolved`
- NetworkManager
- a separately maintained system-level nftables kill switch

The kill switch tested in this project was **not AirVPN Eddie or AirVPN Network Lock**.

Those AirVPN client features were not used or evaluated.

---

## The most important finding

The most useful result was not simply that the VPN worked.

It was discovering that the system's **custom nftables kill switch did not initially work at boot**.

The persistent nftables configuration referenced the `airvpn` interface in a way that required the interface to already exist. During boot, WireGuard had not yet created that interface, so the firewall service failed to load.

When WireGuard was later stopped, the VM was able to fall back to the normal VMware NAT path.

This failure belonged to the tester-maintained nftables configuration, not AirVPN Eddie or Network Lock.

The firewall was corrected, retested, and then subjected to deliberate startup-failure and shutdown testing.

This is a practical example of why a configuration file or a "connected" indicator is not evidence that a security control is actually operating.

---

## What was tested

The investigation covered:

- IPv4 policy routing through WireGuard
- IPv6 policy routing through WireGuard
- controlled IPv6 firewall enforcement using a known-working non-VPN underlay
- DNS selection through `systemd-resolved`
- the mechanism used by `wg-quick` to configure DNS
- DNS visibility inside and outside the tunnel
- direct attempts to use the VMware DNS resolver
- hard-coded public DNS such as `1.1.1.1`
- applications explicitly binding to the physical interface
- VPN-down kill-switch behaviour
- deliberate nftables startup-failure injection
- systemd dependency hardening
- full reboot behaviour
- established-connection shutdown behaviour
- AirVPN entry and exit addressing
- identification of the tested AirVPN DNS service
- DNS resolver egress and EDNS Client Subnet behaviour
- AirVPN's published no-logging claims and the remaining trust boundary
- reproducibility and evidence preservation

---

## Key results

| Test | Result |
|---|---|
| IPv4 default traffic | Routed through AirVPN |
| IPv6 default traffic | Routed through AirVPN |
| DNS visible on physical interface | No port-53 DNS observed |
| DNS visible inside VPN | Yes |
| Direct VMware DNS bypass | Blocked |
| Hard-coded public DNS | Still routed through WireGuard |
| Forced IPv4 through physical interface | Blocked by hardened nftables policy |
| Known-working non-VPN IPv6 path | Blocked by hardened nftables policy |
| VPN stopped | Ordinary Internet blocked after firewall repair |
| Deliberate nftables startup failure before dependency hardening | Direct traffic escaped |
| Deliberate nftables startup failure after dependency hardening | NetworkManager and WireGuard remained inactive |
| Shutdown with broad OUTPUT established-state rule | Known VPN TCP flow emitted a FIN outside the tunnel |
| Shutdown after removing broad OUTPUT established-state rule | Reproduced escape no longer observed |
| Separate VPN entry/exit addresses | Confirmed for tested path |
| DNS resolver identity | AirVPN Unurgunite node |
| EDNS Client Subnet observed | No, in tested diagnostic query |

---

## IPv6 clarification

Version 1.0 reported that forced IPv6 attempts through the physical interface failed.

That result alone was not sufficient to demonstrate firewall enforcement because the VMware NAT environment did not provide a working non-VPN global IPv6 route.

Version 1.1 corrected this limitation.

A temporary ULA IPv6 network was established directly between the VM's `ens33` interface and the VMware host's `vmnet8` interface.

With a temporary nftables exception allowing IPv6 over `ens33`, ICMPv6 succeeded and was externally captured.

After removing that temporary exception and restoring the hardened firewall, the same IPv6 test failed and no matching packets were observed on the host capture.

This demonstrates firewall enforcement against a known-working non-VPN IPv6 path.

It does **not** constitute testing against a native public IPv6 Internet uplink.

---

## DNS clarification

WireGuard itself does not negotiate or push DNS settings.

In this test, the AirVPN-generated `wg-quick` profile contained:

```ini
DNS = 10.128.0.1, fd7d:76ee:e68f:a993::1
```

During interface setup, `wg-quick` invoked:

```text
resolvconf -a airvpn -m 0 -x
```

On the tested Linux Mint system:

```text
/usr/sbin/resolvconf -> /usr/bin/resolvectl
```

and DNS configuration was handled by `systemd-resolved`.

The `airvpn` link received the AirVPN DNS servers and the route-only domain:

```text
~.
```

Stopping `wg-quick@airvpn` removed that resolver configuration. Restarting it restored the configuration.

Simultaneous packet capture showed conventional port-53 DNS on the logical `airvpn` interface, while the physical `ens33` interface showed only encrypted WireGuard UDP traffic to the VPN endpoint.

Plain DNS visible inside the logical VPN interface is therefore not, by itself, evidence of a DNS leak.

---

## Startup and shutdown hardening

Version 1.1 added controlled failure testing rather than relying only on successful reboot observations.

When nftables was deliberately forced to fail during boot, the original systemd dependency arrangement still allowed NetworkManager to start. The VM obtained a physical-interface address and direct Internet traffic was externally observed.

The hardened configuration added explicit dependencies so that NetworkManager requires nftables, and `wg-quick@airvpn` requires both nftables and NetworkManager.

During a final deliberate firewall startup failure:

- nftables failed
- NetworkManager remained inactive
- `wg-quick@airvpn` remained inactive
- `ens33` remained unconfigured
- the `airvpn` interface did not exist
- direct canary probes failed
- no public Internet traffic was observed during the failed state

A separate shutdown experiment identified a broad OUTPUT rule:

```text
ct state established,related accept
```

as the enabling firewall condition for a known established VPN TCP flow to emit a FIN packet over the physical interface during reboot.

Removing that broad OUTPUT rule prevented the reproduced shutdown escape.

---

## What an ISP or local observer can still see

The VPN did not make network activity invisible.

An observer outside the tunnel could still see information such as:

- the VPN endpoint
- transport protocol and port
- packet sizes
- timing
- direction
- volume
- session duration
- traffic bursts

What disappeared from that vantage point were the individual DNS queries and destination connections carried inside WireGuard.

The more precise conclusion is therefore:

> A VPN can hide substantial destination-related metadata from the local or ISP side, but it does not eliminate metadata.

---

## The remaining trust problem

The client-side behaviour was directly tested.

The harder question exists at the other end of the tunnel.

AirVPN is technically positioned to observe or correlate information such as destination IP addresses, DNS queries when its resolver is used, connection timing, and traffic volume.

AirVPN states that it does not retain identifying traffic logs or inspect customer traffic. TorrentFreak records similar statements supplied by the provider.

Those statements were **not independently verified by this lab**.

RTINGS reports that AirVPN has not published an independent third-party audit of its no-logging policy or security infrastructure.

The absence of such an audit does **not** demonstrate that AirVPN logs traffic. It means that provider-side retention remains outside what this client-side investigation can independently establish.

That leads to the project's overall conclusion:

> A VPN does not eliminate trust. It relocates and concentrates it. Direct testing can reduce blind trust on the client side; transparency and independent verification can reduce it on the provider side.

---

## Paper

The current publication PDF is available in this repository:

[Version 1.1 PDF](paper/What_A_VPN_Actually_Hides_AirVPN_Verification_v1.1_Zenodo.pdf)

Version 1.0 remains available for historical comparison:

[Version 1.0 PDF](paper/What_A_VPN_Actually_Hides_AirVPN_Verification_v1.0.pdf)

The archival Version 1.1 publication is hosted on Zenodo:

https://doi.org/10.5281/zenodo.23027447

All versions:

https://doi.org/10.5281/zenodo.22986652

---

## Reproducibility

Version 1.1 includes a sanitized reproducibility package containing:

- operating-system and package-version information
- sanitized WireGuard configuration
- nftables configuration
- live firewall state
- routing and policy-routing state
- `systemd-resolved` state
- systemd service dependency information
- `wg-quick` DNS integration details
- reproduction procedures
- evidence indexes
- SHA256 hashes
- reviewed packet-capture artifacts and summaries

The canonical reproducibility package is included with the Zenodo Version 1.1 record:

https://doi.org/10.5281/zenodo.23027447

Raw research captures containing unnecessary local metadata were retained privately rather than published indiscriminately.

---

## Scope

This is a point-in-time technical investigation of one Linux Mint 22.3 VM, one WireGuard configuration, and one AirVPN server path.

It is not proof that:

- every AirVPN server behaves identically;
- AirVPN can never change its policies or configuration;
- AirVPN keeps no server-side logs;
- AirVPN Eddie or Network Lock behaves like the custom nftables configuration tested here;
- a hosting or network provider has no additional visibility;
- the tested AirVPN services reside on the same physical machine or facility;
- IPv6 enforcement has been tested against a native public IPv6 Internet uplink;
- a VPN provides anonymity against every adversary;
- a capable observer with visibility on both sides of the VPN could not attempt traffic correlation.

The paper deliberately distinguishes direct experimental evidence from provider claims and third-party reporting.

---

## Independence

This work is independent.

Kyle Pakhet publishes cybersecurity research under the name Fragmented Sentinel.

This project is not affiliated with, sponsored by, or endorsed by AirVPN, NetworkChuck, tzulo, WireGuard, Zenodo, RTINGS, TorrentFreak, or any other referenced organization.

---

## References

The full reference list is included in the paper.

Core sources include:

- NetworkChuck — video that prompted the investigation  
  https://www.youtube.com/watch?v=axfNxZ1R6C4

- AirVPN  
  https://airvpn.org/

- AirVPN Privacy Notice  
  https://airvpn.org/privacy

- TorrentFreak VPN privacy survey  
  https://torrentfreak.com/best-vpn-anonymous-no-logging/

- RTINGS AirVPN review  
  https://www.rtings.com/vpn/reviews/airvpn/airvpn

- Akamai DNS resolver diagnostic  
  https://www.akamai.com/blog/developers/introducing-new-whoami-tool-dns-resolver-information

- WireGuard Quick Start  
  https://www.wireguard.com/quickstart/

---

## Citation

If you reference this work, please cite:

> Pakhet, K. (2026). *What Does a VPN Actually Hide? Independent Verification of AirVPN, WireGuard, DNS and Kill-Switch Behaviour* (Version 1.1). Zenodo. https://doi.org/10.5281/zenodo.23027447

See [`CITATION.cff`](CITATION.cff) for machine-readable citation metadata.

---

## License

This work is licensed under the Creative Commons Attribution 4.0 International License (CC BY 4.0).

See [`LICENSE`](LICENSE).
