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

## 🛠️ Phase 2: Active Directory DS, DNS, DHCP & Domain Join

Im Rahmen der zweiten Phase wurde der Server `SRV-WIN-01` als Domänencontroller (Domain Controller) eingerichtet sowie grundlegende Netzwerkdienste zur Automatisierung bereitgestellt.

### 1. Active Directory Domain Services (AD DS) & DNS
* Installation der Rollen **AD DS** und **DNS-Server** auf `SRV-WIN-01`.
* Heraufstufung (Promoten) des Servers zum ersten Domänencontroller in einer neuen Gesamtstruktur (Forest): **`lab.local`**.
* Konfiguration des lokalen DNS-Servers (`127.0.0.1` / `192.168.10.20`) für eine korrekte Namensauflösung.

### 2. DHCP-Server Konfiguration
* Installation und Autorisierung der Rolle **DHCP-Server** im Active Directory.
* Erstellung und Aktivierung des IP-Bereichs (Scope) `LabNet Scope`:
  * **Bereich:** `192.168.10.100` – `192.168.10.200` /24
  * **Gateway:** `192.168.10.1`
  * **DNS-Server:** `192.168.10.20` (`lab.local`)
* Erfolgreiche Prüfung der automatischen IP-Adressvergabe auf dem Client `CLI-WIN-01` mittels `ipconfig /all`.

<img width="1024" height="768" alt="VirtualBox_CLI-WIN-01_27_09_2026_21_59_22" src="https://github.com/user-attachments/assets/310e1400-9f21-470e-b2c0-f6a36fc8f127" />


### 3. Domain Join & Benutzerverwaltung
* Erfolgreiche Domänenanbindung (Domain Join) der Client-Maschine `CLI-WIN-01` an die Domäne `lab.local`.
* Erstellung eines Domänen-Benutzers **Max Mustermann** (`mmustermann`) in Active Directory-Benutzer und -Computer (ADUC).
* Erstmalige Anmeldung am Client-System mit dem neu erstellten Domänen-Konto.

---

## 🎯 Nächste Schritte (Phase 3)
- [ ] Integration des Linux-Servers (`SRV-LNX-01`) in die Domäneninfrastruktur oder Konfiguration von SSH/RSYNC.
- [ ] Erstellung von Gruppenrichtlinien (**Group Policy Objects / GPO**) im Active Directory.
- [ ] Einrichten von Netzwerkfreigaben (SMB / Dateiserver) mit Zugriffssteuerung.

## 🛠️ Phase 3: Linux-Integration & Fernverwaltung (SSH)

1. **Netzwerk- & DNS-Konfiguration (Ubuntu Server):**
   * Anpassung der Netplan-Konfiguration (`/etc/netplan/00-installer-config.yaml`) auf `SRV-LNX-01` mit statischer IP-Adresse (`192.168.10.10/24`) und Zuweisung des Domänen-DNS-Servers (`192.168.10.20`).
   * Erstellung eines **A-Records** und **PTR-Records** in der Forward-Lookupzone der Windows-DNS-Verwaltung für den Host `srv-lnx-01.lab.local`.

<img width="1280" height="800" alt="SSH,LinuxVirtualBox_SRV-LNX-01_28_09_2026_00_13_12" src="https://github.com/user-attachments/assets/d8aa33be-0c73-405e-ac7f-db33ba83bfaa" />

2. **SSH-Fernverwaltung:**
   * Aktivierung und Start des OpenSSH-Dienstes (`openssh-server`) auf Ubuntu.
   * Erfolgreiche Fernverbindung via SSH über die Windows PowerShell von `CLI-WIN-01` aus (`ssh user@srv-lnx-01.lab.local`).
<img width="1024" height="768" alt="Verbindung Client Linux SSH_CLI-WIN-01_28_09_2026_00_14_01" src="https://github.com/user-attachments/assets/819d6e29-d69d-4449-861f-5ca25c278d9d" />

## 🛠️ Phase 3.2: Gruppenrichtlinien (GPO) & Network Drive Mapping

1. **Active Directory Strukturierung (OU):**
   * Erstellung der Organisationseinheit (OU) **`Lab-OU`** im Domänen-Stamm `lab.local`.
   * Verschieben des Domänen-Benutzers **Max Mustermann** (`mmustermann`) in die neue OU zur gezielten Zuweisung von Gruppenrichtlinien.

2. **Dateiserver & Netzwerktrennzeichen (SMB Share):**
   * Erstellung und Freigabe des lokalen Ordners `C:\CompanyShare` als сетевой ресурс `\\srv-win-01.lab.local\CompanyShare`.
   * Konfiguration der NTFS- und Freigabeberechtigungen für Domänen-Benutzer.

3. **GPO-Konfiguration & Laufwerkszuordnung:**
   * Erstellung des Gruppenrichtlinienobjekts (GPO) **`GPO_MapNetworkDrive`** und Verknüpfung mit der `Lab-OU`.
   * Konfiguration der Laufwerkszuordnung unter *Benutzerkonfiguration -> Einstellungen -> Windows-Einstellungen -> Laufwerkszuordnungen*:
     * **Aktion:** Aktualisieren / Ersetzen
     * **Pfad:** `\\srv-win-01.lab.local\CompanyShare`
     * **Laufwerksbuchstabe:** `Z:`
     * **Option:** *Im Sicherheitskontext des angemeldeten Benutzers ausführen*.
   * Erfolgreiche Validierung auf `CLI-WIN-01` via `gpupdate /force` und automatischer Einbindung des Netzwerklaufwerks `Z:`.
   <img width="1024" height="768" alt="DISK Z_CLI-WIN-01_28_09_2026_11_00_56" src="https://github.com/user-attachments/assets/e3789527-5aad-4d79-af72-2881507e904d" />

## 🛠️ Phase 3.3: Zugriffssteuerung & NTFS-Berechtigungen (SMB / Dateiserver)

1. **Active Directory Sicherheitsgruppen (AGDLP-Prinzip):**
   * Erstellung der globalen Sicherheitsgruppen **`GRP_IT_Users`** und **`GRP_HR_Users`** in der `Lab-OU`.
   * Zuweisung des Domänen-Benutzers **Max Mustermann** (`mmustermann`) zur Gruppe `GRP_IT_Users`.

2. **Ordnerstruktur & SMB-Freigabe:**
   * Erstellung der Ordnerstruktur `C:\CompanyData` mit den Unterordnern `IT` und `HR` auf `SRV-WIN-01`.
   * Freigabe des Hauptordners als SMB-Share **`CompanyData`** (`\\srv-win-01.lab.local\CompanyData`) mit Vollzugriff auf Freigabe-Ebene für authentifizierte Benutzer.

3. **NTFS-Berechtigungen & Vererbung:**
   * Deaktivierung der Vererbung auf den Unterordnern `IT` und `HR`.
   * Konfiguration der NTFS-Zugriffssteuerungslisten (ACLs):
     * Ordner `IT`: Vollzugriff/Ändern ausschließlich für **`GRP_IT_Users`**.
     * Ordner `HR`: Vollzugriff/Ändern ausschließlich für **`GRP_HR_Users`**.

4. **Validierung & Funktionstest:**
   * Erfolgreicher Zugriff auf den Ordner `\\srv-win-01.lab.local\CompanyData\IT` über den Client `CLI-WIN-01` (Benutzer `mmustermann`).
   * Verifikation der Zugriffsverweigerung (*Zugriff verweigert*) beim Versuch, den Ordner `HR` zu öffnen.
   
   <img width="1024" height="768" alt="Zugriff auf ordner_CLI-WIN-01_28_09_2026_16_13_06" src="https://github.com/user-attachments/assets/32523625-ff25-4e8c-8051-0153b912d173" />

