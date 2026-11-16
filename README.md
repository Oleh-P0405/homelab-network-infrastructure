> [!NOTE]
> 💻 **Projekt erstellt von OLEH PONOCHOVNYI (https://github.com/Oleh-P0405)** | *IT-Systemadministration Lab*
---

# homelab-network-infrastructure
Aufbau einer virtuellen Testumgebung (Windows Server 2022, Ubuntu (Linux) Server, Windows 10) in VirtualBox

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
- [x] Einrichten von **Active Directory Domain Services (AD DS)** auf `SRV-WIN-01`.
- [x] Anbinden des Clients `CLI-WIN-01` an die Domäne.
- [x] Konfiguration von **DHCP-** und **DNS-Diensten**.

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
- [x] Integration des Linux-Servers (`SRV-LNX-01`) in die Domäneninfrastruktur oder Konfiguration von SSH/RSYNC.
- [x] Erstellung von Gruppenrichtlinien (**Group Policy Objects / GPO**) im Active Directory.
- [x] Einrichten von Netzwerkfreigaben (SMB / Dateiserver) mit Zugriffssteuerung.

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
   
---

## 🎯 Nächste Schritte (Phase 4)

- [x] **Phase 4.1:** Integration des Linux-Servers (`SRV-LNX-01`) in die Domäne & Netzdienste (SMB / Nginx).
- [x] **Phase 4.2:** Erweiterte Gruppenrichtlinien (**Group Policy Objects / GPO**).
- [x] **Phase 4.3:** Projektabschluss, Gesamtvalidierung & Dokumentation.

## 🛠️ Phase 4.1: Linux-Integration in die Domäne & Netzdienste (SMB / Nginx)

1. **SMB/CIFS-Netzwerkfreigabe-Einbindung:**
   * Installation der benötigten Paketwerkzeuge (`cifs-utils`, `smbclient`) auf `SRV-LNX-01`.
   * Erstellung der geschützten Anmeldedatei `/etc/wincredentials` (Rechte `600`) mit den AD-Zugangsdaten des Benutzers **Max Mustermann** (`maxmustermann`).
   * Erstellung des lokalen Mount-Punkts `/mnt/companydata`.
   * Erfolgreiches Einbinden der Windows-Freigabe `//srv-win-01.lab.local/CompanyData` unter Verwendung von SMB 3.0.

2. **Nginx-Webserver-Bereitstellung:**
   * Installation und Aktivierung des Webservers **Nginx** auf `SRV-LNX-01`.
   * Erstellung einer benutzerdefinierten Intranet-Startseite in `/var/www/html/index.html` (*"Willkommen im lab.local Intranet!"*).

3. **DNS-Integration auf dem Domänencontroller (`SRV-WIN-01`):**
   * Konfiguration eines **A-Records** in der DNS-Zone `lab.local`: `srv-lnx-01` -> `192.168.10.10`.
   * Erstellung eines **CNAME-Alias (Pseudonym)**: `web.lab.local` -> `srv-lnx-01.lab.local`.

4. **Validierung & Funktionstest:**
   * Aufruf der Intranet-Webseite über den Browser auf dem Client `CLI-WIN-01` unter der Adresse **`http://web.lab.local`**.
   * Verifikation der vollständigen Erreichbarkeit des Linux-Webservers über den DNS-Namen.
   * Bestätigung des Lese- und Schreibzugriffs auf die eingebundenen SMB-Ordner (`IT` / `HR`) auf dem Linux-System.

5. **Troubleshooting & Fehlerbehebung:**
   * **`NT_STATUS_LOGON_FAILURE` / Fehler -13:** Behebung durch Korrektur des SAM-Benutzernamens auf `maxmustermann` in `/etc/wincredentials`.
   * **`could not resolve address`:** Behebung durch explizite Konfiguration des DNS-Servers (`192.168.10.20`) in `/etc/resolv.conf`.

<img width="1024" height="768" alt="web lab local2_CLI-WIN-01_01_11_2026_14_50_08" src="https://github.com/user-attachments/assets/215b36c5-398e-4ace-a23b-1d073007e4bb" />

## 🛡️ Phase 4.2: Erweiterte Gruppenrichtlinien (GPO - Group Policy Objects)

1. **Erstellung und Konfiguration der GPO (`Sec_Policy_LegalNotice`):**
  - Erstellung eines neuen GPO-Objekts `Sec_Policy_LegalNotice` in der **Gruppenrichtlinien-Verwaltung (`gpmc.msc`)**.
  - Verknüpfung der Richtlinie mit der Ziel-Organisationseinheit **`Lab-OU`**.
  - Konfiguration der Computerkonfiguration (*Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Local Policies -> Security Options*):
    - **Interaktive Anmeldung: Titel des Meldungsfelds:** `Achtung - Firmennetzwerk lab.local`
    - **Interaktive Anmeldung: Text für Benutzer:** `Unbefugter Zugriff untersagt. Alle Aktivitäten werden überwacht.`

2. **Troubleshooting & OU-Strukturkorrektur:**
  - **Problem:** Die Richtlinie wurde nach initialer Erstellung nicht auf dem Client angewendet.
  - **Ursache:** Der Client-PC befindet sich im Standard-Container `Computers`, auf den keine OU-spezifischen GPOs wirken, und wies eine abweichende Namenskonvention auf.
  - **Lösung:** 
    1. Umbenennung des Client-Computers in den Standardnamen **`CLI-WIN-01`** via `sysdm.cpl`.
    2. Verschieben des Computer-Objekts `CLI-WIN-01` aus dem `Computers`-Container in die **`Lab-OU`** via Active Directory-Benutzer und -Computer (`dsa.msc`).

3. **Validierung & Funktionstest auf `CLI-WIN-01`:**
  - Ausführung von `gpupdate /force` in einer administrativen Eingabeaufforderung.
  - Verifikation der angewendeten Richtlinien via `gpresult /r /scope computer` sowie `rsop.msc`.
  - Erfolgreiche Bestätigung: Beim Neustart des Systems erscheint vor der Benutzeranmeldung der konfigurierte rechtliche Sicherheitshinweis (Legal Notice) im Pop-up-Fenster.

<img width="1024" height="768" alt="GPO Phase 4 2_CLI-WIN-01_04_11_2026_15_03_18" src="https://github.com/user-attachments/assets/0f0fef57-ee23-4556-9811-0597c624bde4" />

4. **Benutzerbezogene GPO (`User_Restrict_ControlPanel`):**
  - Erstellung eines neuen GPO-Objekts `User_Restrict_ControlPanel` in der **Gruppenrichtlinien-Verwaltung (`gpmc.msc`)**     und Verknüpfung mit der **`Lab-OU`**.
  - Konfiguration der Benutzerkonfiguration (*Benutzerkonfiguration -> Richtlinien -> Administrative Vorlagen ->              Systemsteuerung*):
    - **Zugriff auf Systemsteuerung und PC-Einstellungen nicht zulassen:** `Aktiviert` (Sperrt zentral den Zugriff auf die    Systemsteuerung und die Windows-Einstellungen für Domänenbenutzer).
  - **Validierung & Funktionstest auf `CLI-WIN-01`:**
    - Ausführung von `gpupdate /force` unter dem Benutzerkonto `Max Mustermann` (`mmustermann`).
    - **Erfolgreiche Bestätigung:** Der Versuch, die Systemsteuerung oder die Windows-Einstellungen (`Win + I`) zu öffnen,    wird erfolgreich mit einer Systembeschränkungsmeldung blockiert.

    <img width="1024" height="768" alt="GPO_Teil2_Beschränkungen_CLI-WIN-01_13_11_2026_11_57_22" src="https://github.com/user-attachments/assets/eb521c2a-9676-4c0f-a835-1028e86da5fe" />

✅ Phase 4.3: Gesamtvalidierung & Projektabschluss

1. DNS- & Active Directory-Diagnose (SRV-WIN-01):

   * Überprüfung der Domänen- und DNS-Integrität mittels dcdiag /test:dns.

   * DNS-Troubleshooting & Optimierung:

   * Problem: Initiale DNS-Tests meldeten Fehler bei den Untertests Forwarders (Forw) und Record Registration (RReg).

   * Lösung: Konfiguration eines externen DNS-Forwarders (8.8.8.8) im DNS-Manager (dnsmgmt.msc), Anpassung der lokalen IPv4-DNS-Bindung auf SRV-WIN-01 (ncpa.cpl) sowie Neuregistrierung der DNS-Einträge via ipconfig /flushdns, ipconfig /registerdns und Neustart des Dienstes Netlogon. - <img width="1024" height="768" alt="FInal teil3_SRV-WIN-01_13_11_2026_12_26_46" src="https://github.com/user-attachments/assets/eb8e63ec-606b-440f-83f8-a227067a1c69" />


   * Ergebnis: Bestätigung durch lab.local hat den Test DNS bestanden (alle domänenrelevanten Untertests erfolgreich mit         PASS abgeschlossen).
   *<img width="1024" height="768" alt="Final teil4_SRV-WIN-01_13_11_2026_13_06_07" src="https://github.com/user-attachments/assets/c8775740-b379-4225-8239-32e8ddaa7262" />*


2. End-to-End Client- & GPO-Validierung (CLI-WIN-01):

   * Verifikation der Namensauflösung und Konnektivität zum Domain Controller (nslookup lab.local, ping SRV-WIN-01.lab.local).

   * Bestätigung der korrekten OU-Zugehörigkeit von CLI-WIN-01 und Max Mustermann (mmustermann) innerhalb der Lab-OU.

   * Überprüfung der angewendeten Gruppenrichtlinien via gpresult /r:

   * Sec_Policy_LegalNotice: Anmeldebanner wird vor jedem System-Login erzwungen.

   * User_Restrict_ControlPanel: Zugriff auf Systemsteuerung und Windows-Einstellungen ist für Domänenbenutzer gesperrt.

   *<img width="1024" height="768" alt="Final Teil1_CLI-WIN-01_13_11_2026_12_10_30" src="https://github.com/user-attachments/assets/b97468d5-cf9e-4068-a4c9-b3932ba5251c" />*
   *<img width="1024" height="768" alt="Finall teil2_CLI-WIN-01_13_11_2026_12_16_41" src="https://github.com/user-attachments/assets/3a90a819-42c1-48ff-aa24-13b285399ccc" />*


3. Projektabschluss:

   * Vollständiger Abschluss und Dokumentation aller Projektphasen (Netzwerk-Setup, Active Directory & DNS, GPO-Härtung sowie Client-Integration).

   * Die Laborumgebung lab.local ist fehlerfrei validiert und einsatzbereit.

*<img width="1024" height="768" alt="Final teil4_SRV-WIN-01_13_11_2026_13_06_07" src="https://github.com/user-attachments/assets/ed7d5ce4-b74b-4ecc-b24c-b95c0dc1584d" />*

---

# Fazit & Lessons Learned

In diesem Projekt wurde eine vollständige, realitätsnahe Unternehmensinfrastruktur auf Basis von **Windows Server (Active Directory & DNS)** sowie **Windows- und Linux-Clients** von Grund auf aufgebaut, abgesichert und dokumentiert.

### 📋 Zusammenfassung der Projektschritte

* **Phase 1: Netzwerk & Grundlagen (`SRV-WIN-01` & `CLI-WIN-01`)**
  - [x] Grundkonfiguration der statischen IP-Adressen und Netzwerkinfrastruktur.
  - [x] Namensauflösung und funktionierende Internet-Anbindung über DNS-Forwarder.

* **Phase 2: Active Directory & DNS-Infrastruktur**
  - [x] Einrichten von **Active Directory Domain Services (AD DS)** auf `SRV-WIN-01`.
  - [x] Anbinden des Clients `CLI-WIN-01` an die Domäne.
  - [x] Konfiguration von **DHCP-** und **DNS-Diensten**.

* **Phase 3: Gruppenrichtlinien (GPOs) & Sicherheit / Linux-Integration & Fernverwaltung (SSH)**
  - [x] Integration des Linux-Servers (`SRV-LNX-01`) in die Domäneninfrastruktur oder Konfiguration von SSH/RSYNC.
  - [x] Erstellung von Gruppenrichtlinien (**Group Policy Objects / GPO**) im Active Directory.
  - [x] Einrichten von Netzwerkfreigaben (SMB / Dateiserver) mit Zugriffssteuerung.

* **Phase 4: Gesamtvalidierung & Projektabschluss**
  - [x] **Phase 4.1:** Integration des Linux-Servers (`SRV-LNX-01`) in die Domäne & Netzdienste (SMB / Nginx).
  - [x] **Phase 4.2:** Erweiterte Gruppenrichtlinien (**Group Policy Objects / GPO**).
  - [x] **Phase 4.3:** Projektabschluss, Gesamtvalidierung & Dokumentation.

### Wichtigste Erkenntnisse (Lessons Learned)
* **DNS als Fundament von Active Directory:** Eine fehlerfreie Namensauflösung (AD-integrierte Zonen, korrekte Forwarder und dynamische SRV-Einträge) ist zwingende Voraussetzung für den Domänenbeitritt und die Replikation. Mit `dcdiag /test:dns` lassen sich Konfigurationsfehler gezielt isolieren und beheben.
* **Strukturierte OU-Planung:** Gruppenrichtlinien (GPOs) wirken nur dann zuverlässig, wenn Computer- und Benutzerobjekte sauber in den vorgesehenen Organisationseinheiten (`Lab-OU`) statt im Standard-Container `Computers` verwaltet werden.
* **Trennung von Computer- und Benutzerkonfiguration:** Sicherheitsrichtlinien wie das Anmeldebanner (`Sec_Policy_LegalNotice`) greifen auf Maschinenebene beim Systemstart, während Einschränkungen wie die Sperrung der Systemsteuerung (`User_Restrict_ControlPanel`) gezielt auf Benutzerebene angewendet werden.

---
> [!NOTE]
> 💻 **Projekt erstellt von OLEH PONOCHOVNYI (https://github.com/Oleh-P0405)** | *IT-Systemadministration Lab*
