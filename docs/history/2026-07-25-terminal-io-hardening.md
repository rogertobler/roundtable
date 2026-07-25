# Terminal-I/O-Härtung

Datum: 25. Juli 2026

## Ausgangslage

Zwei externe technische Reviews bestätigten das Reply-/Default-Routing, hoben
aber dieselbe Hauptrisikozone hervor: Roundtable muss rohe tmux-Ausgabe in
stabile Messengerblöcke überführen und Messengertext in eine interaktive CLI
schreiben, während dieselbe Session lokal attachbar bleibt.

Die erste Baseline beschrieb diese Pfade bereits, legte aber konkurrierende
lokale/mobile Writes, Teilabstürze während einer Zustellung, Reattach-Identität
und Terminalmaße noch nicht streng genug fest.

## Entscheidung

- Ein normaler interaktiver tmux-Client und Roundtable schreiben
  standardmäßig nie gleichzeitig in dasselbe Pane. Eine mobile Übernahme
  detached normale Clients erst nach ausdrücklicher Bestätigung.
- Die Agenten-CLI läuft direkt als Pane-Prozess ohne Shell-Fallback.
- Jeder Pane-Write besitzt persistierte Zwischenzustände. Ein Absturz nach
  Schreibbeginn führt nötigenfalls zu `delivery_uncertain`, niemals zu einem
  blinden Retry.
- Automatisches Reattach verlangt stabile tmux-IDs und Roundtable-eigene User
  Options mit Session-UUID und Runtime-Generation.
- Terminalmaße sind pro Runtime fest und gespeichert.
- Bracketed Paste wird nur verwendet, wenn die Zielanwendung es aktiviert hat;
  Multiline-Verhalten wird pro Agentenversion validiert.
- `pipe-pane` und tmux Control Mode werden im Spike verglichen. Control Mode
  gilt nicht vorab als Lösung für Nachrichtengrenzen oder TUI-Rendering.
- Raw Output bleibt wirklich raw und wird durch Rechte, Retention,
  Deaktivierbarkeit und optional Verschlüsselung geschützt. Redigierte
  Messenger-/Diagnoselogik ist davon getrennt.

## Nicht übernommen

- Kein globales `stty -echo`: Es kann interaktive Agenten-TUIs beschädigen.
- Kein erzwungenes `TERM=dumb` oder `CI=true`: Roundtable soll die native CLI
  unverändert betreiben.
- Kein reiner Text-Echo-Canceller: Legitime identische Agentenausgabe darf
  nicht verschwinden.
- Keine Behauptung, `capture-pane` könne nach einem Teilabsturz eine
  Exactly-once-Zustellung beweisen.
- Keine automatische Prozesspause allein wegen vieler Terminalzeilen.

## Auswirkungen

Der technische Spike wird vor Produktimplementierung deutlich strenger. Der
Linux-MVP enthält bereits persistente Inputzustände, Reattach-Marker und die
lokale Schreibsperre. Approval-Buttons bleiben Phase 2; der MVP tunnelt freie
Replies auf originale Approval-Prompts.

## Betroffene Dokumente

- `README.md`
- `docs/README.md`
- `docs/01-product-vision.md`
- `docs/02-functional-requirements.md`
- `docs/04-architecture.md`
- `docs/05-domain-model.md`
- `docs/06-agents-and-runtimes.md`
- `docs/07-security-and-operations.md`
- `docs/08-roadmap.md`
- `docs/09-decisions-and-open-questions.md`
- `docs/10-existing-solutions.md`
- `docs/11-ai-review-brief.md`
- `docs/12-ai-second-opinion-request.md`
