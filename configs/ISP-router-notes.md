# ISP router configuration

The diagram uses two point-to-point WAN links:

    SRX-1 203.0.113.1/30 --- ISP e0/0 203.0.113.2/30
    ISP e0/1 203.0.114.1/30 --- SRX-2 203.0.114.2/30

For a Cisco IOSv/IOU-style EVE-NG ISP router, load ISP-router.cfg, or paste this configuration:

    enable
    configure terminal
    hostname ISP
    interface Ethernet0/0
     description To-SRX-1-ge-0/0/0
     ip address 203.0.113.2 255.255.255.252
     no shutdown
    interface Ethernet0/1
     description To-SRX-2-ge-0/0/0
     ip address 203.0.114.1 255.255.255.252
     no shutdown
    end
    write memory

No static routes are needed on the ISP: both SRX WAN networks are directly connected. Do not add LAN-A or LAN-B routes to the ISP; each SRX sends the remote LAN prefix to st0.0, where it is encrypted.
