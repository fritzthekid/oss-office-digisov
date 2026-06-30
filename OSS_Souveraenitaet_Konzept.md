# Digitale Souveränität

**OSS-Plattformkonzept für dezentrale Arbeitsumgebungen**

*Entwurf – Juni 2026 · Eduard Moser & claude.ai*

---

## Inhaltsverzeichnis

1. Einleitung
2. Grundprinzipien
3. Architekturübersicht
4. Client-Typen und Paketempfehlungen
5. Exchange-Funktionalität im OSS-Stack
6. Stärken und Schwächen gegenüber openDesk
7. Datensouveränität
8. Empfehlung und Migrationspfad

Anhang A – Vergleich OSS-Ansätze für digitale Souveränität
Anhang B – Proxmox VE als Infrastrukturschicht für mittlere Organisationen
Anhang C – Kostenvergleich europäischer IaaS-Anbieter
Anhang D – TPM-gestützte Authentifizierung

Quellenverzeichnis

---

# 1 Einleitung

Die zunehmende Abhängigkeit von proprietären Cloud-Plattformen — insbesondere Microsoft 365 — stellt Organisationen vor grundlegende Fragen der digitalen Souveränität: Wer kontrolliert die eigenen Daten? Wie vermeidet man Herstellerabhängigkeiten? Und wie lässt sich eine praxistaugliche Arbeitsumgebung aufbauen, die ohne proprietäre Software auskommt?

Dieses Konzept beschreibt eine vollständig auf Open-Source-Software basierende Arbeitsplattform, die als Alternative zu zentral verwalteten Diensten wie Microsoft 365 oder dem Bund-Länder-Projekt openDesk positioniert ist. Es richtet sich an kleine und mittlere Organisationen, NGOs, Initiativen und Einzelpersonen, die digitale Souveränität nicht nur als politisches Ziel, sondern als praktische Anforderung verstehen.

Der Ansatz folgt dem „thin server / fat client"-Prinzip: Server übernehmen ausschließlich Infrastrukturaufgaben (Ablage, Synchronisation, Kommunikation), während rechenintensive Aufgaben — von der Textverarbeitung bis zur KI-Inferenz — auf den Clients verbleiben. Dies steht im bewussten Kontrast zu server-zentrischen Ansätzen wie openDesk, die Anwendungslogik und Rendering auf zentrale Infrastruktur verlagern.

Das Dokument gliedert sich in acht Kapitel und vier Anhänge. Die Kapitel 2–4 beschreiben Prinzipien, Architektur und empfohlene Software. Kapitel 5 zeigt, wie Exchange-Funktionalität im OSS-Stack abgebildet wird. Kapitel 6 und 7 behandeln den Vergleich mit openDesk und das Thema Datensouveränität. Kapitel 8 gibt eine konkrete Empfehlung für die schrittweise Einführung. Die Anhänge vertiefen den Vergleich mit dem Schleswig-Holstein-Modell (A), die Proxmox-Infrastruktur für mittlere Organisationen (B), einen europäischen IaaS-Kostenvergleich (C) und TPM-gestützte Authentifizierung (D).

# 2 Grundprinzipien

Im Mittelpunkt steht das Prinzip der digitalen Souveränität: Kontrolle über eigene Daten, Freiheit von Herstellerabhängigkeiten und Nachvollziehbarkeit durch quelloffene Software.

- Ausschließlich Open-Source-Software (OSI-zertifiziert oder vergleichbar)

- Offene Protokolle – kein proprietäres Datenformat, kein Vendor Lock-in

- Lokale Rechenleistung wird genutzt, nicht verschwendet (fat client)

- Server als schlanke Infrastruktur, nicht als Anwendungsplattform

- Offline-Fähigkeit: Kernanwendungen funktionieren ohne Serververbindung

- Datensouveränität: Daten liegen wo der Betreiber es bestimmt

# 3 Architekturübersicht

Die Plattform gliedert sich in drei Schichten: Server-Infrastruktur, offene Protokolle und Client-Ebene.

## 3.1 Server-Infrastruktur

Der Server ist bewusst dünn gehalten. Er stellt Speicher, Kalender, Kommunikation und sicheren Zugang bereit – keine Anwendungslogik, kein serverseitiges Rendering. Damit bleibt die Last gering und die Hardware-Anforderungen moderat (z. B. Raspberry Pi 4 oder kleiner VPS).

| **Dienst**              | **Software**                | **Protokoll / Standard**  | **Funktion**                                  |
|-------------------------|-----------------------------|---------------------------|-----------------------------------------------|
| Dateiablage & Kalender  | Nextcloud                   | WebDAV / CalDAV / CardDAV | Zentrale Ablage, Kalender, Kontakte, Aufgaben |
| Kollaborative Dokumente | Collabora Online (optional) | WOPI                      | Co-Editing – nur bei Bedarf, separater Host   |
| E-Mail (MTA)            | Postfix                     | SMTP                      | Mailversand und -empfang                      |
| E-Mail (MDA/IMAP)       | Dovecot                     | IMAP / LMTP               | Postfächer, Shared Mailboxen                  |
| Spamfilter              | rspamd                      | –                         | Spam- und Virenschutz                         |
| Kommunikation           | Matrix (Synapse)            | Matrix-Protokoll          | Chat, Videokonferenz (Element)                |
| VPN / Zugang            | WireGuard                   | WireGuard                 | Sicherer Fernzugriff                          |
| Reverse Proxy           | nginx / Caddy               | HTTPS / TLS               | Zertifikate, Routing                          |
| Authentifizierung       | Keycloak (optional)         | OIDC / SAML               | Single Sign-On                                |

## 3.2 Protokollschicht

Alle Dienste kommunizieren ausschließlich über offene, standardisierte Protokolle. Dadurch ist jede Komponente austauschbar, ohne die Gesamtarchitektur zu berühren.

- WebDAV / CalDAV / CardDAV – Datei-, Kalender- und Kontaktsync

- SMTP / IMAP – E-Mail (RFC-konform, vollständig interoperabel)

- Matrix / XMPP – dezentrale Kommunikation

- OIDC / SAML – Authentifizierung und SSO

- WireGuard – VPN-Zugang

## 3.3 Client-Ebene

Auf der Client-Ebene liegt die eigentliche Arbeitsleistung. Clients sind vollständig funktionsfähig auch ohne aktive Serververbindung; Synchronisation erfolgt bei Verbindungsaufbau.

# 4 Client-Typen und Paketempfehlungen

## 4.1 Standard-Arbeitsplatz (Linux-Desktop)

Für Büroarbeit, Dokumentenerstellung und Kommunikation. Empfohlene Distribution: Ubuntu LTS, Debian stable oder Linux Mint.

| **Kategorie**     | **Paket**                | **Bemerkung**                                      |
|-------------------|--------------------------|----------------------------------------------------|
| Office            | LibreOffice              | Writer, Calc, Impress – voller OOXML-Import/Export |
| E-Mail & Kalender | Thunderbird              | mit DAV-Erweiterung (TbSync) für CalDAV/CardDAV    |
| Dateiablage       | Nextcloud-Desktop-Client | automatischer Sync, Konfliktbehandlung             |
| Kommunikation     | Element (Matrix-Client)  | Chat und Videoanruf                                |
| PDF               | Okular / Evince          | Anzeige; Export aus LibreOffice                    |
| Browser           | Firefox                  | mit uBlock Origin                                  |
| Passwörter        | KeePassXC                | lokal, kein Cloud-Zwang                            |

## 4.2 Entwickler-Workstation

Für Softwareentwicklung, Skripting und Administration. Setzt auf einem Standard-Desktop auf.

| **Kategorie**     | **Paket**                 | **Bemerkung**                                 |
|-------------------|---------------------------|-----------------------------------------------|
| IDE               | VS Code (Codium) / Neovim | VSCodium = Build ohne Microsoft-Telemetrie    |
| Versionskontrolle | Git + Gitea (Server)      | Gitea als selbst-gehostetes GitHub-Äquivalent |
| Container         | Docker / Podman           | Podman rootless bevorzugt                     |
| Terminal          | Alacritty / kitty + tmux  |                                               |
| Skriptsprachen    | Python 3, Bash, Node.js   |                                               |
| Datenbank-GUI     | DBeaver                   | OSS, unterstützt alle gängigen DBs            |

## 4.3 KI-Workstation (lokale Inferenz)

Für rechenintensive KI-Aufgaben lokal – kein Cloud-Dienst, keine Datenweitergabe. Empfohlene Hardware: dedizierter Desktop mit diskreter GPU (z. B. RTX 4070 Ti oder 5070 Ti).

| **Kategorie**   | **Paket**                  | **Bemerkung**                                           |
|-----------------|----------------------------|---------------------------------------------------------|
| LLM-Inferenz    | Ollama                     | einfache Verwaltung lokaler Modelle (llama, mistral, …) |
| LLM-Frontend    | Open WebUI                 | Browser-UI für Ollama, selbst-gehostet                  |
| Bildgenerierung | Stable Diffusion (ComfyUI) | lokal, kein Cloud-Zwang                                 |
| Vektor-DB       | Chroma / Qdrant            | für RAG-Anwendungen                                     |
| GPU-Stack       | CUDA / ROCm + nvidia-smi   | je nach GPU-Hersteller                                  |

## 4.4 Mobile (Android / iOS)

| **Funktion**        | **Android**            | **iOS**                                   |
|---------------------|------------------------|-------------------------------------------|
| Dateiablage         | Nextcloud-App          | Nextcloud-App                             |
| Kalender & Kontakte | DAVx⁵                  | Nativ (CalDAV/CardDAV direkt einstellbar) |
| E-Mail              | K-9 Mail / Thunderbird | Canary Mail / Mimestream                  |
| Matrix-Chat         | Element                | Element                                   |
| VPN                 | WireGuard-App          | WireGuard-App                             |
| Passwörter          | KeePassDX              | Strongbox                                 |

# 5 Exchange-Funktionalität im OSS-Stack

Microsoft Exchange ist in Office-365-Umgebungen mehr als ein Mailserver – er bündelt E-Mail, Kalender, Kontakte, Aufgaben, Ressourcenbuchung und Push-Synchronisation. Im OSS-Stack ist diese Funktionalität dezentral über mehrere Komponenten abgebildet, die über offene Standards interagieren.

| **Exchange-Funktion**        | **OSS-Äquivalent**              | **Protokoll / Paket** | **Einschränkung**                                 |
|------------------------------|---------------------------------|-----------------------|---------------------------------------------------|
| E-Mail                       | Postfix + Dovecot               | SMTP / IMAP           | Vollständig äquivalent                            |
| Kalender                     | Nextcloud Calendar              | CalDAV                | Vollständig äquivalent                            |
| Kontakte                     | Nextcloud Contacts              | CardDAV               | Vollständig äquivalent                            |
| Global Address List          | LDAP (OpenLDAP)                 | LDAP                  | Manuelle Pflege erforderlich                      |
| Aufgaben / Tasks             | Nextcloud Tasks                 | CalDAV (VTODO)        | Vollständig äquivalent                            |
| Ressourcenbuchung            | Nextcloud Calendar (Ressourcen) | CalDAV                | Vollständig äquivalent                            |
| Push auf Mobile              | DAVx⁵ + Nextcloud               | CalDAV / CardDAV      | Kein nativer Push; Poll-Intervall konfigurierbar  |
| Shared Mailboxen             | Dovecot + Sieve                 | IMAP ACL              | Vollständig äquivalent                            |
| Spamschutz                   | rspamd                          | Milter                | Vollständig äquivalent                            |
| Kalendereinladungen (extern) | Postfix + iCal                  | iCal/RFC 5545         | Interoperabel; Outlook-Nutzer außen: Z-Push nötig |
| ActiveSync (Outlook-Clients) | Z-Push (optional)               | ActiveSync            | Nur nötig wenn Outlook-Clients gewünscht          |

### Hinweis zu Outlook-Kompatibilität

Outlook-Clients erwarten intern ActiveSync oder MAPI, nicht CalDAV. In einer homogenen OSS-Umgebung ist das kein Problem – Thunderbird und native Kalender-Apps kommunizieren direkt per CalDAV/CardDAV. Sobald Outlook-Nutzer integriert werden müssen, ist Z-Push als ActiveSync-Brücke der pragmatische Weg. Z-Push ist Open Source (AGPLv3) und in Nextcloud integrierbar.

# 6 Stärken und Schwächen gegenüber openDesk

openDesk ist der Bund-Länder-Ansatz für souveräne Bürosoftware in der öffentlichen Verwaltung. Er hat seine Berechtigung in stark regulierten, zentralisierten IT-Umgebungen. Das vorliegende Konzept verfolgt einen anderen Ansatz und hat andere Stärken.

## 6.1 Stärken dieses Konzepts

- **Lokale Rechenleistung wird genutzt**

- KI-Inferenz, Bildbearbeitung, Kompilierung laufen lokal – kein Server-Bottleneck

- Clients bleiben leistungsfähig bei Serverausfall

- **Geringere Server-Hardware-Anforderungen**

- Kein serverseitiges Rendering (kein Collabora-Load bei vielen Nutzern)

- Raspberry Pi 4 oder kleiner VPS genügt für Kerninfrastruktur

- **Offline-Fähigkeit**

- LibreOffice, Thunderbird, KeePassXC funktionieren ohne Netz

- **Einfachere Wartung**

- Weniger Abhängigkeiten server-seitig, klare Trennung der Verantwortlichkeiten

- **Kein Vendor Lock-in auf Betriebssystemebene**

- openDesk setzt faktisch auf bestimmte Linux-Distributionen; dieses Konzept ist distributionsagnostisch

## 6.2 Schwächen und Grenzen

- **Kein echtes Co-Editing im Standardfall**

- LibreOffice lokal + Nextcloud-Ablage ermöglicht kein gleichzeitiges Bearbeiten derselben Datei

- Lösung: Collabora auf separatem Host (optional, bei Bedarf)

- **Heterogenere Client-Landschaft**

- Unterschiedliche Geräte / Distributionen erfordern mehr Konfigurationsaufwand

- openDesk setzt auf verwaltete Thin-Client-Images

- **OOXML-Kompatibilität**

- LibreOffice importiert .docx/.xlsx zuverlässig; pixelgenaues Layout-Übernahme bei komplexen Vorlagen nicht garantiert

- PDF-Export ist konsistent und praxistauglich

- **Zertifizierung / Compliance**

- openDesk wird BSI-begleitet; dieses Konzept muss Compliance-Anforderungen (z. B. BSI Grundschutz) eigenständig erfüllen

## 6.3 Vergleichstabelle

| **Kriterium**              | **Dieses Konzept**           | **openDesk**                 |
|----------------------------|------------------------------|------------------------------|
| Rechenleistung             | Client (fat)                 | Server (thin client)         |
| Offline-Fähigkeit          | Vollständig                  | Eingeschränkt                |
| Co-Editing                 | Optional (Collabora separat) | Integriert                   |
| Server-Hardware            | Gering                       | Höher                        |
| Verwaltungsaufwand Clients | Höher                        | Geringer (Thin-Client-Image) |
| BSI-Zertifizierung         | Eigenverantwortung           | Geplant / begleitet          |
| Vendor Lock-in             | Keiner                       | Keiner (beide OSS)           |
| OOXML-Kompatibilität (PDF) | Konsistent                   | Konsistent                   |

# 7 Datensouveränität

## 7.1 Prinzipien

- Alle Daten liegen auf Infrastruktur unter Kontrolle des Betreibers (On-Premise oder dedizierter VPS)

- Keine Telemetrie, kein Cloud-Zwang, keine Drittland-Übermittlung

- Vollständig quelloffene Software – keine proprietären Binärkomponenten im Kern-Stack

- Offene Protokolle ermöglichen Migration ohne Datenverlust

## 7.2 Hosting-Szenarien

| **Szenario**                | **Infrastruktur**                         | **Souveränitätsniveau** | **Geeignet für**                   |
|-----------------------------|-------------------------------------------|-------------------------|------------------------------------|
| Vollständig On-Premise      | Eigene Hardware (z. B. RPi4 / Homeserver) | Maximal                 | Kleine Gruppen, Privat, Vereine    |
| Dedizierter VPS (EU)        | Hetzner DE / Netcup / Greenbone           | Hoch                    | KMU, Initiativen, NGO              |
| Shared Hosting OSS-Anbieter | Hetzner Storage Box + VPS                 | Mittel-hoch             | Wenn eigene Hardware nicht möglich |
| Hyperscaler (AWS/Azure)     | Nein – CLOUD Act Risiko                   | Nicht akzeptabel        | –                                  |

## 7.3 Rechtliche Rahmenbedingungen

- DSGVO: Verarbeitungsverzeichnis, Auftragsverarbeitungsvertrag (AVV) bei externem Hosting

- CLOUD Act: US-amerikanische Cloud-Anbieter (auch EU-Rechenzentren) sind grundsätzlich betroffen – zu vermeiden

- BSI Grundschutz: bei Einsatz in Behörden / kritischer Infrastruktur als Orientierung empfohlen

- Gesundheitsdaten (z. B. Psychotherapiepraxis): erhöhte Anforderungen nach § 9 DSGVO; On-Premise oder nachweislich DSGVO-konformer EU-Anbieter obligatorisch

## 7.4 Backup und Verfügbarkeit

- Restic + rclone – verschlüsselte Backups auf eigene oder Hetzner Storage Box

- 3-2-1-Regel: 3 Kopien, 2 verschiedene Medien, 1 offsite

- WireGuard-Tunnel für verschlüsselten Fernzugriff auch bei dynamischer IP

- Monitoring: Checkmk (OSS-Edition) oder Grafana + Prometheus

# 8 Empfehlung und Migrationspfad

Das Konzept ist praxistauglich und kann schrittweise eingeführt werden. Empfohlene Reihenfolge:

- **Phase 1 – Kern-Infrastruktur: Nextcloud + Postfix/Dovecot + WireGuard auf eigenem Server oder VPS**

- **Phase 2 – Client-Rollout: LibreOffice, Thunderbird (mit TbSync), Nextcloud-Desktop-Client, Element**

- **Phase 3 – Kommunikation: Matrix/Synapse für Chat und Videokonferenz**

- **Phase 4 – Optional: Collabora auf separatem Host für Co-Editing; Keycloak für SSO; Ollama auf KI-Workstation**

Collabora wird bewusst als optional markiert: Im Regelfall reicht LibreOffice lokal + Nextcloud als reiner Dateisync. Co-Editing ist die Ausnahme, nicht die Regel – und wenn, dann auf dedizierter Hardware, nicht auf dem Kern-Server.

---

# Anhang A – Vergleich OSS-Ansätze für digitale Souveränität

Dieser Anhang stellt das vorliegende Konzept in den Kontext zweier etablierter öffentlicher Initiativen: dem Ansatz des Landes Schleswig-Holstein und dem Bund-Länder-Projekt openDesk. Alle drei Ansätze verfolgen dasselbe Ziel — Unabhängigkeit von proprietären US-Produkten — setzen aber unterschiedliche Schwerpunkte.

## A.1 Schleswig-Holstein: Praxisbeispiel Landesverwaltung

### A.1.1 Was wurde umgesetzt?

Im Oktober 2025 schloss die Landesverwaltung Schleswig-Holstein die Migration von Microsoft Exchange und Outlook auf Open-Xchange und Thunderbird ab — 44.000 Postfächer, über 100 Millionen E-Mails und Kalendereinträge in sechs Monaten.[^1] Das Land ist damit das erste deutsche Bundesland, das Exchange vollständig abgelöst hat.

Der Gesamtstack umfasst:

- **Open-Xchange App Suite** — Groupware-Server (Mail, Kalender, Kontakte, Aufgaben) mit Web-Frontend[^2]

- **Thunderbird** — Desktop-Client (ersetzt Outlook)

- **Univention Nubus (UCS)** — Identitäts- und Zugriffsmanagement, ersetzt Microsoft Active Directory[^3]

- **Nextcloud** — Dateiablage und Kollaboration (rollout laufend)

- **OpenTalk** — Videokonferenz (OSS, selbst-gehostet)[^4]

- **Linux-Migration** — Windows-Ablösung wird pilotiert[^5]

- **Dataport AöR** — IT-Dienstleister der norddeutschen Länder, verantwortlich für Betrieb[^6]

### A.1.2 Wirtschaftliche Bilanz

Das Land berichtet von Einsparungen über 15 Millionen Euro jährlich an Lizenzkosten, denen 2026 einmalig 9 Millionen Euro Migrationsaufwand gegenüberstehen.[^7] Bis Dezember 2025 waren bereits 80 % der Arbeitsplätze umgestellt, mit positivem Zwischenfazit.[^8]

Die verbleibenden 20 % der Arbeitsplätze benötigen noch Microsoft Office, weil Fachverfahren technische Abhängigkeiten bedingen. Migrationspfade sind vereinbart.

## A.2 Protokolle: Warum alle drei Ansätze kompatibel sind

Der entscheidende Punkt: Open-Xchange, Nextcloud und Postfix/Dovecot sprechen dieselben offenen Protokolle. Die Schnittstellen sind identisch — lediglich die Implementierungsschicht darüber unterscheidet sich.[^9][^10]

| **Protokoll / Standard** | **Open-Xchange (SH)** | **Nextcloud (eigenes Konzept)** | **Postfix/Dovecot** |
|--------------------------|-----------------------|---------------------------------|---------------------|
| IMAP / SMTP              | ✓ nativ               | via Postfix/Dovecot             | ✓ nativ             |
| CalDAV (Kalender)        | ✓ nativ               | ✓ nativ                         | —                   |
| CardDAV (Kontakte)       | ✓ nativ               | ✓ nativ                         | —                   |
| WebDAV (Dateien)         | ✓ (OX Drive)          | ✓ nativ                         | —                   |
| iCal / RFC 5545          | ✓                     | ✓                               | —                   |
| ActiveSync (Outlook)     | ✓ (nativ)             | via Z-Push                      | via Z-Push          |
| LDAP / OIDC              | ✓                     | via Keycloak / LDAP             | —                   |
| Matrix / XMPP            | —                     | ✓ (separater Dienst)            | —                   |

Das bedeutet: Ein Thunderbird-Client, der mit Open-Xchange kommuniziert, könnte mit denselben Einstellungen (IMAP/CalDAV/CardDAV) auch gegen einen Nextcloud+Dovecot-Stack arbeiten. Die Protokolle sind austauschbar, nicht die Produkte.

Einzige Ausnahme: ActiveSync für Outlook-Clients. Open-Xchange unterstützt ActiveSync nativ; im eigenen Konzept ist dafür Z-Push als Brücke nötig.[^11]

## A.3 Drei Ansätze im Vergleich

| **Kriterium**      | **Schleswig-Holstein**           | **openDesk (ZenDiS)**            | **Eigenes Konzept**             |
|--------------------|----------------------------------|----------------------------------|---------------------------------|
| Zielgruppe         | Landesverwaltung (30.000 Nutzer) | Öffentliche Verwaltung allgemein | KMU, Initiativen, Privat        |
| Mail-Kern          | Open-Xchange + Thunderbird       | Open-Xchange / Zimbra            | Postfix + Dovecot + Thunderbird |
| Kalender/Kontakte  | OX (CalDAV/CardDAV)              | OX (CalDAV/CardDAV)              | Nextcloud (CalDAV/CardDAV)      |
| Dateiablage        | Nextcloud                        | Nextcloud                        | Nextcloud                       |
| Videokommunikation | OpenTalk                         | OpenTalk / Jitsi                 | Matrix / Element                |
| IdM / SSO          | Univention Nubus                 | Univention Nubus / Keycloak      | Keycloak (optional)             |
| Office-Suite       | LibreOffice                      | LibreOffice + Collabora          | LibreOffice lokal               |
| Client-Paradigma   | Managed Desktop (thin)           | Thin Client / Browser            | Fat Client (lokal)              |
| Betrieb            | Dataport AöR (zentral)           | Zentraler Dienst                 | Selbst / VPS                    |
| BSI-Begleitung     | Geplant                          | Ja (ZenDiS)                      | Eigenverantwortung              |
| Lizenzkosten       | Stark reduziert (\>15 Mio. €/a)  | Stark reduziert                  | Minimal                         |

## A.4 Wesentliche Unterschiede und Begründungen

### A.4.1 Open-Xchange vs. Postfix/Dovecot/Nextcloud

Open-Xchange ist eine integrierte Groupware-Plattform: Mail, Kalender, Kontakte und Dateiablage in einem Produkt mit eigenem Web-Frontend. Das ist für große, zentral verwaltete Umgebungen ein Vorteil — ein System, ein Admin-Interface, ein Supportvertrag.

Der eigene Konzeptansatz trennt diese Funktionen bewusst:

- Postfix/Dovecot für Mail — minimalistisch, extrem stabil, seit Jahrzehnten bewährt

- Nextcloud für Kalender, Kontakte, Dateien — modular, mit eigenem Ökosystem

- Matrix/Element für Kommunikation — dezentrales Protokoll, kein zentraler Server nötig

Das ergibt mehr Flexibilität und geringere Abhängigkeit von einem einzelnen Produkt, erfordert aber mehr Konfigurationsaufwand und Admin-Kenntnisse.

### A.4.2 Thin Client (SH/openDesk) vs. Fat Client (eigenes Konzept)

Schleswig-Holstein und openDesk setzen auf verwaltete Arbeitsplätze, bei denen Anwendungslogik zunehmend server- oder browser-seitig läuft. Das vereinfacht die Client-Administration erheblich — besonders in einer Verwaltung mit heterogenen, schwer kontrollierbaren Endgeräten.

Das eigene Konzept verfolgt das Gegenmodell: Der Client ist der Arbeitspferd. Das hat konkrete Gründe:

- Rechenintensive Aufgaben (KI-Inferenz, Bildbearbeitung, Kompilierung) laufen lokal ohne Server-Bottleneck

- Offline-Fähigkeit: LibreOffice, Thunderbird und KeePassXC funktionieren ohne Netzverbindung

- Geringere Server-Hardware: Ein Raspberry Pi 4 oder kleiner VPS genügt für die Kerninfrastruktur

- Datenschutz bei sensiblen Aufgaben: Dokumente verlassen den Client nicht für serverseitiges Rendering

### A.4.3 Univention Nubus vs. eigenes IdM

Univention Nubus (ehemals UCS) ist eine vollständige Identity-Management-Plattform für Unternehmen und Behörden, die Active Directory funktional ersetzt.[^12] Für Schleswig-Holstein mit ~30.000 Nutzern und komplexen Rollenstrukturen ist das die richtige Wahl.

Im eigenen Konzept ist Keycloak als optionale Komponente vorgesehen — leichtgewichtiger, aber weniger umfangreich. Für kleine Gruppen genügt oft auch lokale Nutzerverwaltung (LDAP) direkt in Nextcloud und Dovecot.

### A.4.4 openDesk-Spezifika

openDesk (entwickelt durch ZenDiS im Auftrag von Bund und Ländern) ist der zentralisierte Ansatz für die öffentliche Verwaltung.[^13] Es kombiniert ähnliche Komponenten wie SH (Open-Xchange, Nextcloud, Collabora, OpenTalk), setzt aber auf eine gemeinsame, BSI-begleitete Plattform.

Kernunterschiede zum eigenen Konzept:

- openDesk ist für Kubernetes-Betrieb ausgelegt — komplexer Betrieb, aber skalierbar

- Collabora ist integriert (kein opt-in) — höhere Server-Last by default

- BSI-Begleitung und Zertifizierungspfad — relevant für Behörden, für private/NGO-Nutzung überdimensioniert

- Kein fat-client-Prinzip — lokale GPU- und CPU-Ressourcen werden nicht adressiert

## A.5 Einordnung und Fazit

Alle drei Ansätze sind technisch kompatibel, weil sie auf denselben offenen Protokollen aufbauen. Ein Wechsel zwischen den Ansätzen — etwa späterer Umstieg von eigenem Postfix/Dovecot auf Open-Xchange — wäre ohne Datenverlust möglich, da CalDAV-, CardDAV- und IMAP-Daten standardisiert sind.

Die Wahl des Ansatzes hängt primär von Skalierung und Betriebsmodell ab:

| **Szenario**                                      | **Empfohlener Ansatz**                                         |
|---------------------------------------------------|----------------------------------------------------------------|
| Landesverwaltung / Großbetrieb (\>1.000 Nutzer)   | Schleswig-Holstein-Modell (Open-Xchange + Dataport/Univention) |
| Öffentliche Verwaltung mit Compliance-Anforderung | openDesk (ZenDiS, BSI-begleitet)                               |
| KMU, NGO, Initiative, Einzelperson / Kleingruppe  | Eigenes Konzept (Nextcloud + Postfix/Dovecot, fat client)      |
| Hybride Umgebung (Mix aus Outlook-Nutzern + OSS)  | Open-Xchange oder Z-Push als ActiveSync-Brücke                 |

Das Schleswig-Holstein-Projekt ist dabei das bisher überzeugendste Praxisbeispiel, dass der OSS-Weg in großem Maßstab funktioniert — wirtschaftlich, technisch und politisch. Es bestätigt die Grundannahmen des eigenen Konzepts und liefert konkrete Migrationserfahrungen, von denen auch kleinere Initiativen profitieren können.

## A.6 Microsoft 365 im Architekturvergleich: Was bei der Migration abgedeckt wird

Microsoft 365 ist der de-facto-Standard, von dem migriert wird. Die folgende Übersicht zeigt, dass jede M365-Funktion im OSS-Stack vollständig abgedeckt ist — und ordnet beide Seiten in das Client-Server-Architekturmodell ein.

### A.6.1 Architekturprinzip: Fat Server vs. Thin Server

Microsoft 365 folgt einem **Fat-Server-Prinzip**: Anwendungslogik, Datenhaltung, Rendering und Geschäftslogik laufen auf Microsofts Cloud-Infrastruktur. Die Desktop-Anwendungen (Outlook, Word, Excel) sind zwar lokal installierte Fat Clients, aber zunehmend von permanenter Cloud-Anbindung abhängig — für Lizenzvalidierung, Co-Authoring, Echtzeit-Sync und KI-Funktionen (Copilot). Ohne Serververbindung ist die Funktionalität eingeschränkt; ohne Lizenzserver wird sie nach kurzer Toleranzzeit vollständig gesperrt.

Das eigene OSS-Konzept kehrt dieses Verhältnis um: Der **Server ist dünn** (Speicher, Sync, Routing), der **Client ist autonom** (Anwendungslogik, Rendering, KI-Inferenz). Offline-Fähigkeit ist kein Notfallmodus, sondern Normalzustand.

| **Architekturmerkmal**       | **Microsoft 365**                          | **OSS-Konzept (dieses Dokument)**          |
|------------------------------|--------------------------------------------|--------------------------------------------|
| Anwendungslogik              | Server (Cloud) + Client (lokal)            | Client (lokal)                             |
| Datenhaltung                 | Server (Cloud, US CLOUD Act)               | Server (eigene Infrastruktur, EU)          |
| Rendering (Dokumente)        | Server (Office Online) oder Client         | Client (LibreOffice)                       |
| KI-Funktionen                | Server (Copilot, Cloud-only)               | Client (Ollama, lokale GPU)                |
| Lizenzabhängigkeit           | Permanente Cloud-Validierung               | Keine (OSS)                                |
| Offline-Fähigkeit            | Eingeschränkt (Toleranzzeit)               | Vollständig                                |
| Vendor Lock-in               | Hoch (proprietäre Formate, Protokolle)     | Keiner (offene Standards)                  |
| Kontrolle über Daten         | Microsoft (Vertrag, nicht Technik)         | Betreiber (Technik und Vertrag)            |

### A.6.2 Funktionsabdeckung: M365 → OSS-Äquivalent

Bei einer Migration von Microsoft 365 auf den OSS-Stack geht keine Funktion verloren. Die folgende Tabelle zeigt die vollständige Abdeckung:

| **M365-Komponente**          | **Funktion**                        | **OSS-Äquivalent**                     | **Kapitel** |
|------------------------------|-------------------------------------|----------------------------------------|-------------|
| Exchange Online              | E-Mail, Kalender, Kontakte          | Postfix + Dovecot + Nextcloud          | 3, 5        |
| Outlook (Desktop)            | Mail-Client, Kalender-Client        | Thunderbird (+ TbSync)                 | 4           |
| Outlook (Web/OWA)            | Webmail                             | Nextcloud Mail / Roundcube             | 3           |
| Word / Excel / PowerPoint    | Office-Suite (lokal)                | LibreOffice                            | 4           |
| Office Online                | Office-Suite (Browser)              | Collabora Online (optional)            | 3           |
| OneDrive                     | Dateisync, Cloud-Speicher           | Nextcloud + Desktop-Client             | 3, 4        |
| SharePoint                   | Dokumentenmanagement, Intranet      | Nextcloud + Wiki (optional)            | 3           |
| Teams (Chat)                 | Messaging, Kanäle                   | Matrix / Element                       | 3, 4        |
| Teams (Video)                | Videokonferenz                      | Element / OpenTalk / Jitsi             | 3           |
| Azure AD / Entra ID          | Identitätsmanagement, SSO           | Keycloak / Univention Nubus            | 3, A.4      |
| Microsoft To Do / Planner    | Aufgabenverwaltung                  | Nextcloud Tasks / Deck                 | 5           |
| Power Automate               | Workflow-Automatisierung            | n8n / Node-RED (selbst-gehostet)       | —           |
| Microsoft Copilot            | KI-Assistent (Cloud)                | Ollama + Open WebUI (lokal)            | 4           |
| BitLocker + Windows Hello    | Verschlüsselung, Authentifizierung  | LUKS + TPM / FIDO2                     | D           |
| Microsoft Defender           | Endpoint-Schutz                     | ClamAV + rspamd + Firewall             | 3           |
| Intune                       | Geräteverwaltung (MDM)              | Ansible + Fleet (optional)             | —           |

Die Spalte *Kapitel* verweist auf die Stelle in diesem Dokument, an der die jeweilige OSS-Lösung beschrieben wird. Funktionen ohne Kapitelverweis (—) sind nicht Kernbestandteil dieses Konzepts, aber über etablierte OSS-Werkzeuge abdeckbar.

### A.6.3 Zusammenfassung

Der OSS-Stack deckt sämtliche Funktionen ab, die Microsoft 365 bietet. Die architektonischen Unterschiede — lokale Autonomie statt Cloud-Abhängigkeit, offene Protokolle statt proprietärer Formate, eigene Datenhaltung statt Fremdkontrolle — sind dabei keine Einschränkung, sondern der Grund für die Migration.

---

# Anhang B – Proxmox VE als Infrastrukturschicht für mittlere Organisationen

Dieser Anhang beschreibt, wie Proxmox Virtual Environment (VE) als optionale Infrastrukturschicht unter dem OSS-Plattformkonzept eingesetzt werden kann. Im Fokus steht der Anwendungsfall mittlerer Organisationen (50–500 Nutzer) mit Anforderungen an Ausfall­sicherheit und zentralisiertes Ressourcenmanagement.

## B.1 Warum Proxmox VE – Einordnung und Abgrenzung

### B.1.1 Was Proxmox VE ist

Proxmox VE ist eine Open-Source-Plattform für Server-Virtualisierung, die zwei Technologien vereint: KVM (Kernel-based Virtual Machine) für vollständige virtuelle Maschinen und LXC (Linux Containers) für leichtgewichtige Container.[^14] Beides wird über eine einheitliche Web-Oberfläche und API verwaltet. Proxmox ist in Debian GNU/Linux integriert und vollständig quelloffen (AGPLv3).

### B.1.2 Abgrenzung zu OpenStack und VMware

| **Kriterium**       | **Proxmox VE**           | **OpenStack**             | **VMware vSphere** |
|---------------------|--------------------------|---------------------------|--------------------|
| Lizenz              | AGPLv3 (OSS)             | Apache 2.0 (OSS)          | Proprietär         |
| Mindest-Hardware    | 1 Node möglich, HA ab 3  | 5+ Nodes empfohlen        | 1 Node möglich     |
| Betriebskomplexität | Mittel                   | Sehr hoch                 | Mittel–hoch        |
| Zielgröße           | KMU bis mittelgroße Org. | Große RZ / Cloud-Provider | Enterprise         |
| Eingebautes HA      | Ja (ab 3 Nodes)          | Ja (komplex)              | Ja (teuer)         |
| Eingebautes Backup  | Ja (Proxmox BS)          | Nein (separat)            | Ja (teuer)         |
| Lernkurve           | Moderat                  | Steil                     | Moderat–steil      |
| Community           | Aktiv, dt. Hersteller    | Sehr groß                 | Kommerziell        |

**Proxmox ist der direkte OSS-Ersatz für VMware vSphere.** Nach der Broadcom-Übernahme von VMware 2023 und den massiven Lizenzpreiserhöhungen haben viele Organisationen — auch öffentliche Einrichtungen — auf Proxmox migriert.[^15]

### B.1.3 Wann Proxmox sinnvoll ist

Proxmox als Infrastrukturschicht lohnt sich, wenn mindestens zwei der folgenden Bedingungen zutreffen:

- Mehrere physische Server sollen als einheitliche Ressource verwaltet werden

- Ausfallsicherheit ist gefordert (HA — Dienste überleben den Ausfall eines Knotens)

- Verschiedene Dienste sollen isoliert und unabhängig voneinander betrieben werden

- Live-Migration von Diensten ohne Downtime ist gewünscht

- Zentrale Backup- und Snapshot-Verwaltung über alle Dienste hinweg

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><strong>Nicht empfohlen für:</strong></p>
<p>Einzelinstallationen auf einem Server (dort genügen Docker/Podman direkt). Sehr kleine Gruppen (&lt;10 Nutzer) ohne eigene Hardware. Umgebungen ohne Linux-Admin-Kenntnisse im Team.</p></td>
</tr>
</tbody>
</table>

## B.2 Cluster-Architektur und Ausfallsicherheit

### B.2.1 Das Quorum-Prinzip

Proxmox-Cluster basieren auf Corosync als Cluster-Engine.[^16] Corosync verwendet ein **Quorum-Modell**: Ein Knoten darf nur dann Dienste betreiben oder übernehmen, wenn er weiß, dass die Mehrheit der Cluster-Knoten erreichbar ist. Das verhindert das sogenannte „Split-Brain"-Problem, bei dem zwei isolierte Knoten unabhängig voneinander dieselben Daten schreiben.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><strong>Minimum für echte Ausfallsicherheit: 3 Nodes</strong></p>
<p>Mit 3 Nodes hat der Cluster auch beim Ausfall eines Knotens noch Quorum (2 von 3). Mit 2 Nodes kann beim Ausfall eines Knotens kein Quorum gebildet werden — der verbleibende Knoten friert ein, um Datenkorrruption zu vermeiden. Als kostengünstige Alternative zu einem dritten vollwertigen Server kann ein Proxmox QDevice (z.B. Raspberry Pi) als Quorum-Zeuge dienen.</p></td>
</tr>
</tbody>
</table>

### B.2.2 Referenzarchitektur: 3-Node-Cluster

Die folgende Architektur ist für eine mittlere Organisation mit 50–500 Nutzern ausgelegt. Sie bietet Ausfallsicherheit bei Ausfall eines beliebigen Knotens.

| **Knoten**         | **Rolle**                      | **Empfohlene Hardware**         | **Besonderheit**                  |
|--------------------|--------------------------------|---------------------------------|-----------------------------------|
| Node 1 (pve-01)    | Aktiver Knoten, Cluster-Master | 8–16 Core, 64–128 GB RAM, NVMe  | Primärer Betrieb aller HA-Dienste |
| Node 2 (pve-02)    | Aktiver Knoten, Failover       | 8–16 Core, 64–128 GB RAM, NVMe  | Übernimmt bei Ausfall von Node 1  |
| Node 3 (pve-03)    | Aktiver Knoten / Ceph-OSD      | 8+ Core, 32–64 GB RAM, HDD+NVMe | Ceph-Storage, leichtere Last      |
| QDevice (optional) | Quorum-Zeuge                   | Raspberry Pi 4                  | Nur bei 2 Vollknoten nötig        |

### B.2.3 Netzwerktopologie

Für produktiven Cluster-Betrieb sind drei logisch getrennte Netzwerke empfohlen:[^17]

| **Netz**                | **Zweck**                                  | **Typische Adresse** | **Anforderung**                        |
|-------------------------|--------------------------------------------|----------------------|----------------------------------------|
| Management-Netz         | Proxmox Web-UI, SSH, Cluster-Kommunikation | 10.0.0.0/24          | 1 GbE genügt                           |
| Corosync / Cluster-Netz | Heartbeat, Quorum (dediziert!)             | 10.0.1.0/24          | Dediziertes Interface, niedrige Latenz |
| Storage-Netz            | Ceph-Replikation zwischen Nodes            | 10.0.2.0/24          | 10 GbE empfohlen                       |
| VM/CT-Netz              | Produktiver Traffic der Dienste            | 10.0.10.0/24         | 1–10 GbE je Last                       |

Das Corosync-Netz muss physisch oder logisch vom Management-Netz getrennt sein. Ein gemeinsames Netz führt zu Timeouts unter Last und damit zu unnötigen Failover-Ereignissen.

## B.3 Storage-Konzept: Ceph als verteilter OSS-Speicher

Proxmox VE integriert Ceph nativ — Installation, Verwaltung und Monitoring direkt über die Proxmox-Oberfläche.[^18] Ceph ist ein verteiltes Speichersystem: Daten werden in Echtzeit über mehrere Nodes repliziert. Der Ausfall eines Nodes bedeutet keinen Datenverlust.[^19]

### B.3.1 Ceph-Komponenten im Überblick

| **Komponente**              | **Funktion**                    | **Minimum**               |
|-----------------------------|---------------------------------|---------------------------|
| OSD (Object Storage Daemon) | Verwaltet physische Festplatten | 3 OSDs (je eine pro Node) |
| MON (Monitor)               | Cluster-Karte, Quorum           | 3 MONs (je Node einer)    |
| MGR (Manager)               | Statistiken, Dashboard, Plugins | 1–2 MGRs                  |
| MDS (Metadata Server)       | Nur für CephFS (Dateisystem)    | Optional                  |

### B.3.2 Replikation und Ausfallsicherheit

Ceph repliziert standardmäßig jeden Datenblock dreifach (Replication Factor 3). Der Cluster bleibt lesbar und schreibbar, solange mindestens 2 von 3 Nodes verfügbar sind. Bei NVMe als OSD-Medium für die Ceph-Journal-Disks (WAL/DB) ist die Performance auch unter Replikation produktionstauglich.

### B.3.3 Storage-Typen für verschiedene Dienste

| **Storage-Typ**                | **Ceph-Pendant**         | **Geeignet für**                          |
|--------------------------------|--------------------------|-------------------------------------------|
| Block-Storage (RBD)            | Ceph RBD                 | VM-Disks, Datenbank-Volumes               |
| Objektspeicher (S3-kompatibel) | Ceph RGW (RADOS Gateway) | Nextcloud-Primärspeicher, Backups, Restic |
| Dateisystem                    | CephFS                   | Shared Volumes zwischen Containern        |

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><strong>Praxistipp: Nextcloud auf Ceph RGW</strong></p>
<p>Nextcloud unterstützt S3-kompatiblen Objektspeicher als Primär-Storage nativ. Mit Ceph RGW als Backend liegen Nextcloud-Daten automatisch dreifach repliziert auf allen Cluster-Nodes — ohne separates NAS oder SAN.</p></td>
</tr>
</tbody>
</table>

## B.4 Dienste-Verteilung: LXC-Container vs. KVM-VM

Proxmox bietet zwei Virtualisierungsebenen: LXC-Container teilen den Kernel des Hosts und sind ressourcenschonender; KVM-VMs haben einen vollständig isolierten Kernel und eignen sich für Dienste, die Kernel-Module benötigen oder maximale Isolation erfordern.[^20]

| **Dienst**            | **Typ**      | **Begründung**                                | **HA**             |
|-----------------------|--------------|-----------------------------------------------|--------------------|
| Nextcloud             | LXC          | PHP-Anwendung, kein Kernel-Modul nötig        | Ja                 |
| Postfix + Dovecot     | LXC          | Standard-Linux-Dienste, leichtgewichtig       | Ja (aktiv/passiv)  |
| Matrix / Synapse      | LXC          | Python-Dienst, kein Sonderbedarf              | Ja                 |
| WireGuard VPN         | LXC          | Kernel-Modul im Host verfügbar                | Ja                 |
| Keycloak (IdM/SSO)    | LXC oder KVM | Java-Dienst; KVM bei höherem Isolationsbedarf | Ja                 |
| Collabora Online      | KVM          | Rendering-Engine, eigener Kernel-Stack        | Ja (optional)      |
| Proxmox Backup Server | KVM          | Eigenständiges System, besser isoliert        | Nein (Backup-Tier) |
| Monitoring (Checkmk)  | LXC          | Leichtgewichtig, überwacht alle anderen       | Nein (unkritisch)  |
| Gitea (optional)      | LXC          | Go-Dienst, minimale Ressourcen                | Ja                 |
| OpenTalk / Jitsi      | KVM          | Medienprozessierung, hoher CPU-Bedarf         | Optional           |

### B.4.1 HA-Konfiguration

Proxmox HA Manager überwacht laufende VMs und Container.[^21] Beim Ausfall eines Knotens werden HA-aktivierte Dienste automatisch auf einem der verbleibenden Knoten neu gestartet. Typische Failover-Zeit: 30–90 Sekunden.

| **HA-Modus**                 | **Dienste**                          | **Failover-Zeit** | **Bemerkung**                               |
|------------------------------|--------------------------------------|-------------------|---------------------------------------------|
| Aktiv/Passiv (HA-restart)    | Nextcloud, Dovecot, Matrix, Keycloak | 30–90 Sek.        | Standard-HA in Proxmox                      |
| Aktiv/Aktiv (Load Balancing) | Nginx Reverse Proxy, WireGuard       | 0 Sek.            | Erfordert externe LB oder VRRP (keepalived) |
| Kalt-Standby                 | Collabora, OpenTalk                  | 2–5 Min.          | Nur bei Bedarf gestartet                    |

## B.5 Backup-Konzept mit Proxmox Backup Server

Proxmox Backup Server (PBS) ist eine dedizierte OSS-Backup-Lösung, die eng mit Proxmox VE integriert ist.[^22] PBS unterstützt inkrementelle, deduplizierte Backups mit clientseitiger Verschlüsselung.

| **Eigenschaft**     | **Beschreibung**                                                                            |
|---------------------|---------------------------------------------------------------------------------------------|
| Deduplizierung      | Nur geänderte Datenblöcke werden übertragen — massive Platzeinsparung bei täglichen Backups |
| Verschlüsselung     | AES-256 clientseitig — PBS-Server sieht nur verschlüsselte Blöcke                           |
| Verifikation        | Automatische Integritätsprüfung aller Backups                                               |
| Retention-Policy    | Konfigurierbar: z.B. 7 tägliche, 4 wöchentliche, 12 monatliche Snapshots                    |
| Offsite-Replikation | Sync zu zweitem PBS oder S3-kompatiblem Storage (z.B. Hetzner Storage Box)                  |

Empfohlene Backup-Strategie für mittlere Organisationen (3-2-1-Regel):

- Backup 1: PBS lokal auf dedizierter VM im Cluster (schnelle Wiederherstellung)

- Backup 2: PBS-Replikat auf separatem physischen Gerät (anderer Brandabschnitt)

- Backup 3: Offsite — Hetzner Storage Box oder zweiter Standort via PBS-Sync

## B.6 Monitoring und Betrieb

Proxmox VE bringt ein eingebautes Monitoring-Dashboard mit — ausreichend für Übersicht und Alerting auf Node-Ebene. Für tieferes Service-Monitoring empfiehlt sich Checkmk Raw Edition (OSS).[^23]

| **Ebene**          | **Tool**                  | **Funktion**                                             |
|--------------------|---------------------------|----------------------------------------------------------|
| Infrastruktur      | Proxmox VE Dashboard      | CPU, RAM, Storage, Cluster-Status, HA-Events             |
| Service-Monitoring | Checkmk Raw Edition       | Dienst-Verfügbarkeit, Schwellwert-Alarme, Log-Auswertung |
| Metriken / Graphen | Grafana + InfluxDB        | Langzeit-Metriken, Kapazitätsplanung                     |
| Log-Aggregation    | Loki + Grafana            | Zentrale Log-Auswertung aller Container und VMs          |
| Alerting           | Checkmk oder Alertmanager | E-Mail / Matrix-Benachrichtigung bei Ausfall             |

## B.7 Gesamtbild: OSS-Stack mit Proxmox-Infrastrukturschicht

Die folgende Tabelle zeigt den vollständigen Stack einer mittleren Organisation mit Proxmox als Infrastrukturschicht:

| **Schicht**   | **Komponente**            | **Produkt**                       | **Betrieb**                 |
|---------------|---------------------------|-----------------------------------|-----------------------------|
| Infrastruktur | Hypervisor / Cluster      | Proxmox VE                        | 3-Node-Cluster, HA          |
| Infrastruktur | Verteilter Storage        | Ceph (integriert)                 | Replikation über alle Nodes |
| Infrastruktur | Backup                    | Proxmox Backup Server             | PBS-VM im Cluster           |
| Infrastruktur | Monitoring                | Checkmk + Grafana                 | LXC im Cluster              |
| Dienste       | Dateiablage / Kalender    | Nextcloud                         | LXC, HA, Ceph RGW           |
| Dienste       | E-Mail                    | Postfix + Dovecot + rspamd        | LXC, HA aktiv/passiv        |
| Dienste       | Kommunikation             | Matrix / Synapse + Element        | LXC, HA                     |
| Dienste       | VPN / Zugang              | WireGuard + nginx                 | LXC, aktiv/aktiv (VRRP)     |
| Dienste       | IdM / SSO                 | Keycloak                          | LXC oder KVM, HA            |
| Dienste       | Co-Editing (optional)     | Collabora Online                  | KVM, Kalt-Standby           |
| Dienste       | Videokonferenz (optional) | OpenTalk / Jitsi                  | KVM, bei Bedarf             |
| Clients       | Desktop                   | LibreOffice, Thunderbird, Element | Fat Client, lokal           |
| Clients       | Mobile                    | DAVx⁵, Element, WireGuard         | Android / iOS               |

## B.8 Aufwand-Nutzen-Bewertung

| **Kriterium**         | **Flat-Server-Betrieb**         | **Mit Proxmox-Cluster**                         |
|-----------------------|---------------------------------|-------------------------------------------------|
| Initiale Einrichtung  | Gering (1–2 Tage)               | Mittel (3–5 Tage)                               |
| Ausfallsicherheit     | Keine (Single Point of Failure) | Hoch (Node-Ausfall transparent)                 |
| Live-Migration        | Nicht möglich                   | Ja (ohne Downtime)                              |
| Ressourceneffizienz   | Schlecht (fixe Zuweisung)       | Gut (dynamische Zuweisung)                      |
| Backup                | Manuell / per Skript            | Integriert (PBS)                                |
| Skalierung            | Neuer Server = manuell          | Node hinzufügen = wenige Klicks                 |
| Admin-Aufwand laufend | Mittel                          | Mittel (höhere Komplexität, aber bessere Tools) |
| Empfehlung ab         | —                               | 3+ Server, Ausfallsicherheit gefordert          |

Fazit: Proxmox VE lohnt sich für mittlere Organisationen, sobald mehr als zwei physische Server vorhanden sind und Ausfallsicherheit eine Anforderung ist. Der initiale Mehraufwand bei der Einrichtung amortisiert sich schnell durch reduzierten operativen Aufwand, integriertes Backup und die Möglichkeit, Wartungsarbeiten ohne Downtime durchzuführen.

---

# Anhang C – Kostenvergleich europäischer IaaS-Anbieter

Dieser Anhang vergleicht europäische IaaS-Anbieter für den Betrieb des OSS-Plattformstacks. Alle Dienste laufen vollständig in der Cloud; Administration erfolgt per SSH vom eigenen Rechner. Referenzgröße: 100 Nutzer. Mindestanforderung: BSI C5 Typ 2.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><strong>Hinweis zu den Preisen</strong></p>
<p>Hetzner-Preise exkl. MwSt., nach Preisanpassung 15. Juni 2026. OVHcloud-Preise inkl. MwSt. (19%). Open Telekom Cloud: stundenbasiertes Modell ohne fixe Monatspreise – daher qualitative Einordnung statt Zahlenvergleich. Alle Angaben sind Richtwerte – aktuelle Preislisten der Anbieter konsultieren.</p></td>
</tr>
</tbody>
</table>

## C.1 Ressourcen-Sizing für 100 Nutzer

Grundlage ist der vollständige OSS-Stack aus dem Hauptkonzept (Kapitel 3–5).

| **Dienst**                       | **vCPU** | **RAM** | **Storage** | **Bemerkung**                             |
|----------------------------------|----------|---------|-------------|-------------------------------------------|
| Nextcloud (App + DB)             | 4        | 8 GB    | 500 GB      | SSD; Nutzdaten als Objektspeicher separat |
| Postfix + Dovecot + rspamd       | 2        | 4 GB    | 200 GB      | Mailstore                                 |
| Matrix / Synapse                 | 2        | 4 GB    | 100 GB      | Nachrichtenhistorie wächst                |
| WireGuard + nginx                | 1        | 1 GB    | 20 GB       | Leichtgewichtig                           |
| Keycloak (SSO)                   | 2        | 4 GB    | 20 GB       | Java-Dienst, RAM-intensiv                 |
| Monitoring (Checkmk)             | 1        | 2 GB    | 50 GB       |                                           |
| Backup-Instanz (restic + rclone) | 1        | 2 GB    | 50 GB       | Offsite via Storage Box                   |
| Reserve / Headroom (20%)         | 3        | 5 GB    | –           | Last-Peaks und Updates                    |
| Σ Gesamt (gerundet)              | 16       | 30 GB   | ~950 GB     | \+ Objektspeicher für Nutzdaten           |

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><strong>Collabora Online (optional)</strong></p>
<p>Falls Co-Editing benötigt wird: +4 vCPU, +8 GB RAM, +50 GB Storage auf separater VM. Nicht in den Basiskosten enthalten.</p></td>
</tr>
</tbody>
</table>

## C.2 Europäische IaaS-Anbieter im Überblick

Alle genannten Anbieter sind europäisch, DSGVO-konform und verfügen über BSI C5 Typ 2.[^24]

| **Anbieter**       | **Hauptsitz**    | **RZ Deutschland**    | **BSI C5 Typ 2** | **CLOUD Act** | **Basis**        |
|--------------------|------------------|-----------------------|------------------|---------------|------------------|
| Hetzner Cloud      | Gunzenhausen, DE | Falkenstein, Nürnberg | ✓ (März 2026)    | Nein          | Eigenentwicklung |
| OVHcloud           | Roubaix, FR      | Strasbourg            | ✓                | Nein          | OpenStack        |
| Open Telekom Cloud | Bonn, DE         | Biere, Magdeburg      | ✓                | Nein          | OpenStack        |
| IONOS Cloud        | Montabaur, DE    | Frankfurt, Berlin     | ✓ (Typ 1/2)      | Nein          | Eigenentwicklung |

**Hetzner** erhält im März 2026 das BSI C5:2020 Typ 2 Testat und ist zusätzlich als KRITIS-Betreiber nach §8a BSIG zertifiziert.[^25]

**OVHcloud und Open Telekom Cloud** betreiben ihre Infrastruktur auf OpenStack-Basis. Das ist für den Alltagsbetrieb von VMs transparent — relevant wird es nur bei direkter API-Nutzung (Terraform, Swift-Objektspeicher). OVHcloud erklärt explizit, nicht dem US CLOUD Act zu unterliegen.[^26]

## C.3 OpenStack-Unterstützung: Wann relevant, wann nicht?

Eine häufige Frage: Bringt ein OpenStack-basierter Anbieter (OVHcloud, OTC) einen praktischen Vorteil gegenüber Hetzner?

Für den beschriebenen Anwendungsfall — OSS-Dienste auf Cloud-VMs, Administration per SSH — ist die Antwort: Nein, kein direkter Vorteil im Betrieb.

| **Szenario**                               | **OpenStack-Vorteil?** | **Bemerkung**                                                    |
|--------------------------------------------|------------------------|------------------------------------------------------------------|
| VMs per SSH verwalten                      | Nein                   | Identisch auf allen Anbietern                                    |
| Dienste installieren und betreiben         | Nein                   | Betriebssystem-Ebene, unabhängig von Plattform                   |
| Objektspeicher (S3-kompatibel)             | Bedingt                | OVHcloud Swift / OTC OBS: S3-API verfügbar, aber Hetzner auch    |
| Infrastruktur per Terraform provisionieren | Ja                     | OpenStack-Provider weit verbreitet; Hetzner hat eigenen Provider |
| Anbieter wechseln ohne Lock-in             | Ja                     | VM-Images portierbar über OpenStack-API; bei Hetzner manuell     |
| Managed Kubernetes / Datenbanken           | Ja (OTC)               | OTC bietet umfangreichere Managed Services                       |
| Compliance öffentliche Verwaltung / KRITIS | Bedingt                | OTC stärker positioniert; Hetzner mit C5 nun gleichwertig        |

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><strong>Fazit OpenStack</strong></p>
<p>Für den SSH-basierten Betrieb von Nextcloud, Postfix, Matrix und Co. ist OpenStack als Plattformunterlage irrelevant. Es wird relevant sobald Infrastruktur programmatisch verwaltet wird (Terraform/Heat), Managed Services genutzt werden (Datenbanken, Kubernetes) oder Portierbarkeit zwischen Anbietern auf API-Ebene gewünscht wird.</p></td>
</tr>
</tbody>
</table>

## C.4 Preisvergleich: Hetzner vs. OVHcloud

Open Telekom Cloud wird nicht in den direkten Zahlenvergleich einbezogen, da das Preismodell stundenbasiert ohne fixe Monatstarife funktioniert und ein fairer Vergleich einen eigenen Preisrechner-Durchlauf erfordert. OTC positioniert sich preislich über Hetzner und OVHcloud, bietet dafür umfangreichere Managed Services und stärkere Enterprise-SLAs.[^27][^28]

### C.4.1 Hetzner Cloud

Preise exkl. MwSt., nach Preisanpassung 15. Juni 2026.[^29]

| **VM / Dienst**                   | **Typ**      | **vCPU** | **RAM** | **€/Monat** |
|-----------------------------------|--------------|----------|---------|-------------|
| Nextcloud App+DB                  | CX43         | 4        | 8 GB    | 15,99       |
| Postfix + Dovecot + rspamd        | CX33         | 4        | 8 GB    | 8,49        |
| Matrix / Synapse                  | CX33         | 4        | 8 GB    | 8,49        |
| WireGuard + nginx                 | CX23         | 2        | 4 GB    | 5,49        |
| Keycloak SSO                      | CX33         | 4        | 8 GB    | 8,49        |
| Monitoring + Backup-Instanz       | CX23         | 2        | 4 GB    | 5,49        |
| Objektspeicher (500 GB Volume)    | Block Volume | –        | –       | ~25,00      |
| Backup offsite (Storage Box BX30) | Storage Box  | –        | –       | 3,81        |
| Floating IP + Traffic             | –            | –        | –       | ~2,00       |

|                                          |                 |
|------------------------------------------|-----------------|
| **Σ Hetzner Cloud Gesamt (exkl. MwSt.)** | **~83 €/Monat** |

*Inkl. 19% MwSt.: ~ 99 €/Monat — **0,99 €/Nutzer/Monat***

### C.4.2 OVHcloud

Vergleichbarer Stack, Preise inkl. MwSt.[^30]

| **VM / Dienst**                        | **Typ** | **vCPU** | **RAM** | **€/Monat (inkl. MwSt.)** |
|----------------------------------------|---------|----------|---------|---------------------------|
| Nextcloud App+DB                       | VPS-3   | 4        | 8 GB    | ~12,38                    |
| Postfix + Dovecot + rspamd             | VPS-2   | 3        | 4 GB    | ~8,58                     |
| Matrix / Synapse                       | VPS-2   | 3        | 4 GB    | ~8,58                     |
| WireGuard + nginx                      | VPS-1   | 2        | 2 GB    | ~4,53                     |
| Keycloak SSO                           | VPS-2   | 3        | 4 GB    | ~8,58                     |
| Monitoring + Backup-Instanz            | VPS-1   | 2        | 2 GB    | ~4,53                     |
| Objektspeicher (Object Storage 500 GB) | –       | –        | –       | ~12,50                    |
| Backup offsite (Object Storage)        | –       | –        | –       | ~6,00                     |

|                                     |                 |
|-------------------------------------|-----------------|
| **Σ OVHcloud Gesamt (inkl. MwSt.)** | **~66 €/Monat** |

***0,66 €/Nutzer/Monat***

### C.4.3 Direktvergleich

| **Kriterium**                  | **Hetzner Cloud**       | **OVHcloud**        | **Open Telekom Cloud**                     |
|--------------------------------|-------------------------|---------------------|--------------------------------------------|
| Monatliche Kosten (100 Nutzer) | ~99 € (inkl. MwSt.)     | ~66 € (inkl. MwSt.) | Höher (Enterprise-Positionierung)          |
| Kosten/Nutzer/Monat            | ~0,99 €                 | ~0,66 €             | n/a (stundenbasiert)                       |
| Traffic inklusive              | 20 TB/VM                | Unlimitiert         | Separat abgerechnet                        |
| Egress-Kosten                  | Keine (inkl.)           | Keine (inkl.)       | Ja, separat                                |
| Objektspeicher                 | Block Volume / S3       | OVH Object Storage  | OBS (S3-kompatibel)                        |
| SLA Verfügbarkeit              | 99,9%                   | 99,9%               | 99,95% (höher)                             |
| Support                        | Community / Ticket      | Community / Ticket  | 24/7 inkl.                                 |
| BSI C5 Typ 2                   | ✓ (März 2026)           | ✓                   | ✓                                          |
| CLOUD Act Risiko               | Nein                    | Nein                | Nein                                       |
| OpenStack-Basis                | Nein (Eigenentwicklung) | Ja                  | Ja                                         |
| Zielgruppe                     | KMU, Entwickler, NGO    | KMU bis Mittelstand | Öffentliche Verwaltung, KRITIS, Enterprise |

## C.5 Versteckte Kosten und Fallstricke

| **Kostenart**             | **Hetzner**      | **OVHcloud**  | **OTC**                  |
|---------------------------|------------------|---------------|--------------------------|
| Egress / Outbound-Traffic | Inkl. (20 TB/VM) | Inkl.         | Separat (0,02–0,05 €/GB) |
| Floating / Elastic IP     | ~2 €/Monat       | Inkl.         | Separat                  |
| Snapshots / Backups       | 20% Aufpreis     | Täglich inkl. | Separat                  |
| Load Balancer             | Separat ab ~6 €  | Separat       | Separat                  |
| IPv4-Adresse              | Ab 0,50 €/Monat  | Inkl.         | Separat                  |
| Support-Eskalation        | Ticket           | Ticket        | 24/7 inkl.               |

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><strong>Wichtig: Egress bei OTC</strong></p>
<p>Bei der Open Telekom Cloud werden ausgehende Daten separat abgerechnet. Bei datenintensiven Diensten (Videokonferenz, große Dateiablage) kann das die Gesamtkosten deutlich erhöhen. Vor Buchung den Preisrechner mit realistischen Traffic-Annahmen durchrechnen.</p></td>
</tr>
</tbody>
</table>

## C.6 Einordnung: Microsoft 365 als Vergleichsmaßstab

Microsoft 365 Business Standard kostet Stand 2026 ca. 12,50 €/Nutzer/Monat (exkl. MwSt.), also ~1.488 €/Jahr für 100 Nutzer.[^31]

| **Lösung**                      | **€/Nutzer/Monat** | **€/Jahr (100 Nutzer)** | **Einsparung ggü. M365** | **Souveränität**    |
|---------------------------------|--------------------|-------------------------|--------------------------|---------------------|
| Microsoft 365 Business Standard | ~12,50 €           | ~15.000 €               | –                        | Nein (US CLOUD Act) |
| Hetzner Cloud + OSS-Stack       | ~0,99 €            | ~1.188 €                | ~92%                     | Ja (BSI C5 Typ 2)   |
| OVHcloud + OSS-Stack            | ~0,66 €            | ~792 €                  | ~95%                     | Ja (kein CLOUD Act) |
| Open Telekom Cloud + OSS-Stack  | ~2–5 €\*           | ~2.400–6.000 €\*        | ~60–84%                  | Ja (BSI C5 Typ 2)   |

\* OTC-Schätzung auf Basis des stundenbasierten Preismodells; stark abhängig von Laufzeit und gebuchten Managed Services.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><strong>Einsparung gegenüber Microsoft 365</strong></p>
<p>Selbst der teurere OTC-Betrieb ist deutlich günstiger als Microsoft 365 — bei vollständiger Datensouveränität und ohne CLOUD-Act-Risiko. Die jährliche Einsparung gegenüber M365 kann direkt in Admin-Kapazität investiert werden.</p></td>
</tr>
</tbody>
</table>

## C.7 Empfehlung

| **Szenario**                           | **Empfohlener Anbieter** | **Begründung**                                                                |
|----------------------------------------|--------------------------|-------------------------------------------------------------------------------|
| Einstieg / Pilotbetrieb                | Hetzner Cloud            | Günstigster Einstieg, BSI C5 Typ 2, sofort skalierbar, deutsches Unternehmen  |
| Kostenoptimierung bei stabilem Betrieb | OVHcloud                 | Günstiger als Hetzner, Traffic inkl., OpenStack-Basis für spätere API-Nutzung |
| Wachstum mit Managed Services          | OVHcloud oder OTC        | Managed Kubernetes, Datenbanken, höhere SLAs verfügbar                        |
| Gesundheit / KRITIS (§393 SGB V)       | OTC oder Hetzner         | Beide erfüllen C5 Typ 2 Pflicht; OTC mit stärkerem Enterprise-Support         |
| Öffentliche Verwaltung / Compliance    | OTC                      | Stärkste KRITIS- und Behörden-Positionierung, GAIA-X ready                    |

Für eine mittlere Organisation mit 100 Nutzern ist Hetzner Cloud der pragmatische Einstieg: niedrigster Aufwand, transparente Preise, BSI C5 Typ 2 erfüllt. OVHcloud ist die kostengünstigere Alternative mit OpenStack-Unterlage für Organisationen die später Terraform oder Managed Services nutzen möchten. Open Telekom Cloud ist die richtige Wahl sobald Enterprise-SLAs, Managed Services oder eine besonders starke KRITIS-Compliance-Positionierung gefragt sind.

---

# Anhang D – TPM-gestützte Authentifizierung als Alternative zu Shared-Secret-Verfahren

## D.1 Kernproblem klassischer 2FA (TOTP)

TOTP-Verfahren (RFC 6238) erfordern einen identischen Seed auf Client und Server – ein „SharedSecret", das zwangsläufig auf beiden Seiten im Klartext vorliegen muss, um den HMAC unabhängig berechnen zu können. Das ist kein Implementierungsfehler, sondern eine architektonische Eigenschaft des Verfahrens.

## D.2 TPM-basierter Ansatz (Windows Hello, FIDO2/WebAuthn)

Im Gegensatz dazu erzeugt das TPM ein asymmetrisches Schlüsselpaar lokal im Chip. Der private Schlüssel verlässt das Hardware-Modul nie – auch nicht verschlüsselt. Der Server (z. B. Microsoft) speichert nur den öffentlichen Schlüssel. PIN/Biometrie dienen lediglich dazu, eine lokale Autorisierungs-Session freizuschalten, die das TPM zur Signatur einer Challenge veranlasst. Dadurch entfällt das grundsätzliche Shared-Secret-Problem.

## D.3 Schutzmechanismen

- **PCR-Bindung (Measured Boot):** Schlüssel können an einen bestimmten, kryptographisch verketteten Boot-Zustand gesealed werden. Ein abweichender Bootpfad (z. B. fremdes Live-Linux) verändert die PCR-Werte, wodurch die Freigabe fehlschlägt.

- **PreBoot-PIN:** Verhindert automatische Schlüsselfreigabe allein durch passende PCR-Werte; ohne korrekte Eingabe bleibt der Schlüssel versiegelt, auch bei physischem Zugriff über externes OS.

- **Anti-Hammering:** Nach begrenzter Zahl von Fehlversuchen sperrt sich der Chip selbst mit exponentiell steigender Lockout-Zeit – wirksam unabhängig vom verwendeten Betriebssystem.

## D.4 Grenzen

TPM-Schutz ist gerätegebunden. Schwach konfigurierte Setups (TPM-only ohne PIN, ältere Firmware) sind nicht immun gegen Bus-Sniffing-Angriffe bei physischem Zugriff – konkret nachgewiesen u. a. bei BitLocker-Konfigurationen ohne PreBoot-PIN.

## D.5 Linux-Äquivalente

tpm2-tools, systemd-cryptenroll, clevis/tang bieten vergleichbare Funktionalität (Sealing, PCR-Bindung, automatisches Entschlüsseln). Für plattformneutrale, herstellerunabhängige Authentifizierung mit identischem Sicherheitsmodell (nicht-exportierbarer privater Schlüssel) bieten sich FIDO2-Hardware-Token an – im Gegensatz zu Windows Hello, das proprietär an die Microsoft-Plattform gebunden ist.

---

# Quellenverzeichnis

[^1]: Pressemitteilung Schleswig-Holstein, 2. Oktober 2025: [Mailsystem der Landesverwaltung komplett auf Open Source umgestellt](https://www.schleswig-holstein.de/DE/landesregierung/ministerien-behoerden/I/Presse/PI/2025/cds/251006_cds_ox)

[^2]: Open-Xchange Produktseite: [OX App Suite](https://www.open-xchange.com/products/ox-app-suite/)

[^3]: Univention Nubus (UCS): [univention.de/produkte/nubus/](https://www.univention.de/produkte/nubus/)

[^4]: OpenTalk (Videokonferenz, OSS): [opentalk.eu](https://opentalk.eu/)

[^5]: Pressemitteilung Schleswig-Holstein, Dezember 2025: [Digitale Souveränität ist möglich und wirtschaftlich](https://www.schleswig-holstein.de/DE/landesregierung/ministerien-behoerden/I/_startseite/Artikel2025/IV/251204_cds_digitale_souveraenitaet)

[^6]: Dataport AöR – IT-Dienstleister der norddeutschen Länder: [dataport.de](https://www.dataport.de/)

[^7]: Einsparungen und wirtschaftliche Bilanz der OSS-Migration in SH: vgl. [^5]

[^8]: HiSolutions Research, Dezember 2025: [Positive Stimmung bei der Open-Source-Migration in Schleswig-Holstein](https://research.hisolutions.com/2025/12/positive-stimmung-bei-der-open-source-migration-in-schleswig-holstein/)

[^9]: RFC 5545 – iCalendar: [datatracker.ietf.org/doc/html/rfc5545](https://datatracker.ietf.org/doc/html/rfc5545)

[^10]: Nextcloud CalDAV/CardDAV Dokumentation: [docs.nextcloud.com – Groupware](https://docs.nextcloud.com/server/latest/user_manual/en/groupware/index.html)

[^11]: Z-Push (ActiveSync Bridge, AGPL): [z-push.org](https://z-push.org/)

[^12]: BFH-Interview mit Digitalisierungsminister Schrödter, März 2026: [Open Source als Schlüssel zur Souveränität in der digitalen öffentlichen Infrastruktur](https://www.bfh.ch/de/aktuell/storys/2026/open-source-fuer-digitale-souveraenitaet/)

[^13]: openDesk – ZenDiS: [zendis.de/opendesk](https://zendis.de/opendesk)

[^14]: Proxmox VE Dokumentation: [pve.proxmox.com/wiki/Main_Page](https://pve.proxmox.com/wiki/Main_Page)

[^15]: Proxmox vs. VMware vSphere – Vergleich nach Broadcom-Übernahme: [pve.proxmox.com – Comparison](https://pve.proxmox.com/wiki/Comparison_with_other_products)

[^16]: Corosync Cluster Engine (Quorum): [corosync.github.io](https://corosync.github.io/corosync/)

[^17]: Proxmox SDN (Software Defined Networking): [pve.proxmox.com/wiki/Software_Defined_Network](https://pve.proxmox.com/wiki/Software_Defined_Network)

[^18]: Ceph Storage – Proxmox Integration: [pve.proxmox.com – Hyper-Converged Ceph](https://pve.proxmox.com/wiki/Deploy_Hyper-Converged_Ceph_Cluster)

[^19]: Ceph – Open Source Storage: [ceph.io](https://ceph.io/)

[^20]: Proxmox LXC Container Dokumentation: [pve.proxmox.com/wiki/Linux_Container](https://pve.proxmox.com/wiki/Linux_Container)

[^21]: Proxmox HA Manager: [pve.proxmox.com/wiki/High_Availability](https://pve.proxmox.com/wiki/High_Availability)

[^22]: Proxmox Backup Server: [proxmox.com/de/proxmox-backup-server](https://www.proxmox.com/de/proxmox-backup-server)

[^23]: Checkmk OSS-Edition (Monitoring): [checkmk.com – Raw Edition](https://checkmk.com/de/product/editions)

[^24]: BSI C5 Kriterienkatalog: [bsi.bund.de – Kriterienkatalog C5](https://www.bsi.bund.de/DE/Themen/Unternehmen-und-Organisationen/Informationen-und-Empfehlungen/Empfehlungen-nach-Angriffszielen/Cloud-Computing/Kriterienkatalog-C5/kriterienkatalog-c5_node.html)

[^25]: Hetzner BSI C5:2020 Typ 2 Testat (März 2026): [hetzner.com – BSI C5 Zertifizierung](https://www.hetzner.com/de/news/hetzner-receives-bsi-c5-certification/)

[^26]: OVHcloud: kein US CLOUD Act Risiko: [ovhcloud.com – „Your service is not subject to the US CLOUD Act"](https://www.ovhcloud.com/en/vps/)

[^27]: Open Telekom Cloud BSI C5 Typ 2: [telekom.com – OTC BSI C5 Zertifikat](https://www.telekom.com/de/medien/medieninformationen/detail/open-telekom-cloud-erhaelt-bsi-c5-zertifikat-605188)

[^28]: Open Telekom Cloud Preismodell Computing: [public.t-cloud.com – Preismodelle Computing](https://public.t-cloud.com/de/preise/preismodelle/computing-container)

[^29]: Hetzner Preisanpassung 15. Juni 2026 (exkl. MwSt.): [docs.hetzner.com – Price Adjustment](https://docs.hetzner.com/general/infrastructure-and-availability/price-adjustment/)

[^30]: OVHcloud VPS Deutschland (inkl. MwSt.): [ovhcloud.com/de/vps/](https://www.ovhcloud.com/de/vps/)

[^31]: Microsoft 365 Business Standard: [microsoft.com/de-de/microsoft-365/business](https://www.microsoft.com/de-de/microsoft-365/business/microsoft-365-business-standard) – ca. 12,50 €/Nutzer/Monat (Stand 2026)
