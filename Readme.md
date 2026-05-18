
# Extended Access Control List (ACL) Lab – Cisco Packet Tracer

## 📌 Lab Objective 
Implement an **Extended ACL** to block specific traffic (HTTP, HTTPS, ICMP, DNS) from selected hosts and networks to a server, while permitting all other traffic.

---

## 🌐 Network Addressing

| Network          | Subnet Mask          | Devices / Role                          |
|------------------|----------------------|------------------------------------------|
| `40.0.0.0`       | `/8` (255.0.0.0)     | Left-side hosts (Laptop0, PC4, PC5, PC6) |
| `128.20.0.0`     | `/16` (255.255.0.0)  | Backbone (green area, inter-router links)|
| `192.168.10.0`   | `/24` (255.255.255.0)| Bottom hosts (Laptop1, PC7)              |
| `50.50.0.0`      | `/16` (255.255.0.0)  | Server network (Server0 at `50.50.0.2`)  |

---

## 📋 Filtering Requirements

| # | Source                          | Destination     | Protocol / Service      | Action     |
|---|--------------------------------|----------------|-------------------------|------------|
| 1 | Host `40.0.0.2`                | `50.50.0.2`    | HTTP (80) + HTTPS (443) | **Deny**   |
| 2 | Host `40.0.0.1`                | `50.50.0.2`    | ICMP (ping/echo)        | **Deny**   |
| 3 | Host `192.168.10.1`            | `50.50.0.2`    | DNS (UDP/TCP port 53)   | **Deny**   |
| 4 | Network `192.168.10.0/24`      | `50.50.0.2`    | HTTP (port 80)          | **Deny**   |
| – | All other traffic              | Any            | Any                     | **Permit** |

---

## ⚙️ Configuration (Cisco IOS)

> **Best practice:** Place Extended ACLs as close as possible to the **source** network.

### 🔹 Router3 (connected to `40.0.0.0/8`)

**Interface:** `FastEthernet 0/0` – **inbound** direction

configure terminal
access-list 100 deny tcp host 40.0.0.2 host 50.50.0.2 eq 80
access-list 100 deny tcp host 40.0.0.2 host 50.50.0.2 eq 443
access-list 100 deny icmp host 40.0.0.1 host 50.50.0.2 echo
access-list 100 permit ip any any
interface fastEthernet 0/0
ip access-group 100 in
end
write memory
```

🔹 Router5 (connected to 192.168.10.0/24)

Interface: FastEthernet 0/1 – inbound direction


configure terminal
access-list 101 deny udp host 192.168.10.1 host 50.50.0.2 eq 53
access-list 101 deny tcp host 192.168.10.1 host 50.50.0.2 eq 53
access-list 101 deny tcp 192.168.10.0 0.0.0.255 host 50.50.0.2 eq 80
access-list 101 permit ip any any
interface fastEthernet 0/1
ip access-group 101 in
end
write memory
```

---

✅ Verification

Test Expected Result
40.0.0.2 → browse http://50.50.0.2 ❌ Blocked (no HTTP/HTTPS)
40.0.0.1 → ping 50.50.0.2 ❌ Blocked (timeout)
192.168.10.1 → nslookup to 50.50.0.2 ❌ Blocked (DNS refused)
Any host in 192.168.10.0/24 → HTTP to 50.50.0.2 ❌ Blocked
40.0.0.3 → ping 50.50.0.2 ✅ Permitted
192.168.10.2 → any non-HTTP (e.g., SSH) ✅ Permitted

Show commands:

```cisco
show access-lists
show ip interface fastEthernet 0/0   (on Router3)
show ip interface fastEthernet 0/1   (on Router5)
```


📝 Notes

· Extended ACLs use numbers 100–199 (or 2000–2699 for named extended).
· Wildcard mask 0.0.0.255 = subnet mask 255.255.255.0.
· The permit ip any any at the end prevents the implicit deny from blocking all other traffic.

Eng-Hani Ahmed Abdullah Muhammed.
