# DHCP Pass Rule

Firewall rule that lets DHCP discovery/request broadcasts through, so clients
without an IP address yet can obtain one.

| Field            | Value                         |
|------------------|-------------------------------|
| Action           | Pass / Allow / Accept         |
| IP Version       | IPv4 (DHCPv4 is IPv4-only)    |
| Source IP        | `0.0.0.0`                     |
| Destination IP   | `255.255.255.255`             |
| Protocol / Port  | Any (optionally UDP 67-68)    |
| Status           | Enabled                       |

Save / Apply, then reboot the router or renew DHCP leases on clients if the
change does not take effect immediately.

> Tip: if your router allows it, restricting the rule to **UDP, ports 67-68**
> covers DHCP fully while allowing nothing else to/from the broadcast address.
> "Both" IP versions is harmless but unnecessary — DHCPv6 uses multicast
> (`ff02::1:2`), not `255.255.255.255`.

## Web UI (generic)

1. Firewall → Rules (or Security → Access Control / Traffic Rules).
2. Add / New rule, fill in the table above.
3. Place it **above** any block/deny rule on the LAN interface.
4. Save, then Apply Changes.

## Command-line equivalents

### pfSense / OPNsense
Firewall → Rules → **LAN** → Add: Action *Pass*, Interface *LAN*, Address
Family *IPv4*, Protocol *any* (or UDP), Source *Single host* `0.0.0.0`,
Destination *Single host* `255.255.255.255` → Save → Apply Changes.

### OpenWrt (uci)
```sh
uci add firewall rule
uci set firewall.@rule[-1].name='Allow-DHCP-Broadcast'
uci set firewall.@rule[-1].src='lan'
uci set firewall.@rule[-1].family='ipv4'
uci set firewall.@rule[-1].src_ip='0.0.0.0'
uci set firewall.@rule[-1].dest_ip='255.255.255.255'
uci set firewall.@rule[-1].target='ACCEPT'
uci set firewall.@rule[-1].enabled='1'
uci commit firewall
/etc/init.d/firewall restart
```

### iptables (Linux router)
```sh
iptables -I INPUT -s 0.0.0.0 -d 255.255.255.255 -j ACCEPT
```

### nftables
```sh
nft insert rule inet filter input ip saddr 0.0.0.0 ip daddr 255.255.255.255 accept
```
