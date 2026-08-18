# Troubleshooting the route-based VPN

A tunnel shown as established does not prove that end-to-end traffic is permitted. Verify the packet path in order:

1. Ping each VPC local SRX gateway.
2. Verify WAN reachability between SRX-1 and SRX-2 through the ISP.
3. Check IKEv2 Phase 1: show security ike security-associations.
4. Check IPsec Phase 2: show security ipsec security-associations.
5. Verify the remote LAN route uses st0.0.
6. Generate a VPC-to-VPC ping and inspect IPsec encapsulation and decapsulation counters.
7. Inspect flow sessions and security policy hit counts.

## Confirmed issue: policy order

The IKE/IPsec SAs, tunnel interface, static routes, and IPsec counters were correct, but traffic still failed. The broad default-deny policy was above the specific VPN permit.

On SRX-2, a decrypted 192.168.10.10 to 192.168.20.10 packet enters untrust through st0.0 and leaves toward LAN-B in trust. With this order, it is denied before the permit can be considered:

    untrust -> trust
    1. default-deny       source any, destination any, deny
    2. LAN-A-TO-LAN-B     source LAN-A, destination LAN-B, permit

Correct it with:

    configure
    insert security policies from-zone untrust to-zone trust policy LAN-A-TO-LAN-B before policy default-deny
    commit
    clear security flow session

Make the equivalent correction on SRX-1 for LAN-B-TO-LAN-A in untrust to trust. The supplied configs already use the correct order.

Useful checks:

    show security policies hit-count
    show security flow session destination-prefix 192.168.20.0/24
    show security ipsec statistics
    show log messages | match RT_FLOW

If the specific permit remains at zero hits while default-deny increases, inspect policy order before changing IKE or IPsec settings.
