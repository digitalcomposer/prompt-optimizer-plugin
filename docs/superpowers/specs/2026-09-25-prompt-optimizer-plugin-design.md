# Prompt-Optimizer Plugin — Design

Status: approved (Design-Phase), 2026-09-25

## Ziel

Ein Claude-Code-Plugin, das rohe, unterspezifizierte Nutzer-Prompts vor
der eigentlichen Bearbeitung proaktiv über einen Subagenten optimiert
(umstrukturiert), auf Basis der in `/Volumes/FIT/AI_AI_WISSEN/prompt-engineering-wissen.md`
destillierten Prompting-Techniken. Muss auf anderen Rechnern
installierbar sein (portables Plugin, kein rechnerspezifischer Hack).

## Nicht-Ziele

- Kein automatisches Abfangen JEDES Prompts über einen Hook — das
  Hauptmodell entscheidet proaktiv, wann Optimierung sinnvoll ist
  (Nutzerentscheidung vom 2026-09-25).
- Kein manueller Slash-Command als primärer Trigger — proaktiv statt
  explizit.
- Keine Least-to-Most-Eskalationslogik (braucht Feedback-Schleife,
  passt nicht in einen Single-Pass-Optimizer).
- Kein Temperature-Handling (bei aktuellen Claude-Modellen kein
  nutzbarer API-Parameter mehr, siehe Wissensdatei Abschnitt 7).
- Der Subagent löst die eigentliche Aufgabe nicht selbst — er gibt nur
  den umgeschriebenen Prompt zurück.

## Architektur

Repo-Layout (Standard-Plugin-Marketplace-Struktur, wie beim bereits
installierten `openai-codex`-Plugin beobachtet):

```
prompt-optimizer-plugin/                     (dieses Repo)
  .claude-plugin/
    marketplace.json                         listet das Plugin
  plugins/
    prompt-optimizer/
      .claude-plugin/
        plugin.json                          Name, Version, Beschreibung, Autor
      agents/
        prompt-optimizer.md                  Subagent (schlanke Weiterleit-Hülle)
      skills/
        prompt-optimizing/
          SKILL.md                           Rewrite-Logik/Heuristik (operatives Wissen)
      README.md                              Nutzung + Installationsanleitung
  docs/superpowers/specs/                    dieses Dokument
  README.md                                  Repo-Übersicht (Top-Level)
```

## Komponenten

### Subagent — `plugins/prompt-optimizer/agents/prompt-optimizer.md`

- Frontmatter: `name: prompt-optimizer`, `model: haiku` (Haiku 4.5 —
  reine Textumformung, kein Thinking/Reasoning-Tiefe nötig, minimiert
  Latenz/Kosten pro proaktivem Aufruf), **keine `tools`** (bewusst: der
  Subagent soll nicht selbst Dateien lesen, Befehle ausführen oder die
  Aufgabe lösen — nur der Rohprompt-Text ist sein Input), `skills:
  [prompt-optimizing]`.
- `description` ist der proaktive Trigger für das Hauptmodell (analog
  zum bereits vorhandenen `codex-rescue`-Subagenten). Muss klar
  benennen:
  - **Greift bei:** vagen, unterspezifizierten Anfragen; Anfragen ohne
    erkennbare Constraints/Format; komplexen Analyse-/Erstellungs-Tasks,
    die klar von Struktur profitieren würden.
  - **Greift NICHT bei:** kurzen Bestätigungen/Antworten ("ja", "danke",
    "mach weiter"), Folgefragen innerhalb einer laufenden Aufgabe,
    bereits klar strukturierten Prompts (Rolle+Kontext+Constraints schon
    vorhanden), rein konversationellen Nachrichten.
- Systemprompt-Inhalt: lädt/befolgt die `prompt-optimizing`-Skill,
  gibt **ausschliesslich** den umgeschriebenen Prompt als finalen Report
  zurück — keine Präambel, keine Meta-Kommentare, keine Lösung der
  eigentlichen Aufgabe.

### Skill — `plugins/prompt-optimizer/skills/prompt-optimizing/SKILL.md`

Enthält die operative Entscheidungslogik (destilliert aus
`prompt-engineering-wissen.md`, aber für maschinelles Befolgen
formuliert statt für menschliches Lernen):

1. **Task-Typ klassifizieren:**
   - Strukturierter Content-Erstellungs-Task (Guide/Report/Anleitung
     mit erkennbaren Phasen schreiben) → **OPAL**-Template
     (Observations/Process/Action/Limitations).
   - Alles andere (Default) → **3Cs**-Template (Context/Clarity/
     Constraints).
2. **Zusatzprüfung — analytisch/mehrfaktoriell?** (warum/wie beeinflusst
   X Y, Vergleiche, Ursachenketten, Multi-Faktor-Fragen) → Self-Ask-
   Instruktion an das gewählte Template anhängen: *"Zerlege die Frage
   zuerst in Teilfragen, beantworte jede einzeln, synthetisiere dann die
   Endantwort."*
3. **Template füllen:** nur mit Informationen, die aus dem Originalprompt
   ableitbar sind. Nie Fakten/Details erfinden, die der Nutzer nicht
   impliziert hat — nur strukturieren und klären, nicht inhaltlich
   ergänzen.
4. **Bereits gut strukturierte Prompts:** minimal/unverändert
   zurückgeben statt künstlich aufzublähen (Prompt-Länge ist kein
   Qualitätsmerkmal).
5. **Explizit nicht anwenden:** Least-to-Most (keine Eskalationsstufen),
   Temperature (kein API-Parameter bei aktuellen Modellen, keine
   Kreativitäts-Anweisung als Ersatz vorschlagen).
6. **Output-Vertrag:** Rückgabe ist ausschliesslich der umgeschriebene
   Prompt-Text. Keine Erklärung, kein "Hier ist dein optimierter
   Prompt:", keine Bearbeitung der eigentlichen Aufgabe.

## Datenfluss

```
Nutzer-Prompt
  → Hauptthread prüft proaktiv anhand Subagent-Beschreibung, ob
    Optimierung sinnvoll ist
  → (falls ja) Agent-Tool-Aufruf: subagent_type=prompt-optimizer,
    Prompt = Rohtext des Nutzers
  → Subagent lädt prompt-optimizing-Skill, klassifiziert, wendet
    Template (+ optional Self-Ask) an
  → Subagent gibt NUR den umgeschriebenen Prompt zurück
  → Hauptthread zeigt kurze Zeile "Prompt optimiert zu: ..."
    (Transparenz-Entscheidung vom 2026-09-25)
  → Hauptthread bearbeitet die Aufgabe selbst mit dem optimierten
    Prompt als wirksamer Instruktion (kein weiterer Hop nötig — die
    Rückkehr zum Hauptthread ist bereits die "Weiterleitung")
```

## Fehlerbehandlung

- **Fail-open:** Schlägt der Subagent-Aufruf fehl oder liefert er
  unbrauchbaren Output, arbeitet der Hauptthread mit dem
  Original-Prompt weiter. Kein hartes Gate — Qualitätsverbesserung,
  keine Sicherheitsfunktion.
- **Scope-Guard gegen Selbstständig-Aufgabenlösen:** keine Tools im
  Subagenten + explizite Anweisung in Skill und Subagent-Systemprompt.
- **Über-Triggering vermeiden:** die Ausschlussliste in der
  `description` (kurze Bestätigungen, Folgefragen, bereits klare
  Prompts) ist die primäre Gegenmassnahme gegen unnötige Aufrufe bei
  trivialen Nachrichten.

## Testing

Kein klassisches automatisiertes Testing möglich (Prompt-Engineering,
kein deterministischer Code). Stattdessen manuelle Eval-Checkliste mit
6–8 Beispielprompts, einmal nach dem Bau live in der Session
durchgespielt:

1. Vager Prompt ohne Struktur → sollte optimiert werden (3Cs).
2. Struktur-Content-Task ("schreib mir einen Guide zu X") → sollte
   OPAL-Template auslösen.
3. Analytische Mehrfaktor-Frage ("wie beeinflusst X Y und Z") → sollte
   Self-Ask-Zusatz auslösen.
4. Bereits gut strukturierter Prompt (Rolle+Kontext+Constraints
   vorhanden) → sollte unverändert/minimal zurückkommen.
5. Triviale Bestätigung ("ja, mach das") → Subagent sollte NICHT
   aufgerufen werden.
6. Kurze Faktenfrage ("wie spät ist es in Tokio") → Subagent sollte
   NICHT aufgerufen werden.

## Verteilung / Installation auf anderen Rechnern

1. Lokal entwickeln/testen in diesem Repo.
2. Repo auf GitHub pushen (privat möglich — enthält keine
   personenbezogenen Daten, nur Plugin-Logik).
3. Auf einem anderen Rechner mit Claude Code:
   `/plugin marketplace add <github-repo-url>`
   `/plugin install prompt-optimizer`
4. Exakte CLI-Syntax wird beim Bau gegen die tatsächlich installierte
   Claude-Code-Version verifiziert (nicht geraten) — Details fliessen
   ins README.

## Offene Punkte für die Implementierungsplanung

- Exakter Wortlaut der `description`-Felder (Subagent + Skill) — muss
  beim Bau iterativ gegen die Eval-Checkliste geschärft werden.
- Ob `marketplace.json` lokal (Pfad-Source) zusätzlich zur
  GitHub-Variante getestet werden soll, um vor dem Push schon lokal
  installierbar zu sein.
