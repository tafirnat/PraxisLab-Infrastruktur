# 🛠️ Professional Home-Lab Portfolio & Network Infrastructure

Bu repository, siber güvenlik, ağ yönetimi, sanallaştırma ve sistem yönetimi alanlarında edindiğim pratik tecrübeleri, karşılaştığım hataları ve ürettiğim çözümleri barındıran profesyonel **Home-Lab günlük ve dökümantasyon alanıdır**.

Buradaki temel amaç, teorik bilgileri gerçek dünya senaryolarıyla birleştirerek endüstri standartlarında altyapı yönetimi becerisi kazanmaktır.

---

## 🖥️ 1. Donanım Envanteri (Hardware Inventory)

Home-Lab altyapısı, kaynak yönetimi ve yedekleme stratejileri gözetilerek 3 katmanlı (tiered storage) depolama ve güçlü sanallaştırma/ağ donanımlarından oluşmaktadır:

*   **Ağ Altyapısı (Network Backbone):**
    *   **Router / Firewall:** MikroTik RB5009UG+S+IN *(1x 10G SFP+, 1x 2.5G, 7x 1G Port, ARM 4 Çekirdek İşlemci, RouterOS v7)*
    *   **Yönetilebilir Switch:** TP-Link TL-SG2008P *(8 Port Gigabit, L2+ Managed, 4 Port PoE+ destekli, Omada SDN)*
    *   **Access Point:** TP-Link Omada EAP610 *(Wi-Fi 6 AX1800, PoE+, Multi-SSID & VLAN)*
    *   **Kablolama:** Cat 5e Ethernet altyapısı
*   **Sanallaştırma Sunucusu (Hypervisor):**
    *   **Ana Sunucu (Mini PC):** Geekom A8
    *   **İşlemci:** AMD Ryzen 8000 Serisi (Yüksek yoğunluklu sanallaştırma desteği)
    *   **RAM:** 32 GB DDR5
*   **Katmanlı Depolama Mimarisi (Storage Tiers):**
    *   **Tier 1 (Hot - Dahili):** 1 TB Dahili NVMe M.2 SSD *(Aktif VM ve konteynerler için yüksek IOPS)*
    *   **Tier 2 (Warm - Harici):** 1 TB Samsung 990 EVO Plus NVMe M.2 SSD *(10 Gbps Type-C harici kutu ile bağlı. VM yedekleri ve anlık görüntüler/snapshot'lar için)*
    *   **Tier 3 (Cold - SD Kart):** 128 GB SanDisk Extreme PRO MicroSD Kart *(A2, V30. ISO dosyaları, CT şablonları ve log arşivleri için)*

---

## 🌐 2. Ağ Yapılandırması ve Topoloji (Network Topology)

Ağ güvenliği ve cihaz izolasyonu için **VLAN (Virtual Local Area Network)** tabanlı segmentasyon uygulanmıştır. Yönetim arayüzleri dış dünyaya ve diğer laboratuvar ortamlarına tamamen kapatılmıştır.

### A. Genel Topoloji

| Cihaz | Görevi | Yönetim IP’si | Bağlantı Noktası (Port) |
| :--- | :--- | :--- | :--- |
| **FritzBox** | WAN / Apartman Modemi | - | Router (ether1) |
| **MikroTik RB5009** | Ana Router & Gateway | `10.0.10.1` | ether2 → Switch Port 8 |
| **TP-Link SG2008P** | L2 Yönetilebilir Switch | `10.0.10.2` | Port 1 → AP, Port 7 → Yönetim PC, Port 8 → Router |
| **Omada EAP610** | Kablosuz AP | `10.0.10.3` | Port 1 (Trunk) |
| **Geekom A8** | Proxmox VE Host | `10.0.10.10` | Port 6 (Trunk) |
| **Yönetim Bilgisayarı**| Yönetim Erişimi | `10.0.10.100` (Statik) | Port 7 (Access VLAN 10) |
| **Smart TV** | Ev Ağı Cihazı | `10.0.30.33` (Statik) | Switch Port 5 (Access VLAN 30) |

### B. VLAN Segmentasyonu

| VLAN ID | İsim | Subnet | SSID (Wi-Fi) | Kullanım Amacı |
| :--- | :--- | :--- | :--- | :--- |
| **10** | Mgmt | `10.0.10.0/24` | *(Sadece Kablolu)* | Altyapı Yönetimi (Switch, Router, AP, Proxmox) |
| **21** | WinServer | `10.0.21.0/24` | WinS | Active Directory ve Windows Server Laboratuvarı |
| **22** | LinuxLab | `10.0.22.0/24` | LinS | Linux Sunucuları ve Konteyner Ortamları |
| **30** | Haus | `10.0.30.0/24` | HLab | Kişisel Cihazlar (Güvenli Ev Ağı) |
| **40** | IoT | `10.0.40.0/24` | IoT | Akıllı Ev Cihazları (İnternet Erişimi Kısıtlı/İzole) |
| **50** | Guest | `10.0.50.0/24` | Guest | Misafir Ağı |
| **60** | Printer | `10.0.60.0/24` | Printer | Yazıcı Ağı (Statik IP Tanımlamaları) |
| **99** | Kali | `10.0.99.0/24` | KLan | Sızma Testleri ve Güvenlik Araştırmaları |

---

## 🛠️ 3. Cihaz Yapılandırma Detayları

### Switch (TP-Link SG2008P) Konfigürasyon Matrisi
*   **Yönetim IP'si:** `10.0.10.2` (Statik)
*   **Port Rolleri:**
    *   **Port 1:** AP (Trunk) -> PVID 1, Tagged: `10,21,22,30,40,50,60,99`, Untagged: `1`
    *   **Port 5:** TV (Access) -> PVID 30, Untagged: `30`
    *   **Port 6:** Proxmox Host (Trunk) -> PVID 10, Tagged: `21,22,99`, Untagged: `10`
    *   **Port 7:** Yönetim PC (Access) -> PVID 10, Untagged: `10`
    *   **Port 8:** Router Uplink (Trunk) -> PVID 1, Tagged: `10,21,22,30,40,50,60,99`, Untagged: `1`
    *   **Port 2,3,4:** Boş (Güvenlik gereği kullanılmayan portlar varsayılan VLAN 1'e veya pasif duruma alınabilir)

### MikroTik RouterOS v7 Yapılandırması (Özet)

*   **VLAN Arayüzlerinin Tanımlanması:**
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

*   **IP Adreslerinin Atanması:**
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

*   **Bridge VLAN Ayarları:**
    ```routeros
    /interface bridge vlan add bridge=bridge tagged=bridge,ether2 vlan-ids=10
    /interface bridge vlan add bridge=bridge tagged=ether2 vlan-ids=21,22,30,40,50,60,99
    ```

### Proxmox VE Host Network Yapılandırması (`/etc/network/interfaces`)

Proxmox üzerinde VLAN farkındalığı (`bridge-vlan-aware yes`) etkinleştirilerek VM'lerin doğrudan ilgili VLAN ID'ler ile çalışabilmesi sağlanmıştır.

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

## 🔒 4. Güvenlik Politikası & Bilgi Güvenliği

Bu projenin halka açık (public) sürümünde siber güvenlik prensipleri en üst düzeyde uygulanmaktadır:
1.  **Sıfır Hardcoded Gizli Veri:** Tüm API anahtarları, şifreler ve tokenlar kod dışında tutulur. Yapılandırma şablonları için [`.env.example`](file:///.env.example) dosyası referans alınmalıdır.
2.  **Yönetim Ağı İzolasyonu:** Yönetim arayüzlerine (`10.0.10.x`) erişim sadece kablolu fiziksel Switch Port 7 üzerinden sınırlandırılmıştır. Wi-Fi ağlarından yönetim subnet'ine erişim engellenmiştir.
3.  **Hassas Dosya Filtreleme:** Yerel geliştirme dosyaları, `.env` dosyaları ve özel sertifikalar git takibine girmeyecek şekilde [`.gitignore`](file:///.gitignore) ile izole edilmiştir.
4.  **Güvenlik Kuralları Bildirgesi:** Yapay zeka asistanları ile uyumlu ve güvenli kod geliştirme kurallarının tamamı [`MASTER_DIRECTIVES.md`](file:///MASTER_DIRECTIVES.md) dosyasında tanımlanmıştır.

---

## 📈 5. Yol Haritası ve Gelecek Çalışmalar

- [ ] Linux Lab (VLAN22) üzerinde Docker & Kubernetes (k3s) Cluster kurulumu.
- [ ] Active Directory Ormanı ve Domain Controller yapılandırılması (VLAN21).
- [ ] Centralized Logging (Elasticsearch/Fluentd/Kibana veya Grafana Loki) entegrasyonu.
- [ ] Güvenlik analitiği için Kali VLAN99 üzerinden kontrollü penetrasyon testlerinin gerçekleştirilmesi.
