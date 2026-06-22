# 🛠️ PraxisLab-Infrastruktur: Netzwerk- & Virtualisierungsportfolio


Dieses Repository dient als Dokumentation und Portfolio für mein persönliches **Home-Lab**. Hier halte ich praktische Erfahrungen, Fehlerbehebungen (Troubleshooting) und Lösungen in den Bereichen Netzwerkinfrastruktur, Virtualisierung, Systems Administration und Cybersicherheit fest.

Das Hauptziel dieses Projekts ist es, theoretisches Wissen in praxisnahen Szenarien anzuwenden und eine Infrastruktur nach professionellen Industriestandards aufzubauen.

---

## 🖥️ 1. Hardware-Inventar (Hardware Inventory)

Die Home-Lab-Infrastruktur nutzt eine dedizierte Speicher-Klassifizierung (Tiered Storage) sowie leistungsstarke Netzwerk- und Virtualisierungskomponenten, um Ressourcen optimal zu verwalten und Redundanz zu gewährleisten:

*   **Netzwerkinfrastruktur (Network Backbone):**
    *   **Router / Firewall:** MikroTik RB5009UG+S+IN *(1x 10G SFP+, 1x 2.5G, 7x 1G Ports, ARM 4-Kern-Prozessor, RouterOS v7)*
    *   **Managed Switch:** TP-Link TL-SG2008P *(8 Ports Gigabit, L2+ Managed, 4 Ports PoE+ Unterstützung, Omada SDN)*
    *   **Access Point:** TP-Link Omada EAP610 *(Wi-Fi 6 AX1800, PoE+, Multi-SSID & VLAN-fähig)*
    *   **Verkabelung:** Strukturierte Cat 5e Ethernet-Verkabelung
*   **Virtualisierungsserver (Hypervisor):**
    *   **Hauptserver (Mini-PC):** Geekom A8
    *   **Prozessor:** AMD Ryzen 8000er-Serie (Optimiert für hohe Virtualisierungsdichte)
    *   **Arbeitsspeicher:** 32 GB DDR5 RAM
*   **Speicher-Hierarchie (Storage Tiers):**
    *   **Tier 1 (Hot - Intern):** 1 TB Interne NVMe M.2 SSD *(Für das Betriebssystem und aktive, E/A-intensive VMs)*
    *   **Tier 2 (Warm - Extern):** 1 TB Samsung 990 EVO Plus NVMe M.2 SSD *(Angeschlossen über ein externes USB-C-Gehäuse mit 10 Gbps. Genutzt für VM-Backups und Snapshots)*
    *   **Tier 3 (Cold - SD-Karte):** 128 GB SanDisk Extreme PRO MicroSD-Karte *(A2, V30. Genutzt für ISO-Dateien, CT-Templates und Log-Archive)*

---

## 🌐 2. Netzwerkarchitektur und VLAN-Segmentierung

Zur Erhöhung der Netzwerksicherheit und zur Isolierung von Testumgebungen wird eine **VLAN-basierte Netzsegregation** eingesetzt. Alle administrativen Management-Schnittstellen sind vollständig vom Gastnetzwerk und den Laborumgebungen isoliert.

### A. Netzwerktopologie

| Gerät | Funktion | Management-IP | Port-Zuweisung |
| :--- | :--- | :--- | :--- |
| **FritzBox** | WAN / Hausanschluss-Modem | - | Router (ether1) |
| **MikroTik RB5009** | Haupt-Router & Gateway | `10.0.10.1` | ether2 → Switch Port 8 |
| **TP-Link SG2008P** | L2 Managed Switch | `10.0.10.2` | Port 1 → AP, Port 7 → Management-PC, Port 8 → Router |
| **Omada EAP610** | Kabelloser AP | `10.0.10.3` | Port 1 (Trunk) |
| **Geekom A8** | Proxmox VE Host | `10.0.10.10` | Port 6 (Trunk) |
| **Management-PC** | Administrativer Zugriff | `10.0.10.100` (Statisch) | Port 7 (Access VLAN 10) |
| **Smart TV** | Entertainment-Gerät | `10.0.30.33` (Statisch) | Switch Port 5 (Access VLAN 30) |

### B. VLAN-Struktur

| VLAN ID | Name | Subnetz | SSID (Wi-Fi) | Verwendungszweck |
| :--- | :--- | :--- | :--- | :--- |
| **10** | Mgmt | `10.0.10.0/24` | *(Nur Ethernet)* | Infrastruktur-Management (Switch, Router, AP, Proxmox) |
| **21** | WinServer | `10.0.21.0/24` | WinS | Active Directory & Windows Server Testumgebung |
| **22** | LinuxLab | `10.0.22.0/24` | LinS | Linux-Server und Container-Umgebungen |
| **30** | Haus | `10.0.30.0/24` | HLab | Privates Heimnetzwerk (Sichere Endgeräte) |
| **40** | IoT | `10.0.40.0/24` | IoT | Smart-Home-Geräte (Isoliert, eingeschränkter WAN-Zugriff) |
| **50** | Guest | `10.0.50.0/24` | Guest | Isoliertes Gastnetzwerk |
| **60** | Printer | `10.0.60.0/24` | Printer | Netzwerkdrucker (Statische IP-Vergabe) |
| **99** | Kali | `10.0.99.0/24` | KLan | Penetrationstests & Sicherheitsaudits |

---

## 🛠️ 3. Konfigurationsdetails der Netzwerkkomponenten

### Switch (TP-Link SG2008P) Port-Konfiguration
*   **Management-IP:** `10.0.10.2` (Statisch)
*   **Port-Zuordnung:**
    *   **Port 1:** AP (Trunk) -> PVID 1, Tagged: `10,21,22,30,40,50,60,99`, Untagged: `1`
    *   **Port 5:** TV (Access) -> PVID 30, Untagged: `30`
    *   **Port 6:** Proxmox Host (Trunk) -> PVID 10, Tagged: `21,22,99`, Untagged: `10`
    *   **Port 7:** Management-PC (Access) -> PVID 10, Untagged: `10`
    *   **Port 8:** Router Uplink (Trunk) -> PVID 1, Tagged: `10,21,22,30,40,50,60,99`, Untagged: `1`
    *   **Ports 2,3,4:** Unbenutzt (Aus Sicherheitsgründen standardmäßig deaktiviert oder im VLAN 1 isoliert)

### MikroTik RouterOS v7 Konfiguration (Auszug)

*   **Erstellung der VLAN-Schnittstellen:**
    ```routeros
    /interface vlan add interface=bridge name=VLAN10-Mgmt vlan-id=10
    /interface vlan add interface=bridge name=VLAN21-WinServer vlan-id=21
    /interface vlan add interface=bridge name=VLAN22-LinuxLab vlan-id=22
    /interface vlan add interface=bridge name=VLAN30-Haus vlan-id=30
    /interface vlan add interface=bridge name=VLAN40-IoT vlan-id=40
    /interface vlan add interface=bridge name=VLAN50-Guest vlan-id=50
    /interface vlan add interface=bridge name=VLAN60-Printer vlan-id=60
    /interface vlan add interface=bridge name=VLAN99-Kali vlan-id=99
    ```

*   **Zuweisung der IP-Adressen (Gateways):**
    ```routeros
    /ip address add address=10.0.10.1/24 interface=VLAN10-Mgmt
    /ip address add address=10.0.21.1/24 interface=VLAN21-WinServer
    /ip address add address=10.0.22.1/24 interface=VLAN22-LinuxLab
    /ip address add address=10.0.30.1/24 interface=VLAN30-Haus
    /ip address add address=10.0.40.1/24 interface=VLAN40-IoT
    /ip address add address=10.0.50.1/24 interface=VLAN50-Guest
    /ip address add address=10.0.60.1/24 interface=VLAN60-Printer
    /ip address add address=10.0.99.1/24 interface=VLAN99-Kali
    ```

*   **Bridge-VLAN-Einstellungen (Trunking):**
    ```routeros
    /interface bridge vlan add bridge=bridge tagged=bridge,ether2 vlan-ids=10
    /interface bridge vlan add bridge=bridge tagged=ether2 vlan-ids=21,22,30,40,50,60,99
    ```

### Proxmox VE Host Netzwerkkonfiguration (`/etc/network/interfaces`)

Die Netzwerkschnittstelle in Proxmox ist als VLAN-aware Brücke konfiguriert (`bridge-vlan-aware yes`). Dadurch können virtuelle Maschinen (VMs) und Linux-Container (LXCs) direkt mit den entsprechenden VLAN IDs gestartet und isoliert werden.

```bash
auto lo
iface lo inet loopback

auto enp1s0
iface enp1s0 inet manual

auto vmbr0
iface vmbr0 inet static
    address 10.0.10.10/24
    gateway 10.0.10.1
    bridge-ports enp1s0
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes
    bridge-vids 10 21 22 99
```

---

## 🔒 4. Sicherheitsrichtlinie & Datenschutz (Security & Privacy)

In diesem öffentlichen GitHub-Repository gelten strenge Sicherheits- und Datenschutzprinzipien:
1.  **Keine fest kodierten Anmeldedaten (No Hardcoded Secrets):** Zugangsdaten werden über Umgebungsvariablen gesteuert. Lokale Beispiele finden sich in der [`.env.example`](file:///.env.example) Datei.
2.  **Isolierung des Management-Netzwerks:** Der administrative Zugriff auf die Netzwerkkomponenten (`10.0.10.x`) ist physikalisch auf den Switch-Port 7 beschränkt. Ein Zugriff aus WLAN-Netzwerken ist deaktiviert.
3.  **Ausschließen sensibler Daten:** Die [`.gitignore`](file:///.gitignore) verhindert, dass temporäre Dateien, Passwörter, lokale Konfigurationen und private Schlüssel hochgeladen werden.
4.  **Entwicklungskonformität:** Alle Sicherheits- und Arbeitsrichtlinien für dieses Projekt sind in den [Master-Direktiven](file:///MASTER_DIRECTIVES.md) definiert.

---

## 📈 5. Roadmap und zukünftige Projekte

- [ ] Installation eines Kubernetes-Clusters (k3s) im LinuxLab (VLAN 22).
- [ ] Aufbau einer Windows Domain Controller Testumgebung (Active Directory) im WinServer-Netz (VLAN 21).
- [ ] Implementierung eines zentralen Logging-Systems (z.B. Loki/Grafana oder ELK-Stack).
- [ ] Sicherheitsüberprüfungen und Pentesting-Simulationen über das Kali-VLAN (VLAN 99).
