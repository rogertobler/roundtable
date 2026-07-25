# Entwicklungs-Sessions

Dieses Verzeichnis dokumentiert jede Arbeits- und Entscheidungssession am
Roundtable-Projekt.

Eine Entwicklungs-Session ist eine zusammenhängende Arbeitssitzung zwischen
dem Projektverantwortlichen und einem menschlichen oder AI-basierten
Mitarbeiter. Sie ist nicht mit einer von Roundtable verwalteten
Claude-/Codex-Runtime-Session zu verwechseln.

## Zweck

Ein Sessionbericht hält nachvollziehbar fest:

- welches Ziel die Session hatte,
- welcher Ausgangsstand vorlag,
- welche Anforderungen und Präzisierungen diskutiert wurden,
- welche Entscheidungen getroffen wurden,
- warum diese Entscheidungen getroffen wurden,
- welche Alternativen geprüft oder verworfen wurden,
- welche Dateien und Komponenten geändert wurden,
- welche Tests oder Recherchen durchgeführt wurden,
- welche Commits entstanden,
- welche Punkte offen blieben,
- wie die nächste Session sinnvoll fortsetzt.

Damit kann eine spätere menschliche oder AI-Session verstehen, was bereits
versucht, entschieden und begründet wurde, ohne den ursprünglichen Chat kennen
zu müssen.

## Verhältnis zur normativen Dokumentation

Sessionberichte sind historische Arbeitsprotokolle. Sie dürfen bewusst
Diskussionen, Zwischenstände und später verworfene Ansätze enthalten.

Bei einem Widerspruch gilt folgende Reihenfolge:

1. `README.md` und `docs/README.md`,
2. die aktuellen normativen Dokumente `docs/01` bis `docs/09`,
3. datierte Architekturänderungen unter `docs/history/`,
4. Sessionberichte unter `docs/sessions/`.

Ein Sessionbericht ersetzt keine normative Dokumentation. Ändert eine Session
eine verbindliche Produkt- oder Architekturentscheidung, müssen im selben
Arbeitsgang auch die betroffenen normativen Dokumente aktualisiert und ein
Eintrag unter `docs/history/` angelegt werden.

## Benennung

```text
YYYY-MM-DD-NNN-kurzer-titel.md
```

Beispiel:

```text
2026-07-25-001-product-definition-and-architecture.md
```

`NNN` ist eine fortlaufende Sessionnummer innerhalb des Projekts. Eine über
mehrere technische Unterbrechungen fortgesetzte, inhaltlich
zusammenhängende Arbeit kann denselben Bericht weiterführen. Eine neue
Zielsetzung erhält eine neue Sessionnummer.

## Pflichtinhalt

Jeder Bericht verwendet die Vorlage aus
[`_template.md`](_template.md) und enthält mindestens:

- Session-ID, Datum, Status und Beteiligte,
- Ziel und Ausgangslage,
- besprochene Punkte,
- Entscheidungen mit Begründung,
- verworfene oder vertagte Alternativen,
- durchgeführte Änderungen,
- Verifikation,
- offene Punkte und nächsten Schritt.

Relevante Aussagen sollen konkret sein. „Architektur verbessert“ reicht nicht;
der Bericht muss benennen, welche Architekturentscheidung sich warum geändert
hat.

## Pflege

- Der Bericht wird während der Session fortlaufend ergänzt.
- Vor dem Abschluss einer Session wird er auf Vollständigkeit geprüft.
- Relevante Commit-Hashes werden nach dem Commit ergänzt.
- Spätere Korrekturen werden als datierter Nachtrag kenntlich gemacht.
- Bestehende historische Aussagen werden nicht still an einen neuen Stand
  angepasst.
- Reine Formatkorrekturen dürfen ohne Nachtrag erfolgen.

## Sicherheit

Sessionberichte dürfen keine Secrets enthalten:

- keine Telegram-Bot-Tokens,
- keine API-Schlüssel,
- keine Passwörter,
- keine privaten Authentifizierungsdateien,
- keine vollständigen sensiblen Terminalausgaben,
- keine unnötigen personenbezogenen Daten.

Secrets werden durch neutrale Bezeichnungen wie `<telegram-bot-token>`
ersetzt. Sicherheitsrelevante Entscheidungen und Vorfälle werden trotzdem
inhaltlich dokumentiert.
