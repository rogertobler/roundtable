# Funktionale Anforderungen

Dieses Dokument beschreibt den vollständigen geplanten Funktionsumfang. Die
Kennungen dienen später als Referenz für Issues, Tests und Releases.

## Benutzer und Kanäle

### FR-USER-001: Privater Telegram-Bot

Roundtable stellt einen Telegram-Bot bereit, der standardmäßig nur in privaten
Chats arbeitet. Nicht freigegebene Benutzer erhalten keine Sessiondaten und
können keine Aktionen auslösen.

### FR-USER-002: Benutzer-Pairing

Die Erstverbindung verwendet einen zeitlich begrenzten lokalen Pairing-Code oder
einen vergleichbar sicheren Ablauf. Die Telegram-User-ID wird nach Bestätigung
in die Allowlist aufgenommen.

### FR-USER-003: Mehrere Transportadapter

Die Kerndomäne darf keine Telegram-spezifischen Identifikatoren voraussetzen.
Telegram wird zuerst implementiert. WhatsApp und weitere Kanäle sollen später
als Adapter ergänzt werden.

### FR-USER-004: Mehrere Benutzer

Die erste Version darf auf einen Besitzer beschränkt sein. Das Datenmodell soll
Benutzerzuordnungen und spätere Rollen ermöglichen.

## Projekte

### FR-PROJ-001: Projektverwaltung

Ein Projekt besitzt mindestens:

- eindeutige ID,
- Anzeigename,
- kanonisches Hauptverzeichnis,
- optionale Repository-URL,
- erlaubte Agenten,
- Standard-Agent und Standardmodell,
- erlaubte Arbeitsverzeichnisse,
- Standard-Berechtigungsprofil,
- optionale Startparameter,
- optionale Umgebungsvariablen-Referenzen,
- Standard-Benachrichtigungsmodus.

### FR-PROJ-002: Verzeichnis-Allowlist

Sessions dürfen nur innerhalb explizit freigegebener Projektpfade angelegt
werden. Pfadauflösung muss symbolische Links und Plattformunterschiede sicher
behandeln.

### FR-PROJ-003: Projektprüfung

Vor dem Session-Start prüft Roundtable:

- Verzeichnis existiert und ist erreichbar,
- Agent ist installiert,
- Modell- und Agentkonfiguration ist gültig,
- Git-Anforderungen sind erfüllt,
- Runtime-Backend steht zur Verfügung,
- Berechtigungsprofil ist zulässig.

### FR-PROJ-004: Git-Worktrees

Roundtable kann neue Branches und Worktrees für parallele Sessions erstellen.
Name und Zielpfad werden vor der Erstellung angezeigt. Fehler hinterlassen
keine fälschlich als aktiv markierte Session.

## Sessions

### FR-SESS-001: Session erstellen

Eine Session kann über ein geführtes Menü angelegt werden. Erforderliche
Auswahl:

1. Agent,
2. Modell oder Agentenprofil,
3. Projekt,
4. Arbeitsmodus,
5. Sessionname,
6. Berechtigungsprofil.

Optionale Auswahl:

- konkretes Arbeitsverzeichnis,
- bestehender oder neuer Branch,
- bestehender oder neuer Worktree,
- initiale Aufgabe,
- Benachrichtigungsmodus,
- Runtime-Backend,
- zusätzliche erlaubte Agentenargumente.

### FR-SESS-002: Agent pro Session

Jede Session speichert ihren eigenen Agententyp und ihre eigene
Agentenkonfiguration. Gleichzeitig dürfen mehrere Sessions desselben oder
unterschiedlicher Agenten laufen.

### FR-SESS-003: Session-Liste

Die Session-Liste zeigt mindestens:

- Name,
- Agent,
- Modell, soweit bekannt,
- Projekt,
- Status,
- Default-Markierung,
- Abonnementstatus,
- letzte Aktivität,
- Hinweis auf ausstehende Eingabe oder Freigabe.

### FR-SESS-004: Session-Aktionen

Verfügbare Aktionen:

- Nachricht senden,
- als Default setzen,
- Default-Zuweisung entfernen,
- abonnieren,
- Benachrichtigungsmodus ändern,
- stummschalten,
- Status aktualisieren,
- Terminalansicht öffnen,
- rohe oder vollständige Ausgabe abrufen,
- Verlauf anzeigen,
- Enter, Escape, Tab und Pfeiltasten senden,
- `Ctrl+C` senden,
- Agentenprozess unterbrechen,
- Session fortsetzen,
- Session umbenennen,
- Session neu starten,
- Session stoppen,
- Session archivieren,
- Sessiondaten kontrolliert löschen.

Destruktive Aktionen verlangen eine Bestätigung und zeigen die genaue
Zielsession.

### FR-SESS-005: Session-Status

Unterstützte kanonische Zustände:

- `starting`,
- `ready`,
- `working`,
- `waiting_for_input`,
- `waiting_for_approval`,
- `completed`,
- `interrupted`,
- `stopped`,
- `error`,
- `disconnected`,
- `unknown`.

Agent-Adapter können Rohzustände liefern. Der Core normalisiert sie auf dieses
Statusmodell und speichert zusätzlich Evidenz und Zeitpunkt.

### FR-SESS-006: Default-Session

Pro Benutzer und Transportkontext darf höchstens eine Default-Session gelten.
Sie ist ausschließlich das Standardziel freier Nachrichten.

Eine Default-Session muss erreichbar und für den Benutzer zugänglich sein.
Wird sie gestoppt, gelöscht oder unzugänglich, wird die Default-Zuweisung
entfernt oder ausdrücklich als ungültig angezeigt. Roundtable darf nicht still
eine andere Session wählen.

### FR-SESS-007: Bestehende Sessions

Roundtable soll bestehende Sessions erkennen und übernehmen können, sofern das
Runtime-Backend eine sichere Zuordnung und Ein-/Ausgabe erlaubt.

Eine fremde tmux-Session darf nicht allein aufgrund ihres Namens übernommen
werden. Die Übernahme erfordert Bestätigung und speichert Runtime-ID,
Arbeitsverzeichnis und Agentenzuordnung.

### FR-SESS-008: Terminalwechsel

Der Benutzer kann eine von Roundtable verwaltete Session lokal im Terminal
öffnen oder attachen. Telegram und Terminal greifen auf dieselbe laufende
Agenteninstanz zu.

### FR-SESS-009: Startaufgabe

Eine beim Erstellen angegebene Aufgabe wird erst gesendet, wenn der Agent
erfolgreich gestartet und eingabebereit ist. Fehlgeschlagener Start und
fehlgeschlagene Aufgabenzustellung werden getrennt protokolliert.

## Nachrichtenrouting

### FR-ROUTE-001: Reply-Routing

Zu jeder von Roundtable gesendeten routbaren Nachricht wird dauerhaft
gespeichert:

- Transport,
- Konto oder Bot,
- Chat-ID,
- externe Nachrichten-ID,
- interne Nachrichten-ID,
- Session-ID,
- Zeitpunkt,
- Nachrichtentyp.

Antwortet ein Benutzer auf diese Nachricht, wird seine Eingabe an genau diese
Session geroutet.

### FR-ROUTE-002: Priorität

Die Zielauflösung erfolgt in dieser Reihenfolge:

1. explizite Reply-Zuordnung,
2. explizites Session-Präfix oder ausgewähltes Sessionziel,
3. Default-Session,
4. interaktive Session-Auswahl.

Eine vorhandene Reply-Zuordnung darf nicht durch die Default-Session
überschrieben werden.

### FR-ROUTE-003: Keine geratenen Zustellungen

Ist eine Reply-Zuordnung unbekannt, gelöscht, mehrdeutig oder nicht mehr
zugänglich, sendet Roundtable die Nachricht nicht. Es zeigt den Grund und bietet
eine explizite Session-Auswahl an.

### FR-ROUTE-004: Session-Präfix

Freie Nachrichten können optional mit einem eindeutigen Session-Alias beginnen,
zum Beispiel:

```text
@backend-review prüfe den aktuellen Diff
```

Mehrdeutige Aliase führen zu einer Auswahl. Der Alias wird vor Weiterleitung
entfernt, sofern die Benutzeroberfläche dies klar anzeigt.

### FR-ROUTE-005: Inhaltstreue

Benutzereingaben werden standardmäßig exakt als Text plus die konfigurierte
Abschlusstaste weitergereicht. Roundtable korrigiert, übersetzt oder erweitert
die Eingabe nicht.

### FR-ROUTE-006: Idempotenz

Doppelt zugestellte Transportupdates dürfen nicht zu doppelten
Terminaleingaben führen. Jede eingehende Transportnachricht besitzt einen
dauerhaft gespeicherten Idempotenzschlüssel.

### FR-ROUTE-007: Reihenfolge

Eingaben an dieselbe Session werden seriell und in bestätigter Reihenfolge
zugestellt. Eingaben an verschiedene Sessions dürfen parallel verarbeitet
werden.

### FR-ROUTE-008: Zustellstatus

Roundtable unterscheidet mindestens:

- empfangen,
- Ziel aufgelöst,
- zur Zustellung eingeplant,
- an Runtime geschrieben,
- fehlgeschlagen,
- vom Benutzer abgebrochen.

Ein Häkchen in Telegram darf nicht fälschlich bedeuten, dass der Agent die
Eingabe semantisch verarbeitet hat.

## Agentenausgaben

### FR-OUT-001: Ausgabeerfassung

Roundtable erfasst den fortlaufenden Rohdatenstrom und, soweit verfügbar, den
aktuellen Terminalbildschirm. Beide Darstellungen haben unterschiedliche
Zwecke und dürfen nicht verwechselt werden.

### FR-OUT-002: Transportaufbereitung

Für Telegram darf Roundtable:

- ANSI- und Cursorsteuersequenzen entfernen,
- Spinner und überschreibende Statuszeilen stabilisieren,
- Ausgabe in transportkonforme Blöcke aufteilen,
- lange Inhalte als Datei bereitstellen,
- Markdown-Sonderzeichen korrekt behandeln.

Roundtable darf den Inhalt nicht durch ein Sprachmodell umschreiben oder
verkürzen, ohne dass der Benutzer explizit eine solche Funktion aufruft.

### FR-OUT-003: Zusammenführung

Kleine, schnell aufeinanderfolgende Ausgabestücke dürfen innerhalb eines kurzen
Fensters zu einer Nachricht zusammengeführt werden. Reihenfolge und Inhalt
bleiben erhalten.

### FR-OUT-004: Fortschrittsnachricht

Für flüchtige Arbeitszustände kann Roundtable eine bestehende
Fortschrittsnachricht aktualisieren. Stabile Ergebnisse, Fragen, Fehler und
Freigaben werden als neue routbare Nachrichten gesendet.

### FR-OUT-005: Große Ausgaben

Bei Überschreitung der Transportgrenze:

1. Nachricht verlustfrei teilen oder
2. vollständigen Inhalt als Datei senden und
3. Zuordnung aller Teile zur Session speichern.

Eine Kürzung muss sichtbar gekennzeichnet sein und einen Weg zum vollständigen
Inhalt anbieten.

### FR-OUT-006: Abonnements

Pro Benutzer und Session sind folgende Modi vorgesehen:

- alle stabilen Ausgaben,
- nur Rückfragen,
- nur Freigaben,
- nur Abschlussmeldungen,
- nur Fehler,
- stumm.

Unabhängig vom Modus bleibt der lokale Verlauf abrufbar.

## Rückfragen und Freigaben

### FR-INT-001: Rückfragen

Erkennt ein Agent-Adapter eine Rückfrage, sendet Roundtable sie als neue,
routbare Nachricht. Antwortoptionen können als Buttons dargestellt werden,
wenn ihre Terminalwirkung eindeutig ist.

### FR-INT-002: Freigaben

Eine Freigabe enthält:

- Session, Agent und Projekt,
- unveränderten sichtbaren Prompt,
- soweit erkennbar die angefragte Aktion,
- mögliche Antworten,
- genaue Eingabe oder Tastenfolge pro Button,
- Ablaufstatus.

### FR-INT-003: Keine automatische Entscheidung

Der Standard-Core entscheidet keine Freigabe selbst. Automatische Regeln sind
eine spätere, ausdrücklich konfigurierte Funktion und müssen enger als das
Berechtigungsprofil der Session sein.

### FR-INT-004: Genau einmal beantworten

Eine konkrete Freigabe kann nur einmal erfolgreich beantwortet werden.
Verspätete Button-Klicks zeigen den bereits abgeschlossenen Zustand und senden
keine zweite Eingabe.

### FR-INT-005: Veraltete Interaktion

Vor einer Button-Aktion prüft Roundtable, ob Session, Prompt und erwarteter
Terminalzustand noch zusammenpassen. Kann dies nicht bestätigt werden, wird
nichts gesendet und eine aktuelle Terminalansicht angeboten.

## Terminalsteuerung

### FR-TERM-001: Bildschirmansicht

Der Benutzer kann den aktuellen Terminalbildschirm als formatierten Text
abrufen. Optional kann später ein Bild erzeugt werden.

### FR-TERM-002: Grundtasten

Unterstützt werden mindestens:

- Enter,
- Escape,
- Tab,
- Pfeil hoch,
- Pfeil runter,
- `Ctrl+C`.

Weitere Tasten können über einen erweiterten Modus angeboten werden.

### FR-TERM-003: Texteingabe

Text und abschließende Taste werden getrennt modelliert. Dadurch kann
Roundtable Text ohne Enter einfügen oder ausschließlich eine Taste senden.

### FR-TERM-004: Konkurrenz

Wenn der Benutzer gleichzeitig lokal und über Telegram schreibt, serialisiert
Roundtable nur seine eigenen Eingaben. Die UI weist auf eine aktive lokale
Verbindung hin, sofern das Runtime-Backend sie erkennen kann.

## Verlauf und Audit

### FR-AUDIT-001: Sessionverlauf

Pro Session sind ausgehende Agentennachrichten, eingehende Benutzerantworten,
Statuswechsel, Freigaben und Sessionaktionen zeitlich nachvollziehbar.

### FR-AUDIT-002: Sensible Inhalte

Für Rohdaten, gerenderte Nachrichten und Metadaten gelten getrennte
Aufbewahrungsregeln. Vollständige Terminalausgaben können ausschließlich lokal
gespeichert werden.

### FR-AUDIT-003: Export

Ein Sessionverlauf kann später als strukturierte Datei exportiert werden.
Secrets werden nach konfigurierbaren Regeln maskiert.

## Fehlerbehandlung

### FR-ERR-001: Erkannte Fehler

Roundtable meldet mindestens:

- Agent nicht installiert,
- ungültiges Modell,
- Arbeitsverzeichnis nicht verfügbar,
- Runtime nicht verfügbar,
- Start fehlgeschlagen,
- Agent abgestürzt,
- Session Host nicht erreichbar,
- Telegram-Zustellung fehlgeschlagen,
- unbekannte Reply-Zuordnung,
- Eingabe nicht zustellbar,
- Worktree-Erstellung fehlgeschlagen,
- Persistenzfehler,
- Bot-Authentifizierung fehlgeschlagen.

### FR-ERR-002: Wiederherstellungsaktionen

Je nach Fehler werden sichere Aktionen angeboten:

- erneut versuchen,
- Status neu laden,
- Terminal anzeigen,
- Runtime wiederverbinden,
- Agent fortsetzen,
- Session neu starten,
- Session stoppen,
- Details anzeigen.

### FR-ERR-003: Kein stiller Verlust

Eine nicht zugestellte Benutzereingabe wird sichtbar als fehlgeschlagen
markiert. Sie darf nicht als erfolgreich gelten oder ohne Benutzerentscheidung
an eine andere Session umgeleitet werden.

## Spätere Funktionen

Zum vollständigen Zielumfang gehören außerdem:

- WhatsApp und weitere Transportkanäle,
- Datei-, Bild- und Screenshotübertragung,
- Sprachnachrichten und Transkription,
- automatische Worktrees,
- Agent-zu-Agent-Übergaben,
- planbare und wiederkehrende Aufgaben,
- Review-Pipelines,
- Web-Dashboard,
- Teams, Rollen und Rechte,
- Kosten- und Tokenübersichten,
- optionale Sitzungszusammenfassungen,
- GitHub- und GitLab-Integration,
- Pull-Request-Freigaben,
- Monitoring- und Deployment-Integrationen,
- Verwaltung entfernter Roundtable-Nodes.
