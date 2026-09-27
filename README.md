# homelab-network-infrastructure
Aufbau einer virtuellen Testumgebung (Windows Server 2022, Ubuntu Server, Windows 10) in VirtualBox

# 🚀 Meine IT-Testumgebung: Aufbau einer virtuellen Netzwerk-Infrastruktur

### Das Ziel ist die praxisnahe Vorbereitung auf die Ausbildung zum `Fachinformatik für Systemintegration (FISI)`.

Dieses Projekt zeigt den schrittweisen Aufbau einer isolierten virtuellen Testumgebung auf Basis von **Oracle VirtualBox**. 

---

## 🛜 Netzwerk-Topologie & Komponenten

Die Testumgebung befindet sich in einem isolierten VirtualBox NAT-Netzwerk (`LabNet`).

* **Netzwerk-Segment:** `192.168.10.0/24`
* **Subnetzmaske:** `255.255.255.0`
* **Standardgateway:** `192.168.10.1`

### Virtuelle Maschinen:
| Hostname | Betriebssystem | IP-Adresse | Rolle / Beschreibung |
| :--- | :--- | :--- | :--- |
| **SRV-LNX-01** | Ubuntu Server 24.04 LTS (CLI) | `192.168.10.10` | Linux Core Server |
| **SRV-WIN-01** | Windows Server 2022 Standard | `192.168.10.20` | Domain Controller (geplant) |
| **CLI-WIN-01** | Windows 10/11 Pro | `192.168.10.30` | Domain Client |

---

## 🛠️ Phase 1: Netzwerk-Konfiguration & Basiseinrichtung

1. **VirtualBox NAT-Netzwerk:**
   * Erstellung des benutzerdefinierten NAT-Netzwerks `LabNet` (`192.168.10.0/24`).
   * Deaktivierung des integrierten VirtualBox DHCP-Servers zur Vorbereitung auf einen eigenen DHCP-Server.

2. **Server- & Client-Installation:**
   * Installation von **Ubuntu Server 24.04 LTS** (reine CLI-Installation ohne GUI).
   * Installation von **Windows Server 2022 Standard** (Desktop Experience).
   * Installation von **Windows 10/11** als Client-Betriebssystem.

3. **Netzwerk-Verbindungsprüfung (Troubleshooting):**
   * Konfiguration statischer IPv4-Adressen auf allen Systemen.
   * **Firewall-Anpassung:** Aktivierung der ICMPv4-Inbound-Regel (*Datei- und Druckerfreigabe*) in der Windows Defender Firewall, um ICMP-Echo-Requests (Ping) von Linux zu erlauben.
   * Erfolgreiche Prüfung der Erreichbarkeit aller Systeme untereinander mittels `ping`.

---

## 💡 Troubleshooting & Gelöste Probleme

<img width="1280" height="800" alt="VirtualBox_SRV-LNX-01_27_09_2026_14_58_39" src="https://github.com/user-attachments/assets/1dbddac3-bee2-4fa6-8c1e-112ac56b41dd" />

* **Problem:** Manuelle Behebung von Netzwerkadapterproblemen unter Windows 10 während der Ersteinrichtung (OOBE).
  * **Lösung:** Nutzung des Befehls `OOBE\BYPASSNRO` in der Eingabeaufforderung (`Shift + F10`), um die Ersteinrichtung ohne Microsoft-Konto und aktives DHCP abzuschließen.
* **Problem:** Eingehende Ping-Anfragen an den Windows Server wurden blockiert.
  * **Lösung:** Freischaltung der Regel *Datei- und Druckerfreigabe (Echoanforderung - ICMPv4 eingehend)* in `wf.msc`.

---

## 🎯 Nächste Schritte (Phase 2)
- [ ] Einrichten von **Active Directory Domain Services (AD DS)** auf `SRV-WIN-01`.
- [ ] Anbinden des Clients `CLI-WIN-01` an die Domäne.
- [ ] Konfiguration von **DHCP-** und **DNS-Diensten**.
