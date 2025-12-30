# OWNIT IPTV over Wi-Fi using OpenWrt (No ISP Router)

This document describes how to run OWNIT IPTV **directly over Wi-Fi**
without using the ISP-provided router.

In this setup, OpenWrt is used as the router firmware. Other advanced
firewall/router platforms such as OPNsense or pfSense should also work,
but most stock consumer router firmware will not, due to missing support
for multicast, IGMP, and advanced bridge configuration.

## Background

The setup commonly described in forums is:

![Fiber → ISP router (bridge mode) → Custom router → Ethernet → TV](/images/graph02.png)

This approach avoids multicast complexity by letting the ISP router
handle VLAN tagging and IGMP, but it requires keeping the ISP router
in the network path and typically assumes a wired Ethernet connection
to the TV boxes with VLAN 501 for the Ethernet ports used on the router.

In my case, this approach was not viable:

- The ISP-provided router (**Icotera i4882**) did not function properly or reliably.
- Running Ethernet to the TVs was not possible due to physical layout.
- I wanted a **single-router architecture**, avoiding any dependency on ISP firmware or additional intermediary
  devices, both for reliability and maintainability.

The goal was therefore:

![Fiber → OpenWrt router → Wi-Fi → TV boxes](/images/graph01.png)

No official documentation from OWNIT describes this setup, and I could not
find any complete or working examples of running OWNIT IPTV directly over
Wi-Fi using a third-party router without the ISP-provided router.
All available references assumed an Ethernet connection and the presence
of the ISP router.

## Hardware
- ISP: OWNIT 
- Router: ASUS TUF AX6000
- Firmware: OpenWrt (24.10.5)
- TV boxes: OWNIT IPTV (Android-based)

## Key Concepts

- OWNIT uses IPv4 multicast (**IGMPv2**)
- IPTV traffic arrives on **VLAN 501** on the WAN side
- VLAN 501 terminates at the router and is not extended over Wi-Fi
- Multicast over Wi-Fi requires:
  - **IGMP snooping** on the **Wi-Fi bridge**
  - An active multicast querier
  - Correct IGMP version handling (**IGMPv2**)

## Required Packages

- `omcproxy`
- `luci-app-omcproxy` (optional, depending on layout)


![software/packages](/images/software.png)


## Configuration Summary

### Interfaces
- `iptv`: unmanaged interface

  
![iptv](/images/interfaces-iptv1.png)

- `lan_clients`: bridge containing Wi-Fi interfaces


![bridge](/images/bridge_clients-wifi01.png)

### Bridge Settings (`lan_clients`)
- IGMP snooping: **enabled**
- Multicast querier: **enabled**
- Force IGMP version: **v2**


![IGMP Snooping Bridge Clients](/images/bridge_clients-wifi02.png)
![IGMP Version Bridge Clients](/images/bridge_clients-wifi03.png)


### omcproxy
- Uplink interface: `iptv`
- Downlink interface: `lan_clients`


![omcproxy](/images/omcproxy.png)

### VLAN

![VLAN](/images/VLAN.png)


### WIFI

![WIFI01](/images/WIFI01.png)

![WIFI02](/images/WIFI02.png)

**Multi To Unicast - OFF**


## Common Issues

- **Audio only, no picture**  
  → IGMP version mismatch (must be IGMPv2)
  
- **Choppy playback**  
  → Wi-Fi airtime limitations or multicast handling on the AP

- **No picture at all**  
  → Missing multicast querier on the Wi-Fi bridge

## References

The following forum threads and discussions were helpful during
investigation and troubleshooting, although none described a complete
or working Wi-Fi-based setup without the ISP router present:

- https://www.sweclockers.com/forum/trad/1601107-vlan-med-ownit-tv
- https://www.sweclockers.com/forum/trad/1738761-iptv-fran-ownit-fungerar-inte
- https://www.sweclockers.com/forum/post/15278839
- https://www.sweclockers.com/forum/trad/845719-telia-fiberlan-vlan-id-for-olika-tjanster

## Conclusion

This setup allows TV to function reliably over Wi-Fi using
a single OpenWrt router, completely removing the ISP router from
the network path.

While this configuration requires careful multicast handling,
it provides full control over the network and avoids dependency
on ISP-provided firmware.
