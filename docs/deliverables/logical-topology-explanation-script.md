# TAMAC Inc. — Logical Topology Explanation Script

> **Use with:** Packet Tracer → **Logical** view → `Final Proj.pkt`  
> **Audience:** Instructor / capstone defense panel  
> **Estimated time:** 10–12 minutes (full script) · 5–6 minutes (abbreviated — sections marked ⏩)

---

## Before you start

1. Open `Final Proj.pkt` and switch to **Logical** view.
2. Zoom so the full path **ISP → Edge Router → Core → four IDFs** is visible.
3. Have these devices ready to click if asked: Edge Router, CORE-SW, IDF 1-A, DNS-SVR, one staff PC, Guest laptop.

**Opening line:**

> "This is the logical topology for TAMAC Inc., a two-story e-commerce headquarters with roughly 200 to 250 employees. The design follows a classic three-tier model: WAN edge, distribution/core, and access — with four IDF closets serving eight business zones plus a segregated guest network."

---

## 1. Company context and design goals (~1 min)

> "TAMAC is an e-commerce platform aggregator. Our Packet Tracer build models their HQ on two floors.
>
> **Floor 1** holds the lobby and guest area, Executive, Finance, Operations, Customer Service, and utility closets **IDF 1-A** and **IDF 1-B**.
>
> **Floor 2** holds IT — which is also our **MDF** — plus HR, Marketing, and closets **IDF 2-A** and **IDF 2-B**.
>
> The **MDF** in the IT server room is the network hub: it contains only three devices — the **Edge Router**, the **Core Switch**, and our **DNS server**.
>
> Our design goals were department segmentation with VLANs, inter-VLAN routing at the core, centralized DHCP for staff, isolated guest internet-only access, high-availability uplinks with EtherChannel, and security hardening with ACLs, SSH, and port security."

⏩ **Short version:** "Two-floor HQ, MDF on Floor 2, four IDFs, eight departments plus guest — all segmented by VLAN."

---

## 2. WAN edge — ISP and Edge Router (~1 min)

*[Pan to the left / top of logical diagram — ISP-Router and Edge Router]*

> "Internet connectivity enters through an **ISP Router** on a **serial link**, not a generic cloud. We use a Cisco **2911** with an **HWIC-2T** module for `Serial0/0/0`.
>
> The WAN subnet is **10.0.0.0/30**. The ISP is **10.0.0.1**; our Edge Router WAN address is **10.0.0.2**.
>
> The Edge Router is the **security perimeter**. It runs a **default route** to `10.0.0.1` for all internet-bound traffic and performs **NAT/PAT** so internal private addresses can reach the outside world.
>
> Staff and most internal routing happens deeper in the network — at the core — but the edge router has two special jobs: **WAN/NAT** and **guest network termination**."

⏩ **Short version:** "Serial WAN 10.0.0.0/30 to ISP; Edge Router does NAT and default routing."

---

## 3. Edge Router ↔ Core connection (~1 min)

*[Trace the link from Edge Router `Gi0/0` to Core `Fa0/8`]*

> "The Edge Router connects to the Core Switch on **`GigabitEthernet0/0`**, cabled to the core's **`FastEthernet0/8`**.
>
> This link is a **802.1Q trunk** carrying VLANs **90** and **200** — not a simple access port. That matters because guest traffic and management traffic use different VLANs on the same physical cable.
>
> On the router we created **subinterfaces**:
>
> - **`Gi0/0.200`** — management VLAN, IP **192.168.200.14/28**. This is how the core reaches the internet: its default route points here.
> - **`Gi0/0.90`** — guest VLAN, IP **192.168.90.1/25**. This is the **guest default gateway** and where the **GUEST_RESTRICT** ACL is applied.
>
> We deliberately moved guest Layer-3 to the edge router because Packet Tracer's 3560 often won't reliably bind ACLs to SVIs — but functionally this also matches real-world practice: guest traffic is isolated at the perimeter."

⏩ **Short version:** "Router Gi0/0 trunks VLANs 90 and 200 to core Fa0/8; subinterfaces .90 for guest, .200 for management/internet path."

---

## 4. Core switch — the network brain (~2 min)

*[Click CORE-SW — mention it's a Cisco 3560-24PS]*

> "The **Core Switch** is a **Cisco 3560-24PS**. It is the **Layer-3 center** of the LAN.
>
> **`ip routing`** is enabled. For each staff department VLAN we created an **SVI** — a switched virtual interface — that acts as the **default gateway** for that subnet. For example, VLAN 10 Executive uses **`192.168.10.1`**, VLAN 20 Finance uses **`192.168.20.1`**, and so on.
>
> The core also hosts **DHCP pools** for all **staff** VLANs. Every pool points clients to our internal DNS at **`192.168.100.2`**. Guest DHCP is **not** on the core — it runs on the edge router's guest subinterface.
>
> **Vlan90 on the core is shut down** on purpose. Guest Layer-3 lives only on the router. The guest VLAN still exists as a Layer-2 broadcast domain through the IDFs, but its gateway is **`192.168.90.1`** on the router, not the core.
>
> The core's own management address is **`192.168.200.1`** on **Vlan200**. All four IDF switches also sit in VLAN 200 at **.2 through .5**.
>
> For internet access, the core has:
>
> ```
> ip route 0.0.0.0 0.0.0.0 192.168.200.14
> ```
>
> That sends unknown traffic to the Edge Router on the management subinterface, which NATs it out the serial link.
>
> The core is also the **STP root bridge** running **Rapid PVST+**, so spanning tree converges quickly if a link fails."

⏩ **Short version:** "Core does inter-VLAN routing, staff DHCP, default route to router .14, STP root; Vlan90 SVI is off."

---

## 5. DNS and server VLAN (~30 sec)

*[Point to DNS-SVR connected to Core Fa0/1]*

> "Our **DNS server** sits on **VLAN 100**, subnet **192.168.100.0/28**, static IP **`192.168.100.2`**, cabled to core **`Fa0/1`** as an access port in VLAN 100.
>
> It serves internal names like **`tamac.local`** and **`dns.tamac.local`**. A **SERVER_ACCESS** ACL on **Vlan100** permits only authorized department subnets to reach the server zone — everyone else is denied inbound to that subnet."

---

## 6. Core to IDFs — EtherChannel uplinks (~1 min)

*[Trace Po1–Po4 from core to each IDF]*

> "From the core, **four EtherChannels** — **Po1 through Po4** — connect to our four IDF access switches. Each IDF has **two physical uplinks** bundled with **LACP** for redundancy.
>
> | IDF | EtherChannel | Core ports | IDF ports |
> |-----|--------------|------------|-----------|
> | 1-A | Po1 | Gi0/1–2 | Fa0/23–24 |
> | 1-B | Po2 | Fa0/2–3 | Fa0/23–24 |
> | 2-A | Po3 | Fa0/4–5 | Fa0/23–24 |
> | 2-B | Po4 | Fa0/6–7 | Fa0/23–24 |
>
> All uplinks are **trunks** carrying every active VLAN. If one cable is unplugged, the port-channel stays up and traffic continues — that's our **high-availability** story.
>
> Note: the 3560-24PS only has **two Gigabit ports**, so Po1 uses Gi0/1–2, but Po2–Po4 use FastEthernet ports on the core. That's a real hardware constraint we designed around."

⏩ **Short version:** "Dual uplinks per IDF, LACP EtherChannel, trunks — unplug one link, network stays up."

---

## 7. VLAN and IP plan — the logical segmentation (~1.5 min)

*[Optional: open VLAN table or refer to deliverable 05]*

> "We segment the company into **eleven VLANs**, but only **eight are departments** plus guest, servers, and management. There is **no Procurement department** — VLAN 60 does not exist in our build.
>
> | VLAN | Name | Subnet | Gateway | DHCP |
> |------|------|--------|---------|------|
> | 10 | Executive | 192.168.10.0/26 | .10.1 | Core |
> | 20 | Finance | 192.168.20.0/26 | .20.1 | Core |
> | 30 | Operations | 192.168.30.0/25 | .30.1 | Core |
> | 40 | Customer Service | 192.168.40.0/25 | .40.1 | Core |
> | 50 | IT | 192.168.50.0/26 | .50.1 | Core |
> | 70 | HR | 192.168.70.0/26 | .70.1 | Core |
> | 80 | Marketing | 192.168.80.0/26 | .80.1 | Core |
> | 90 | Guest | 192.168.90.0/25 | **.90.1 on router** | **Router** |
> | 100 | Servers | 192.168.100.0/28 | .100.1 | Static only |
> | 200 | Management | 192.168.200.0/28 | .200.1 | Static only |
> | 999 | Unused | — | — | Ports shutdown |
>
> Subnet sizing reflects department size: Operations and Customer Service use **/25** for more hosts; executive and finance use **/26**. Everything is carved from **`192.168.0.0/16`** with **static routing** — no dynamic routing protocol inside the LAN."

⏩ **Short version:** "Eight dept VLANs + guest + servers + mgmt; guest gateway and DHCP on router; no VLAN 60."

---

## 8. IDF access layer — where users connect (~2 min)

*[Walk each IDF in logical view]*

### IDF 1-A (Floor 1)

> "**IDF 1-A** serves **Executive** and **Finance** wired PCs, plus three access points:
>
> - **Fa0/1–8** — Executive PCs, VLAN 10
> - **Fa0/9–16** — Finance PCs, VLAN 20
> - **Fa0/17** — **Guest AP** in the lobby, VLAN 90, SSID **`TAMAC-Guest`**
> - **Fa0/18** — Executive AP, VLAN 10, SSID **`TAMAC-Corp-exec`**
> - **Fa0/19** — Finance AP, VLAN 20, SSID **`TAMAC-Corp-fin`**
>
> Uplinks **Fa0/23–24** trunk to core Po1."

### IDF 1-B (Floor 1)

> "**IDF 1-B** serves **Customer Service** and **Operations**:
>
> - **Fa0/1–14** and **Fa0/21** — Customer Service, VLAN 40
> - **Fa0/15–20** and **Fa0/22** — Operations, VLAN 30
> - AP-CS on **Fa0/21**, AP-OPS on **Fa0/22** with corporate SSIDs for each department."

### IDF 2-A (Floor 2)

> "**IDF 2-A** serves **IT** and **HR**:
>
> - **Fa0/1–8** and **Fa0/20** — IT PCs, VLAN 50
> - **Fa0/15–19** and **Fa0/21** — HR, VLAN 70
> - **Fa0/9–14** are intentionally **shutdown on VLAN 999** — reserved unused ports, not Procurement
> - AP-IT on Fa0/20, AP-HR on Fa0/21."

### IDF 2-B (Floor 2)

> "**IDF 2-B** serves **Marketing** and HR training space:
>
> - **Fa0/1–12** and **Fa0/16** — Marketing, VLAN 80
> - **Fa0/13–15** and **Fa0/17** — HR training, VLAN 70
> - AP-MARK on Fa0/16."

> "Every IDF port is an **access port** in exactly one VLAN — except the two uplinks, which are trunks. End devices use **DHCP**; we placed two to three representative PCs per department to model a larger office without simulating 200 individual machines."

---

## 9. Wireless logical topology (~1 min)

*[Point to AP icons or Config → Port 1 wireless]*

> "Wireless is **not a separate network** — it extends the same VLANs as wired users.
>
> **Corporate staff** connect to department-specific SSIDs: **`TAMAC-Corp-exec`**, **`TAMAC-Corp-fin`**, and so on. Each AP is an access port in its department VLAN, so a Finance laptop on **`TAMAC-Corp-fin`** gets a **192.168.20.x** address from the core DHCP pool — same as a wired Finance PC.
>
> **Guests** connect only to **`TAMAC-Guest`** on the lobby AP. They receive **192.168.90.x** from the **router's GUEST_POOL**, DNS **8.8.8.8**, and gateway **192.168.90.1**.
>
> APs cable **Port 0** to the switch; wireless is configured on **Port 1** in the GUI. Because many laptops share one AP uplink, AP switch ports use **higher port-security limits** — 50 for guest, 10 for staff APs — instead of the single-MAC limit we use on wired PC ports."

---

## 10. Traffic flows — how packets move (~1.5 min)

### Staff PC → another department

> "When **PC-IT** pings **PC-FIN**, the packet stays in VLAN 50 until it hits the core SVI at **192.168.50.1**. The core routes to **192.168.20.0/26** out the VLAN 20 interface. Return traffic follows the reverse path. No internet involvement — pure **inter-VLAN routing** on the core."

### Staff PC → DNS

> "When any staff device resolves **`dns.tamac.local`**, it queries **192.168.100.2**. The packet routes to VLAN 100. **SERVER_ACCESS** on the core permits that department's subnet to reach the server VLAN."

### Staff PC → internet

> "Internet-bound traffic goes to the department gateway on the core, then follows the core **default route** to **192.168.200.14** on the Edge Router, gets **NATed**, and exits **10.0.0.2** to the ISP."

### Guest laptop → internal (blocked)

> "Guest traffic in VLAN 90 goes through the IDF trunk to the core, then to the router's **`Gi0/0.90`** subinterface. **GUEST_RESTRICT** **denies** all traffic from **192.168.90.0/25** to internal **192.168.x** subnets — including DNS at **192.168.100.2** and Executive at **192.168.10.1**. Permitted traffic can still reach the internet via NAT."

*[Optional demo commands]*

```
! Guest laptop — should FAIL:
ping 192.168.10.1
ping 192.168.100.2

! Guest laptop — should SUCCEED:
ping 10.0.0.1
```

---

## 11. Security overlay on the topology (~1 min)

> "Security is woven through the same logical paths:
>
> - **GUEST_RESTRICT ACL** — Edge Router `Gi0/0.90`, inbound: guest isolation
> - **SERVER_ACCESS ACL** — Core `Vlan100`: only approved dept subnets reach DNS
> - **SSH v2 only** — All infrastructure devices manageable on **VLAN 200**; telnet disabled
> - **Port security** — Wired PC ports: max **1** MAC, sticky, shutdown on violation. *Live demo:* unplug staff PC, plug cable into unconnected **HACK-** PC → port err-disables (red); reconnect original PC + `shutdown`/`no shutdown` → green. See [log 11](../../logs/11-port-security-hacker-pc-demo.md).
> - **AP port security** — Higher limits so Wi-Fi clients don't err-disable the uplink
> - **DHCP snooping** — Configured on core and IDFs with trusted uplinks only; in Packet Tracer we may disable it at runtime so DHCP works reliably during demos
> - **VLAN 999** — Unused ports administratively shutdown so they can't be hijacked"

---

## 12. Closing summary (~30 sec)

> "To summarize the logical topology:
>
> 1. **ISP** connects by serial to the **Edge Router** for WAN and NAT.
> 2. The **Edge Router** trunks to the **Core** for management and guest termination.
> 3. The **Core** provides **inter-VLAN routing**, **staff DHCP**, **DNS access control**, and **four redundant EtherChannel uplinks** to IDFs.
> 4. **Four IDF switches** distribute **eight department VLANs** plus guest to wired PCs and access points across two floors.
> 5. **Guest** is logically isolated at the router — internet yes, internal no.
> 6. **Corporate Wi-Fi** maps one SSID per department onto the same VLANs as wired users.
>
> The result is a segmented, defensible enterprise LAN that models TAMAC's 200-plus employee headquarters with high availability and role-based access — all verifiable in Packet Tracer from any staff PC, guest laptop, or SSH session on the management VLAN."

---

## Q&A cheat sheet (quick answers)

| Question | Answer |
|----------|--------|
| Why is guest on the router, not the core? | PT ACL binding on SVIs is unreliable; router subinterface is verified and is a valid perimeter design |
| Why 192.168.200.14 and not .254? | /28 only allows host addresses .1–.14; .254 is out of range |
| Why no VLAN 60? | TAMAC has no Procurement department — eight depts only |
| How does DHCP work without `ip helper-address`? | Core SVIs are L3 gateways on the same broadcast domain as each VLAN — DHCP is local to the core |
| What proves guest isolation? | Guest ping to 192.168.10.1 fails; ping to 10.0.0.1 succeeds |
| What proves HA? | `show etherchannel summary` — all Po1–4 `(SU)`; unplug one uplink, pings continue |
| Where is the STP root? | CORE-SW — `spanning-tree vlan … root primary` |

---

## Abbreviated 5-minute path

1. Opening + company context (30 sec)  
2. ISP → Edge Router → Core trunk (1 min)  
3. Core SVIs + DHCP + default route (1 min)  
4. EtherChannel to four IDFs (45 sec)  
5. VLAN table highlights — guest on router (45 sec)  
6. One IDF walkthrough + Wi-Fi SSIDs (45 sec)  
7. Guest vs staff traffic flow + close (30 sec)

---

*Script aligned with built topology in `docs/PROGRESS.md` and session logs. If design docs disagree, trust PROGRESS and logs.*
