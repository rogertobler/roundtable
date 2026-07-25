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
- Agent ist in derselben Ausführungsumgebung wie tmux authentifiziert,
- Modell- und Agentkonfiguration ist gültig,
- Git-Anforderungen sind erfüllt,
- Session Backend steht zur Verfügung,
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
- zusätzliche erlaubte Agentenargumente.

Das Session Backend wird anhand der Plattform- und Installationskonfiguration
gewählt und ist im normalen Session-Menü keine Benutzerentscheidung.

Nach Bestätigung legt Roundtable eine echte tmux-Session an, wechselt in das
freigegebene Arbeitsverzeichnis und startet darin die ausgewählte lokal
installierte Agenten-CLI. Roundtable eröffnet keinen separaten API-Dialog.

### FR-SESS-002: Agent pro Session

Jede Session speichert ihren eigenen Agententyp und ihre eigene
Agentenkonfiguration. Gleichzeitig dürfen mehrere Sessions desselben oder
unterschiedlicher Agenten laufen.

Jede laufende Session besitzt genau einen nativen CLI-Prozess als
Gesprächsinstanz. Alle Messenger-Eingaben und lokalen Eingaben erreichen diesen
Prozess.

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
- CLI-Snapshot öffnen,
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

CLI-Definitionen und Output-Erkenner können Rohzustände liefern. Der Core
normalisiert sie auf dieses Statusmodell und speichert zusätzlich Evidenz und
Zeitpunkt. Der Status darf den Tunnel nicht blockieren.

`completed` bezeichnet den Abschluss eines Agententurns oder einer Aufgabe,
nicht automatisch das Ende der CLI-Session. Eine weiterhin laufende
interaktive CLI wechselt anschließend wieder zu `ready`.

### FR-SESS-006: Default-Session

Pro Benutzer und Transportkontext darf höchstens eine Default-Session gelten.
Sie ist ausschließlich das Standardziel freier Nachrichten.

Wird die erste Session eines Benutzer-/Transportkontexts erfolgreich erstellt
und existiert noch kein Default, setzt Roundtable diese Session automatisch als
Default. Später erstellte Sessions verändern den Default nicht. Der Benutzer
kann ihn jederzeit manuell auf eine andere Session setzen.

Eine Default-Session muss erreichbar und für den Benutzer zugänglich sein.
Wird sie gestoppt, gelöscht oder unzugänglich, wird die Default-Zuweisung
entfernt oder ausdrücklich als ungültig angezeigt. Roundtable darf nicht still
eine andere Session wählen.

### FR-SESS-007: Bestehende Sessions

Roundtable soll bestehende Sessions erkennen und übernehmen können, sofern das
Session Backend eine sichere Zuordnung und Ein-/Ausgabe erlaubt.

Eine fremde tmux-Session darf nicht allein aufgrund ihres Namens übernommen
werden. Die Übernahme erfordert Bestätigung und speichert Runtime-ID,
Arbeitsverzeichnis und Agentenzuordnung.

### FR-SESS-008: Terminalwechsel

Der Benutzer kann eine von Roundtable verwaltete Session lokal im Terminal
öffnen oder attachen. Telegram und Terminal greifen auf dieselbe laufende
Agenteninstanz und denselben nativen Chatverlauf zu.

Eine über Telegram gesendete Nachricht muss nach erfolgreicher Zustellung in
der lokal geöffneten CLI-Session sichtbar sein. Eine lokale Eingabe muss vom
Roundtable-Outputpfad erkannt und entsprechend dem Abonnement in die zentrale
Inbox geleitet werden.

### FR-SESS-009: Startaufgabe

Eine beim Erstellen angegebene Aufgabe wird erst gesendet, wenn der Agent
erfolgreich gestartet und eingabebereit ist. Fehlgeschlagener Start und
fehlgeschlagene Aufgabenzustellung werden getrennt protokolliert.

## Nachrichtenrouting

### FR-ROUTE-000: Tunnel-Invariante

Roundtable führt keinen eigenen Agentenkontext. Jede zustellbare
Benutzernachricht wird in die echte CLI-Session geschrieben. Jede
Agentennachricht stammt aus der Ausgabe dieser CLI-Session.

Agenten-APIs, Hooks und strukturierte Ereignisse dürfen nur als optionale
Hinweise für Status, Antwortgrenzen oder Freigaben dienen. Sie sind nicht der
primäre Nachrichtenpfad.

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
4. interaktive Session-Auswahl, falls kein gültiger Default existiert.

Eine vorhandene Reply-Zuordnung darf nicht durch die Default-Session
überschrieben werden.

Das Erstellen einer zweiten oder späteren Session darf eine bestehende
Default-Zuweisung niemals automatisch verändern.

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
Abschlusstaste in die native CLI-Session geschrieben. Roundtable korrigiert,
übersetzt oder erweitert die Eingabe nicht.

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

Für tmux verwendet Roundtable einen kontinuierlichen Outputpfad, beispielsweise
`pipe-pane`, und einen Bildschirm-Snapshot über `capture-pane`. Der
kontinuierliche Stream bewahrt Inhalt; der Snapshot hilft bei Vollbild-TUIs,
Approvals und Wiederverbindung.

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

Erkennt der Output-Erkenner eine Rückfrage, sendet Roundtable sie als neue,
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

Roundtable entscheidet keine Freigabe selbst und erzeugt niemals eigenständig
eine Zustimmung oder Ablehnung. Ein beim Sessionstart ausdrücklich gewählter
agenteneigener Berechtigungsmodus kann beeinflussen, ob die CLI eine Freigabe
anfordert; erscheint ein Prompt, tunnelt Roundtable ausschließlich Prompt und
Benutzerantwort.

### FR-INT-004: Genau einmal beantworten

Eine konkrete Freigabe kann nur einmal erfolgreich beantwortet werden.
Verspätete Button-Klicks zeigen den bereits abgeschlossenen Zustand und senden
keine zweite Eingabe.

### FR-INT-005: Veraltete Interaktion

Vor einer Button-Aktion prüft Roundtable, ob Session, Prompt und erwarteter
Terminalzustand noch zusammenpassen. Kann dies nicht bestätigt werden, wird
nichts gesendet und ein aktueller CLI-Snapshot angeboten.

## Direkter Sessionzugriff und Steuerung

### FR-TERM-001: Bildschirmansicht

Der Benutzer kann den aktuellen Terminalbildschirm als formatierten Text
abrufen. Optional kann später ein Bild erzeugt werden. Dies ist eine Diagnose-
und Komfortfunktion; primäre Oberfläche bleibt der gemeinsame Messenger-Chat.

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
Verbindung hin, sofern das Session Backend sie erkennen kann.

Lokale Eingaben dürfen nicht zu einem getrennten Verlauf führen. Roundtable
beobachtet weiterhin dieselbe CLI-Ausgabe und leitet daraus entstehende
Agentenantworten in die Inbox.

### FR-TERM-005: Nativer Verlauf

Roundtable konfiguriert eine ausreichend große tmux-History und speichert den
Raw Output separat. Lokales Attach zeigt den echten nativen CLI-Zustand; der
vollständige lokale Raw-Verlauf bleibt auch dann abrufbar, wenn die sichtbare
tmux-Scrollback-Grenze erreicht wurde.

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
- CLI-Snapshot anzeigen,
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
