# Umsetzungsroadmap

## Grundsatz

Roundtable soll den beschriebenen Gesamtumfang erreichen. Die Phasen reduzieren
nicht die Vision, sondern schaffen überprüfbare Zwischenschritte. Jede Phase
muss auf den stabilen Verträgen für Transport, Session, Agent und Runtime
aufbauen.

## Phase 0: Technischer Spike

Ziel: Die risikoreichsten Terminal- und Routingannahmen praktisch bestätigen.

Umfang:

- minimale Telegram-Bot-Verbindung,
- eine tmux-Session starten,
- Claude Code und Codex jeweils testweise starten,
- Output über `pipe-pane` erfassen,
- Terminalbild über `capture-pane` lesen,
- literal Text und Grundtasten senden,
- Telegram-Nachrichten-ID einer Session zuordnen,
- Reply an die richtige Session senden,
- Claude und Codex gleichzeitig betreiben,
- typische Approval-Prompts beider Agenten aufzeichnen,
- Router neu starten und tmux-Session wiederfinden.

Abnahme:

- Eine Claude- und eine Codex-Session laufen gleichzeitig.
- Replies auf Nachrichten beider Sessions landen zehnmal hintereinander
  nachweislich in der richtigen Session.
- Freie Nachricht geht an die gespeicherte Default-Session.
- Nach Router-Neustart bleiben tmux-Prozesse aktiv und Zuordnungen bestehen.
- Raw Stream und Screen Snapshot sind praktisch unterscheidbar.

Ergebnis:

- Terminaltranskripte als Testfixtures,
- validierte tmux-Kommandos,
- Entscheidung über Implementierungssprache,
- konkretisierte Adapterverträge.

## Phase 1: Nutzbarer Linux-MVP

Ziel: Roundtable kann von einem einzelnen Besitzer auf Linux oder einem
Linux-VPS täglich genutzt werden.

Umfang:

- installierbarer Roundtable-Daemon,
- privater Telegram-Bot,
- sicheres Einbenutzer-Pairing,
- SQLite-Migrationen,
- Projekt-Allowlist,
- Claude-Code-Adapter,
- Codex-Adapter,
- Agent und Modell pro Session,
- tmux-Runtime,
- mehrere parallele Sessions,
- geführter Session-Start,
- Session-Liste und Detailansicht,
- Reply-Routing,
- Default-Session,
- explizite Sessionauswahl ohne Default,
- Outputaufbereitung ohne inhaltliches Umschreiben,
- Terminal-Snapshot,
- Text, Enter, Escape und `Ctrl+C`,
- Session starten, unterbrechen und stoppen,
- Abonnieren und Stummschalten,
- einfache Statusmeldungen,
- inhaltstreue Rückfragen und Freigaben,
- persistente Nachrichten- und Sessionzuordnungen,
- strukturierte Logs und `roundtable doctor`.

Abnahme:

- Mindestens fünf parallele Sessions funktionieren über mehrere Projekte.
- Claude und Codex können gleichzeitig bedient werden.
- Jede Reply wird über persistiertes Mapping geroutet.
- Ohne Reply wird ausschließlich die konfigurierte Default-Session verwendet.
- Doppelte Telegram-Updates erzeugen keine doppelten Terminaleingaben.
- Neustart des Core beendet tmux-Sessions nicht.
- Nicht autorisierte Telegram-Nutzer sehen keine Metadaten.
- Projekte außerhalb der Allowlist können nicht gestartet werden.
- Lange Ausgaben bleiben vollständig abrufbar.

## Phase 2: Robuste Inbox und Interaktionen

Ziel: Hohe Zuverlässigkeit bei langen Sessions, TUIs und instabilen
Netzverbindungen.

Umfang:

- Transactional Outbox,
- persistente Session-Input-Queues,
- Output-Cursor und Backlog-Recovery,
- stabile Fortschrittsnachrichten,
- alle Benachrichtigungsmodi,
- Interaction Manager,
- Prompt-Fingerprints,
- idempotente Approval-Buttons,
- vollständige Grundtasten,
- Verlauf und Raw-Output-Dateien,
- Datei- und Diff-Versand,
- Secret-Filter,
- Retention und Bereinigung,
- Wiederholungsstrategien und Dead-Letter-Ansicht,
- bestehende tmux-Sessions kontrolliert übernehmen,
- lokales Attach komfortabel anzeigen.

Abnahme:

- Ein Telegram-Ausfall verliert keine stabile Agentennachricht.
- Veraltete Approval-Buttons senden keine Eingabe.
- Schnelle Terminalausgabe erzeugt keine unkontrollierte Nachrichtenflut.
- Raw Output kann mit Prüfsumme vollständig nachvollzogen werden.
- Datenbank- oder Queue-Fehler führen nicht zu stiller Zustellung.

## Phase 3: Git- und Projektkomfort

Ziel: Paralleles Arbeiten an realen Repositories wird ergonomisch und sicher.

Umfang:

- automatische Worktree-Erstellung,
- Branch-Namensregeln,
- bestehende Worktrees auswählen,
- Worktree-Status anzeigen,
- sichere Archivierung,
- Git-Diff als Artefakt,
- Projektvorlagen,
- Agentprofile,
- Standardmodelle und Standardberechtigungen,
- projektbezogene Umgebungsreferenzen,
- GitHub-/GitLab-Verknüpfung als optionale Adapter.

Abnahme:

- Zwei Agenten können dasselbe Repository in getrennten Worktrees bearbeiten.
- Roundtable zeigt jederzeit den exakten Pfad und Branch.
- Stoppen einer Session löscht keinen Worktree automatisch.
- Worktree-Entfernung ist explizit, geprüft und auditierbar.

## Phase 4: macOS

Ziel: Gleichwertiger lokaler Betrieb auf macOS.

Umfang:

- signierbares Binary,
- `launchd`-Integration,
- Keychain-Secrets,
- tmux-Installation oder eigener Unix Session Host,
- Pfad- und Berechtigungstests,
- Update- und Diagnoseablauf,
- Installer beziehungsweise Homebrew-Paket.

Abnahme:

- Funktionsumfang des Linux-MVP läuft auf unterstützten macOS-Versionen.
- Dienst übersteht Logout/Reboot gemäß dokumentierter Betriebsart.
- Agentensessions bleiben bei Core-Neustart erhalten.
- Installation erfordert keine Arbeit am Roundtable-Quellcode.

## Phase 5: Windows

Ziel: Native Windows-Nutzung und klar unterstützter WSL-Modus.

Umfang:

- ConPTY Session Host,
- Named-Pipe-Protokoll,
- persistente Output-Logs,
- Core-Wiederverbindung,
- Windows-Dienst oder Benutzer-Autostart,
- Credential Manager/DPAPI,
- Windows-Pfadmodell,
- Claude/Codex-Erkennung unter Windows,
- optional WSL-tmux-Backend,
- signierter Installer.

Abnahme:

- Claude und Codex können nativ in parallelen ConPTY-Sessions laufen.
- Neustart des Core beendet separate Session Hosts nicht.
- Texte und Tasten werden ohne Shell-Interpretation übertragen.
- WSL- und native Pfade werden in der UI eindeutig unterschieden.
- Installation und Deinstallation hinterlassen keine unkontrollierten Dienste.

## Phase 6: Weitere Eingabe- und Transportarten

Ziel: Roundtable wird zu einer echten kanalunabhängigen Kommunikationsschicht.

Umfang:

- WhatsApp-Transport,
- Dateiempfang,
- Bilder und Screenshots an Agenten,
- Sprachnachrichten,
- lokale oder konfigurierbare Transkription,
- Transport-Capability-Modell,
- kanalübergreifende Default- und Reply-Semantik,
- konsistente Identitätsverknüpfung.

Abnahme:

- Dieselbe Session kann über Telegram und einen zweiten Transport erreicht
  werden.
- Reply-Routing bleibt pro Transport eindeutig.
- Transportlimits verändern nicht den Inhalt.
- Dateien werden vor Zustellung autorisiert und sicher gespeichert.

## Phase 7: Multi-Agent-Workflows

Ziel: Kontrollierte Zusammenarbeit mehrerer Sessions.

Umfang:

- Übergabe einer Nachricht oder eines Artefakts zwischen Sessions,
- plan -> implement -> review -> fix,
- konfigurierbare Bestätigung pro Übergabeschritt,
- Workflowzustände,
- Abbruch und Fortsetzung,
- gemeinsame Abschlusszusammenfassung auf explizite Anforderung,
- wiederkehrende und geplante Aufgaben,
- Review-Pipelines.

Abnahme:

- Jede Agent-zu-Agent-Nachricht besitzt Quelle, Ziel und Auditspur.
- Kein Agent darf ohne konfigurierte Regel beliebige Sessions bedienen.
- Benutzer kann vor jedem Übergabeschritt eingreifen.
- Workflowfehler stoppen nicht unabhängige Sessions.

## Phase 8: Mehrere Nodes, Web und Teams

Ziel: Roundtable skaliert über einen einzelnen Rechner und Benutzer hinaus.

Umfang:

- mehrere lokale oder entfernte Nodes,
- sichere Node-Identitäten,
- Web-Dashboard,
- mehrere Benutzer,
- Rollen und Projektberechtigungen,
- Session-Sharing,
- Kosten- und Tokenübersichten,
- Statistiken,
- Monitoring- und Deployment-Integrationen,
- mobile Pull-Request-Freigaben.

Abnahme:

- Sessionzugriff ist pro Benutzer und Projekt begrenzt.
- Node-Ausfall ist klar vom Agentenausfall unterscheidbar.
- Telegram und Web verwenden denselben Core und dieselben Invarianten.
- Auditdaten weisen Benutzer, Transport, Node und Session aus.

## Querschnittsarbeit

Jede Phase enthält:

- automatisierte Unit- und Integrationstests,
- Tests mit echten oder aufgezeichneten Agenten-TUIs,
- Datenbankmigrationen,
- Sicherheitsprüfung,
- dokumentierte Upgrade- und Recovery-Wege,
- plattformspezifische CI,
- Versions- und Capability-Matrix für Claude Code und Codex,
- verständliche Fehlermeldungen,
- aktualisierte Dokumentation.

## Teststrategie

### Unit-Tests

- Routingpriorität,
- Default-Session-Invarianten,
- Idempotenz,
- Statusnormalisierung,
- Berechtigungsentscheidungen,
- Outputsegmentierung,
- ANSI-Verarbeitung,
- Secret-Redaktion,
- Interaction-Zustandsautomat.

### Adapter-Contract-Tests

Jeder Transport-, Agenten- und Runtime-Adapter muss dieselbe Contract-Suite
bestehen.

### Transkript-Tests

Aufgezeichnete Ausgaben verschiedener Claude- und Codex-Versionen prüfen:

- Ready-Erkennung,
- Fragen,
- Approval-Optionen,
- Spinner,
- Vollbildänderungen,
- Abschluss,
- Fehler.

### Integrationstests

- echter lokaler Telegram-Testbot in separater Umgebung,
- echte tmux-Sessions,
- Neustart und Wiederverbindung,
- parallele Eingaben,
- Netzwerkunterbrechung,
- volle oder gesperrte Datenbank,
- veraltete Callbacks.

### End-to-End-Kernszenario

1. Claude-Session A starten.
2. Codex-Session B starten.
3. Session B als Default festlegen.
4. Agent A und B erzeugen je eine Nachricht.
5. Auf A antworten und Zustellung an A prüfen.
6. Auf B antworten und Zustellung an B prüfen.
7. Freie Nachricht senden und Zustellung an B prüfen.
8. Core neu starten.
9. Schritte 5 bis 7 wiederholen.
10. Eine veraltete Freigabe ablehnen, ohne Eingabe zu senden.

Dieses Szenario ist die dauerhafte Definition des zentralen Produktversprechens.

## Releasekriterien

Ein Release ist nicht bereit, wenn:

- eine Reply unter realistischen Bedingungen an die falsche Session gelangen
  kann,
- doppelte Updates doppelte Eingaben erzeugen,
- der Installer Secrets unsicher speichert,
- laufende Sessions beim normalen Core-Update unnötig beendet werden,
- eine Plattform nur durch nicht dokumentierte manuelle Entwicklungsschritte
  funktioniert,
- neue Agentenversionen ohne sicheren Fallback riskante Buttons erzeugen.
