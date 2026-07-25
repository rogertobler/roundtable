# Telegram-Bedienkonzept

## Kanalmodell

Die erste Oberfläche ist ein privater Chat mit einem Telegram-Bot. Ein
Telegram-Kanal ist dafür ungeeignet, weil Roundtable bidirektionale
Interaktionen, Reply-Metadaten und Menüs benötigt.

Die gemeinsame Inbox enthält Nachrichten aller Sessions, die der Benutzer
abonniert hat. Eine Session muss keinen eigenen Chat und kein eigenes Topic
besitzen. Die Zuordnung geschieht über gespeicherte Nachrichten-IDs.

Der Telegram-Chat ist die primäre Roundtable-Oberfläche. Er zeigt eine
gemeinsame, zeitlich sortierte Inbox mehrerer echter CLI-Sessions. Roundtable
führt keinen zusätzlichen Agenten-Chat außerhalb dieser Sessions.

Telegram Topics können später optional als alternative Darstellung unterstützt
werden, sind aber nicht die Grundlage des Routings.

## Hauptnavigation

Das Hauptmenü enthält:

```text
[Neue Session]  [Sessions]
[Projekte]      [Default-Session]
[Inbox-Modi]    [Fehler]
[Einstellungen]
```

Zusätzlich sind kompakte Slash-Kommandos möglich:

```text
/new
/sessions
/default
/projects
/errors
/settings
```

Die Buttons sind die primäre Oberfläche. Kommandos dienen erfahrenen
Benutzern und als Fallback.

## Neue Session

Der geführte Ablauf hält pro Benutzer einen kurzlebigen Entwurf. Jeder Schritt
kann zurückgegangen oder abgebrochen werden.

### 1. Agent

```text
Neue Session

Welcher Agent soll laufen?

[Claude Code]
[Codex]
[Weitere Agenten ...]
[Abbrechen]
```

Nur lokal installierte und für mindestens ein Projekt erlaubte Agenten sind
direkt auswählbar. Nicht verfügbare Agenten können mit Installationshinweis
angezeigt werden.

### 2. Modell

```text
Agent: Claude Code

Modell auswählen:

[Projektstandard]
[Opus]
[Sonnet]
[Agentenstandard]
[Zurück]
```

Modellnamen stammen aus der lokalen Agentenkonfiguration oder einem
aktualisierbaren Capability-Katalog. Roundtable muss unbekannte, manuell
konfigurierte Modellwerte unterstützen.

### 3. Projekt

```text
Projekt auswählen:

[DumbleScore]
[Inscribe]
[Pizza Ninjas]
[Projekt hinzufügen]
[Zurück]
```

Es werden nur freigegebene Projekte angezeigt.

### 4. Arbeitsmodus

```text
Wie soll die Session arbeiten?

[Projektverzeichnis]
[Bestehender Worktree]
[Neuer Worktree]
[Nur lesen]
[Zurück]
```

Bei einem neuen Worktree folgen Branchname und Zielpfad. Vor der Erstellung
zeigt Roundtable eine Zusammenfassung.

### 5. Name

```text
Wie soll die Session heißen?

Beispiel: Backend Import Fix
```

Namen müssen innerhalb des Benutzerkontexts eindeutig unterscheidbar sein.
Roundtable erzeugt zusätzlich eine unveränderliche interne ID und einen
optionalen eindeutigen Alias.

### 6. Berechtigungsprofil

```text
Berechtigungsprofil:

[Nur lesen]
[Standard]
[Vertrauenswürdig]
[Eingeschränkt]
[Zurück]
```

Die genaue Wirkung wird in der Session-Zusammenfassung angezeigt. Das
Berechtigungsprofil ist eine Roundtable-Grenze und kann zusätzlich zum
agenteneigenen Berechtigungsmodus wirken.

### 7. Startaufgabe

```text
Optionale erste Aufgabe senden:

[Text eingeben]
[Ohne Aufgabe starten]
[Zurück]
```

### 8. Bestätigung

```text
Session starten?

Name: Backend Import Fix
Agent: Claude Code
Modell: Opus
Projekt: DumbleScore
Arbeitsmodus: Neuer Worktree
Branch: agent/backend-import-fix
Berechtigungen: Standard
Benachrichtigungen: Alle stabilen Ausgaben

[Starten]
[Bearbeiten]
[Abbrechen]
```

Nach erfolgreichem Start:

```text
Claude Code · Backend Import Fix
Status: bereit
Projekt: DumbleScore
Modell: Opus

Die Session wurde gestartet.

[Nachricht senden]
[Als Default setzen]
[CLI-Snapshot]
[Details]
```

Diese Nachricht ist bereits routbar. Eine Reply darauf erreicht die neue
Session.

Im Hintergrund wurde eine tmux-Session erstellt und darin die gewählte
Agenten-CLI gestartet. Alle folgenden Telegram-Eingaben erscheinen auch in
dieser nativen CLI.

Ist dies die erste erfolgreich erstellte Session und existiert noch kein
Default, zeigt die Bestätigung zusätzlich:

```text
Diese Session ist jetzt deine Default-Session.
Freie Nachrichten ohne Reply werden an sie gesendet.
```

Weitere Sessionstarts verändern den Default nicht.

## Session-Liste

```text
Sessions

● Backend Feature
  Claude Code · Opus · arbeitet
  DumbleScore · vor 20 Sekunden

★ Backend Review
  Codex · GPT-Modell · wartet auf Eingabe
  DumbleScore · vor 2 Minuten

○ Website
  Claude Code · Sonnet · gestoppt
  Inscribe · gestern

[Neue Session]
[Aktualisieren]
```

Legende:

- `★` ist die Default-Session.
- `●` ist eine laufende oder erreichbare Session.
- `○` ist eine gestoppte oder archivierte Session.

Farben oder Emojis sind nur zusätzliche Hinweise; der Zustand muss immer auch
als Text lesbar sein.

## Session-Detail

```text
Backend Review

Agent: Codex
Modell: GPT-Modell
Projekt: DumbleScore
Verzeichnis: /srv/projects/dumblescore-review
Branch: agent/codex-review
Status: wartet auf Eingabe
Runtime: tmux · verbunden
Letzte Aktivität: vor 2 Minuten
Inbox: alle stabilen Ausgaben
Default: ja

[Nachricht] [Terminal]
[Unterbrechen] [Fortsetzen]
[Inbox-Modus] [Default entfernen]
[Neu starten] [Stoppen]
[Verlauf] [Weitere Aktionen]
```

## Nachrichtenformat

Jede neue stabile Agentennachricht enthält eine kompakte Herkunftszeile:

```text
Claude Code · Backend Feature
DumbleScore · wartet auf Antwort

Soll ich vor der Migration prüfen, ob die alte Spalte noch verwendet wird?
```

Die Herkunft muss auch bei weitergeleiteten oder als Datei gesendeten Inhalten
erkennbar bleiben. Lange Metadaten gehören in die Detailansicht, nicht in jede
Nachricht.

Jede stabile Nachricht ist intern mit der Session verknüpft. Ein sichtbarer
Hinweis „Antworte auf diese Nachricht“ kann beim Onboarding angezeigt und
später ausgeblendet werden.

## Reply-Routing

Beispiel:

```text
Claude Code · Backend Feature:
"Soll ich die Migration ausführen?"

Benutzer antwortet per Telegram-Reply:
"Prüfe zuerst alle Verwendungen."
```

Roundtable:

1. liest die ID der beantworteten Telegram-Nachricht,
2. findet die gespeicherte Session-Zuordnung,
3. validiert Benutzerzugriff und Sessionzustand,
4. speichert die eingehende Nachricht,
5. schreibt exakt den Benutzertext und standardmäßig Enter in die Session,
6. protokolliert den Zustellstatus.

Ein Reply-Thread kann beliebig lang sein. Entscheidend ist immer die konkrete
beantwortete Nachricht, nicht eine Interpretation des Gesprächsinhalts.

## Freie Nachrichten und Default-Session

Eine Nachricht ohne Reply und ohne explizites Sessionziel geht an die
Default-Session:

```text
Benutzer:
"Führe jetzt die Tests aus."

Roundtable:
-> Default-Session "Backend Review"
```

Roundtable bestätigt die Zielsession kompakt, besonders beim ersten Senden nach
einem Default-Wechsel:

```text
An Backend Review · Codex gesendet.
```

Gibt es keine gültige Default-Session:

```text
An welche Session soll diese Nachricht gesendet werden?

[Backend Feature · Claude]
[Backend Review · Codex]
[Website · Claude]
[Abbrechen]
```

Der ursprüngliche Text wird als ausstehende Nachricht gespeichert und nach
Auswahl genau einmal zugestellt.

## Explizites Sessionziel

Optional kann ein Alias verwendet werden:

```text
@backend-feature führe die betroffenen Tests aus
```

Vor dem Senden zeigt Roundtable bei Bedarf:

```text
An Backend Feature · Claude Code gesendet.
```

Ein Präfix ist kein Ersatz für Replies. Replies haben immer Vorrang.

## Default festlegen

Im Session-Menü:

```text
Backend Review als Default-Session festlegen?

Freie Nachrichten ohne Reply werden künftig an diese Session gesendet.
Replies bleiben unverändert ihrer Ursprungssession zugeordnet.

[Als Default setzen]
[Abbrechen]
```

Nach dem Wechsel:

```text
Default-Session: Backend Review · Codex
```

Es gibt bewusst keine automatisch wechselnde „aktive Session“. Ein
Statuswechsel oder eine neue Agentennachricht darf die Default-Session nicht
verändern. Ausschließlich die erste Session wird initial automatisch zum
Default, wenn vorher kein Default existiert. Danach wird er nur manuell
gewechselt.

## Freigaben

Der sichtbare Agentenprompt wird nicht umformuliert:

```text
Codex · Backend Review
DumbleScore · wartet auf Freigabe

Codex möchte folgenden Befehl ausführen:

npm test

[Einmal erlauben]
[Ablehnen]
[CLI-Snapshot]
```

Jede Schaltfläche ist intern an eine konkrete Eingabe gebunden, beispielsweise:

```text
Einmal erlauben -> Taste "1", danach Enter
Ablehnen        -> Taste "3", danach Enter
```

Die Zuordnung stammt aus der Agenten-Definition oder aus der aktuell erkannten
CLI-Interaktion. Roundtable zeigt auf Wunsch die tatsächliche Tastenfolge.

Kann der erwartete Prompt nicht mehr bestätigt werden, wird keine Taste
gesendet:

```text
Diese Freigabe ist nicht mehr aktuell.

[Aktuellen CLI-Snapshot anzeigen]
[Session öffnen]
```

Eine freie Textantwort auf den Freigabeprompt wird ebenfalls exakt an die
zugehörige Session gesendet. Roundtable entscheidet nicht, ob der Text eine
Zustimmung darstellt.

## Optionaler CLI-Snapshot

```text
CLI-Snapshot · Backend Review

┌──────────────────────────────────────────────┐
│ Running test suite...                        │
│ 128 tests passed                             │
│ 2 tests failed                               │
│                                              │
│ Codex is waiting for input.                  │
└──────────────────────────────────────────────┘

[Aktualisieren]
[Enter] [Esc] [Ctrl+C]
[Text senden]
[Weitere Tasten]
```

Die Ansicht ist ein Snapshot derselben CLI-Session, die Roundtable tunnelt. Sie
ist eine Diagnosefunktion und suggeriert keinen kontinuierlichen Live-Stream.
Der normale Arbeitsfluss bleibt die gemeinsame Inbox. Für die vollständige
native Bedienung kann der Benutzer die Session lokal mit tmux öffnen.

## Fortschrittsausgaben

Flüchtige Statusausgaben dürfen dieselbe Telegram-Nachricht aktualisieren:

```text
Claude Code · Backend Feature
Status: arbeitet

Projekt analysiert
Tests werden ausgeführt
```

Eine neue stabile Nachricht wird erzeugt, sobald:

- der Agent eine Eingabe erwartet,
- eine Freigabe nötig ist,
- ein Fehler auftritt,
- ein klarer Abschluss vorliegt,
- eine längere, eigenständige Antwort vorliegt.

Die Erkennung darf agentenspezifisch sein. Bei Unsicherheit wird die Ausgabe als
normale inhaltstreue Ausgabe behandelt.

## Benachrichtigungsmodi

```text
Inbox-Modus für Backend Feature:

[Alle stabilen Ausgaben ✓]
[Nur Rückfragen]
[Nur Freigaben]
[Nur Abschluss]
[Nur Fehler]
[Stumm]
```

„Stumm“ beendet oder pausiert die Session nicht. Der vollständige lokale
Verlauf bleibt erhalten.

## Fehlerdarstellung

Fehler nennen konkrete Session und Wirkung:

```text
Backend Review · Codex
Eingabe konnte nicht zugestellt werden.

Grund: Session Host ist nicht erreichbar.
Die Nachricht wurde nicht an eine andere Session gesendet.

[Erneut versuchen]
[Status prüfen]
[CLI-Snapshot]
[Details]
```

Technische Details werden auf Wunsch gezeigt, nicht ungefiltert in jede
Inbox-Nachricht geschrieben.

## Schutz vor Fehlbedienung

- Stoppen, Löschen, Worktree-Entfernung und riskante Neustarts benötigen eine
  Bestätigung.
- Jeder Bestätigungsdialog nennt Session, Agent und Projekt.
- Alte Inline-Buttons sind idempotent und dürfen keine Aktion zweimal auslösen.
- Bei unklarer Reply-Zuordnung wird nicht geraten.
- Der Default-Wechsel ist explizit und wird protokolliert.
- Eine Agentennachricht kann die Default-Session nicht selbst ändern.
- Sensible Inhalte können vor dem Versand blockiert oder maskiert werden.
