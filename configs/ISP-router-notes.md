# ISP router notes

The ISP router is only the underlay transit device. Give each WAN link a routed subnet and ensure that the peer SRX WAN address is reachable through the ISP.

    SRX-1 WAN --- 10.0.12.0/30 --- ISP --- 10.0.23.0/30 --- SRX-2 WAN

The supplied SRX configs use a /32 peer-WAN static route through the local ISP next hop. If the SRXs have a default route toward the ISP, the /32 route is optional.

Do not route LAN-A or LAN-B through the ISP. Each SRX routes the remote LAN prefix to st0.0 and encrypts that traffic.
