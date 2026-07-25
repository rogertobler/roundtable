# Roundtable-Dokumentation

Stand: 25. Juli 2026

Dieses Verzeichnis beschreibt die gemeinsame Produktvision und die geplante
technische Architektur von Roundtable. Es ist die zentrale Referenz für alle
Menschen und Agent-Sessions, die am Projekt arbeiten.

Roundtable ist eine lokale, plattformübergreifende Steuerungs- und
Kommunikationsebene für mehrere gleichzeitig laufende, interaktive
Terminal-Agenten. Telegram ist der erste Kommunikationskanal. Claude Code und
OpenAI Codex sind die ersten Agenten. Weitere Kanäle und Agenten sollen über
Adapter ergänzt werden können.

## Die Produktidee in einem Satz

> Roundtable verbindet Messaging-Kanäle mit dauerhaften Agent-Sessions:
> Antworten werden an die Ursprungssession geroutet, freie Nachrichten an die
> vom Benutzer festgelegte Default-Session.

## Verbindliche Grundprinzipien

1. **Der Agent gehört zur Session.** Es gibt keinen global aktiven Agenten.
   Claude Code, Codex und andere Agenten können gleichzeitig in beliebig vielen
   Sessions laufen.
2. **Replies bestimmen das Routing.** Eine Antwort auf eine von Roundtable
   gesendete Nachricht geht immer zurück an die Session, aus der diese Nachricht
   stammt.
3. **Die Default-Session gilt nur für freie Nachrichten.** Sie ist das Ziel
   einer Nachricht, die keine Telegram-Reply und kein explizites
   Session-Präfix enthält.
4. **Roundtable verändert keine Agenteninhalte.** Ein- und Ausgaben werden
   inhaltstreu weitergeleitet. Roundtable formuliert nicht um, beantwortet
   nichts selbst und trifft keine Entscheidungen für den Benutzer.
5. **Freigaben bleiben Agenteninteraktionen.** Roundtable zeigt die originale
   Frage an und leitet die Benutzerantwort oder den zugeordneten Tastendruck an
   exakt dieselbe Session weiter.
6. **Roundtable ist kein eigener KI-Agent.** Es ist Router, Session-Manager,
   Terminal-Brücke, Inbox und Audit-Schicht.
7. **Messaging ist keine rohe Terminalspiegelung.** Terminalartefakte wie
   ANSI-Sequenzen, Spinner und wiederholt überschriebene Zeilen dürfen für die
   Darstellung normalisiert werden. Der semantische Inhalt bleibt unverändert,
   und die rohe Ausgabe wird lokal vorgehalten.
8. **Installierbarkeit ist ein Produktmerkmal.** Roundtable soll auf Linux,
   macOS und Windows lokal sowie auf einem VPS betrieben werden können.
9. **Lokale Kontrolle bleibt erhalten.** Agenten laufen auf dem Rechner des
   Benutzers. Projekte, Zugangsdaten und vollständige Terminalausgaben müssen
   nicht an einen Roundtable-Cloud-Dienst übertragen werden.
10. **Sicherheit ist standardmäßig restriktiv.** Nur freigegebene Benutzer,
    Projekte, Verzeichnisse und Aktionen werden zugelassen.

## Beispiel

Parallel laufende Sessions können so aussehen:

| Session | Agent | Modell | Projekt | Status |
| --- | --- | --- | --- | --- |
| Backend Feature | Claude Code | Opus | DumbleScore | arbeitet |
| Backend Review | Codex | konfiguriertes GPT-Modell | DumbleScore | wartet |
| Website | Claude Code | Sonnet | Inscribe | bereit |

Das Routing bleibt auch bei parallelen Sessions eindeutig:

```text
Reply auf Nachricht aus "Backend Feature"
  -> Claude-Session "Backend Feature"

Reply auf Nachricht aus "Backend Review"
  -> Codex-Session "Backend Review"

Freie Nachricht ohne Reply
  -> festgelegte Default-Session
```

## Dokumente

- [01-product-vision.md](01-product-vision.md): Vision, Zielgruppen,
  Produktgrenzen und Anwendungsfälle
- [02-functional-requirements.md](02-functional-requirements.md): vollständiger
  geplanter Funktionsumfang und Akzeptanzregeln
- [03-telegram-ux.md](03-telegram-ux.md): Telegram-Menüs, Nachrichtenformat und
  Interaktionsabläufe
- [04-architecture.md](04-architecture.md): Komponenten, Datenflüsse,
  Schnittstellen und Deployment
- [05-domain-model.md](05-domain-model.md): Domänenobjekte, Zustände,
  Persistenz und Invarianten
- [06-agents-and-runtimes.md](06-agents-and-runtimes.md): Agent-Adapter,
  Terminal-Backends und Plattformstrategie
- [07-security-and-operations.md](07-security-and-operations.md): Bedrohungsmodell,
  Berechtigungen, Installation und Betrieb
- [08-roadmap.md](08-roadmap.md): Umsetzungsphasen, Abnahmekriterien und spätere
  Erweiterungen
- [09-decisions-and-open-questions.md](09-decisions-and-open-questions.md):
  getroffene Entscheidungen und noch offene Detailfragen
- [10-existing-solutions.md](10-existing-solutions.md): bestehende ähnliche
  Projekte, technische Referenzen und Abgrenzung

## Terminologie

| Begriff | Bedeutung |
| --- | --- |
| Roundtable Core | Kanal- und plattformunabhängige Geschäftslogik |
| Transport | Kommunikationskanal wie Telegram oder später WhatsApp |
| Session | Logische, dauerhaft identifizierbare Agenteninstanz |
| Agent | Terminalprogramm wie Claude Code oder Codex |
| Agent-Adapter | Agentenspezifischer Start, Fähigkeiten und Interaktionen |
| Runtime | Technischer Host der interaktiven Terminal-Sitzung |
| Session Host | Prozess oder Dienst, der eine PTY-Sitzung dauerhaft hält |
| Projekt | Vorkonfigurierter und freigegebener Arbeitskontext |
| Default-Session | Ziel für freie Benutzernachrichten |
| Reply-Routing | Zuordnung einer Antwort über die beantwortete Nachricht |
| Inbox | Gemeinsamer privater Chat mit Ereignissen aller abonnierten Sessions |
| Approval | Interaktive Freigabeanfrage des Agenten |
| Raw Output | Unveränderte Ausgabe des Terminalprozesses |
| Rendered Output | Für einen Transport technisch aufbereitete Darstellung |

## Status dieses Dokumentsatzes

Die Dokumente beschreiben das gewünschte Zielprodukt. Die Roadmap legt fest, in
welcher Reihenfolge dieses Ziel umgesetzt wird. Eine Funktion, die erst in einer
späteren Phase steht, ist damit nicht aus dem Produktumfang gestrichen.

Abweichende Implementierungsentscheidungen sollen in
`09-decisions-and-open-questions.md` festgehalten werden. Dadurch bleibt
nachvollziehbar, ob eine Abweichung eine bewusste Entscheidung oder nur ein
ungeklärtes Implementierungsdetail ist.
