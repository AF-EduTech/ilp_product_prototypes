# Lerndatenmessung

## Messpunkte

### Kategorie 1: Aktivitätsnachweise (War jemand aktiv?)

| Messpunkt | Event | Granularität |
|---|---|---|
| Session Start/End | Login, Logout, Tab-Close | pro Session |
| Page Visibility | `visibilitychange` Event (Tab im Vordergrund?) | pro Unterkapitel |
| Scroll-Tiefe | Scroll-Position in % auf Content-Seite | pro Unterkapitel |
| Zeit auf Unterkapitel | Dauer von Öffnen bis Verlassen | pro Unterkapitel |
| ILA-Interaktion | Chatbot geöffnet, Nachricht gesendet | pro Session |

> Kombination aus Session-Zeit + Page Visibility + Scroll gibt einen belastbaren Aktivitätsnachweis — reine Login-Zeit wäre zu leicht zu manipulieren.

---

### Kategorie 2: Lernfortschritt (Wie weit ist jemand?)

| Messpunkt | Event | Granularität |
|---|---|---|
| Unterkapitel abgeschlossen | Letztes Content-Element gesehen | pro Unterkapitel |
| Kapitel abgeschlossen | Alle Unterkapitel done | pro Kapitel |
| Modul abgeschlossen | Alle Kapitel done | pro Modul |
| Kurs abgeschlossen | Alle Module done | Kurs |
| Wiederaufnahme | Kurs nach Pause fortgesetzt | pro Session |

---

### Kategorie 3: Lerngeschwindigkeit (Wie schnell/langsam?)

| Messpunkt | Berechnung | Vergleichswert |
|---|---|---|
| Zeit pro Unterkapitel | Aktive Zeit (mit Page Visibility bereinigt) | Baseline aus Pilotdaten |
| Zeit pro Modul | Summe Unterkapitel-Zeiten | Soll-Dauer (1 Woche) |
| Pace-Ratio | Ist-Zeit / Soll-Zeit | < 0.7 = schnell, > 1.4 = langsam |
| Inaktivitäts-Gaps | Tage ohne Login im Kurszeitraum | Schwellwert konfigurierbar |

---

### Kategorie 4: Lernerfolg (Wie gut ist jemand?)

| Messpunkt | Event | Aggregation |
|---|---|---|
| Quiz-Versuch | Antwort abgeschickt (richtig/falsch, Zeitstempel) | pro Quiz-Item |
| Quiz-Score | Richtige Antworten / Gesamtversuche | pro Unterkapitel |
| Wiederholungsversuche | Wie oft wurde ein Quiz wiederholt | pro Quiz |
| Arbeitsblatt eingereicht | Submit-Event mit Wort-/Zeichenzahl | pro Arbeitsblatt |
| Code-Quiz Ergebnis | Validierungsergebnis gegen Musterlösung | pro Code-Quiz |
| Lernkarten-Durchlauf | Karten durchgegangen, pass/fail pro Karte | pro Lernkarten-Set |

---

### Kategorie 5: Problemindikatoren (für Lernbegleiter-Dashboard)

Werden nicht direkt gemessen, sondern aus den Rohdaten berechnet:

| Indikator | Regel |
|---|---|
| Langsam-Lerner | Pace-Ratio > 1.4 über 3+ Tage |
| Quiz-Schwierigkeiten | Score < 60% in einem Unterkapitel |
| Inaktiv | Kein Login seit X Tagen (konfigurierbar, z.B. 3 Tage) |
| Steckengeblieben | Gleiches Unterkapitel > 2× der Soll-Zeit |
| Dropout-Risiko | Kombination aus Inaktivität + niedrigem Score |

---

## Tech-Stack

### Schicht 1: Event Collection (Frontend → Backend)

```
React (Content Delivery)
  → Custom Event Tracker (lightweight JS, ~50 Zeilen)
  → POST /api/events (Rails API Endpoint)
  → Celery Worker nimmt Events entgegen und schreibt sie weg
```

- Eigene Events ohne externen Analytics-Dienst (datenschutzkonformer, günstiger)
- Events als JSON: `{ user_id, course_id, event_type, metadata, timestamp }`
- Batching im Frontend (alle 30 Sek. oder bei Seitenwechsel) um Server-Last zu reduzieren

### Schicht 2: Event Store (Rohdaten)

```
Postgres (bestehend) + TimescaleDB Extension (kostenlos, OSS)
```

- TimescaleDB macht Postgres zur Time-Series-Datenbank: automatisches Partitionieren nach Zeit, schnelle Aggregationen über Zeiträume
- Passt direkt in Rails-Stack, kein neues System
- Retention-Policy auf 36 Monate konfigurierbar

### Schicht 3: Aggregation & Auswertung

```
Celery Worker (bestehend)
  → Periodische Jobs: Pace-Ratio berechnen, Problemindikatoren prüfen
  → Ergebnisse in Postgres Materialized Views
Rails
  → Liest Materialized Views für Dashboards (fast, weil vorberechnet)
```

- Near-Realtime durch kurze Celery-Intervalle (z.B. alle 5 Minuten)
- Materialized Views: einmal definiert, automatisch aktuell — kein eigener Data-Warehouse-Aufwand

### Schicht 4: Visualisierung

```
Lernbegleiter-Dashboard (React, bestehend)
  → Rails API liefert aggregierte Kennzahlen
  → Recharts oder Chart.js für Graphen (leichtgewichtig, Open Source)
```

---

## Architektur-Übersicht

```
React Frontend
  │  (Batch-Events alle 30s)
  ▼
Rails API /events
  │
  ▼
Celery → Postgres + TimescaleDB
              │
              ├── Raw Events (36 Monate)
              ├── Materialized Views (Pace, Score, Indikatoren)
              │
              ▼
         Rails API (Aggregierte Daten)
              │
              ▼
         Lernbegleiter-Dashboard (React)
```

---

## Anforderungsabdeckung

| Anforderung | Abgedeckt durch |
|---|---|
| Aktivitätsnachweis | Page Visibility + Session-Events + Scroll |
| Lerngeschwindigkeit | Pace-Ratio aus Zeit-Events |
| Lernerfolg | Quiz-Score-Events |
| Near-Realtime | Celery 5-Min-Jobs |
| 36 Monate Retention | TimescaleDB Retention Policy |
| Problemindikation | Materialized Views mit Schwellwert-Regeln |
| DSGVO | Eigener Stack, keine Drittanbieter |
