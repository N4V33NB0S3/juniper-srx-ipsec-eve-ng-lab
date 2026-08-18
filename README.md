# Juniper SRX Route-Based IPsec Site-to-Site VPN (EVE-NG)

An EVE-NG lab connecting two LANs through an ISP router using an IKEv2, route-based IPsec VPN between Juniper SRX firewalls.

> Security note: No real pre-shared key, password, public IP address, or device ID belongs in this repository. Replace all <placeholder> values locally before loading a config.

## Topology

    VPC1 (192.168.10.10/24)                 VPC2 (192.168.20.10/24)
              |                                        |
       SRX-1 trust                              trust SRX-2
    LAN-A 192.168.10.0/24                   LAN-B 192.168.20.0/24
              |                                        |
       SRX-1 untrust --- ISP / routed WAN --- untrust SRX-2
                       IKEv2/IPsec over st0.0

The ISP router provides only underlay reachability; it does not terminate the VPN.

## Addressing

| Component | Address / network |
| --- | --- |
| VPC1 / SRX-1 gateway | 192.168.10.10/24 / 192.168.10.1 |
| LAN-A | 192.168.10.0/24 |
| VPC2 / SRX-2 gateway | 192.168.20.10/24 / 192.168.20.1 |
| LAN-B | 192.168.20.0/24 |
| WAN | Replace <SRX1_WAN_IP>, <SRX2_WAN_IP>, and next-hop placeholders |
| PSK | Replace <REPLACE_WITH_STRONG_PSK> locally on both SRXs |

## Contents

    configs/SRX-1.set          LAN-A firewall baseline
    configs/SRX-2.set          LAN-B firewall baseline
    configs/ISP-router-notes.md  Underlay routing notes
    docs/troubleshooting.md    Packet-path checks and the policy-order lesson

## Deploy

1. Build two SRXs, one routed ISP node, and two VPCs in EVE-NG.
2. Connect VPC1 to SRX-1 ge-0/0/1; connect both SRX ge-0/0/0 interfaces to the ISP; connect VPC2 to SRX-2 ge-0/0/1. Update the config if your interface names differ.
3. Set VPC1 to 192.168.10.10/24 with gateway 192.168.10.1. Set VPC2 to 192.168.20.10/24 with gateway 192.168.20.1.
4. Edit each matching set-format config, replacing every placeholder and the PSK.
5. Add the ISP underlay routes described in configs/ISP-router-notes.md.
6. On each SRX: configure; load set terminal; paste the edited config; commit check; commit.
7. Test a ping from VPC1 to 192.168.20.10 and the reverse from VPC2.

## Validation

    show security ike security-associations
    show security ipsec security-associations
    show security ipsec statistics
    show route 192.168.20.0/24
    show security flow session
    show security policies hit-count

Expected: IKE and IPsec SAs are established, the remote LAN route uses st0.0, IPsec counters increase, and the specific permit policies receive hits.

## Critical lesson: policy order

Junos evaluates security policies from top to bottom. A broad default-deny before a specific VPN permit captures and denies the packet before the permit is evaluated.

    Incorrect: default-deny -> LAN-A-TO-LAN-B
    Correct:   LAN-A-TO-LAN-B -> default-deny

The provided configs already place specific permits first. See docs/troubleshooting.md for the confirmed troubleshooting outcome.

## Push to GitHub

Create a new empty GitHub repository, then run this from this folder:

    git init
    git add .
    git status
    git commit -m "Add Juniper SRX IPsec EVE-NG lab"
    git branch -M main
    git remote add origin https://github.com/<YOUR-USERNAME>/juniper-srx-ipsec-eve-ng-lab.git
    git push -u origin main

Before the first push, inspect git status and search staged files for the real PSK. Keep device backups and EVE-NG exports outside Git.

## License

MIT. See LICENSE.
