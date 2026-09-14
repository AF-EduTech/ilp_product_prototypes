# Intelligent Learning Assistant – Version 1

## Leitidee

Der ILA V1 ist kein zweiter Kurs und keine Suchmaschine. Er ist ein Gesprächspartner mit drei Eigenschaften:

1. Er kennt den **Lernstand** des Lernenden (aus der Lerndatenmessung).
2. Er kennt den **Kursinhalt** (Storyboards aus der Content Factory).
3. Er kennt die **Musterlösungen nicht**.

Punkt 3 ist die zentrale Designentscheidung. Was nicht im Kontext des Modells liegt, kann es nicht ausgeben — kein Prompt-Trick hilft dagegen.

---

## Das Modus-Modell

Ein Chat, drei Modi. Der Modus wird **nicht vom Modell entschieden**, sondern deterministisch aus dem Zustand der Content Delivery abgeleitet, und ist im Header immer sichtbar.

| | **Erklärmodus** (Standard) | **Übungsmodus** | **Prüfungsmodus** |
|---|---|---|---|
| Wann | Kein offenes interaktives Element | Interaktives Element geöffnet, noch nicht abgegeben | Abschlussquiz |
| Sieht Kursinhalte | ja (Text/Bild-Storyboards, alle bereits freigeschalteten Unterkapitel) | ja | – |
| Sieht Aufgabenstellung | – | ja | – |
| Sieht Musterlösung | **nie** | **nie** | – |
| Verhalten | erklärt, fasst zusammen, gibt Beispiele | Hinweis-Leiter, keine Lösungsvorschläge | ILA ist deaktiviert |
| Header-Badge | „Erklärmodus" | „Übungsmodus – ich gebe Hinweise, keine Lösungen" | ausgegraut |

Nach der Abgabe eines Elements schaltet der ILA für genau dieses Element in den Erklärmodus: *„Jetzt, wo du abgegeben hast, gehe ich Frage 2 gern komplett mit dir durch."* Das ist der Anreiz-Mechanismus — nicht Verbot, sondern Vertagung.

---

## Frage 1: Wie nutzen wir Lernfortschritt und Quiz-Erfolg?

### Der Kontext-Snapshot

Bei jeder Anfrage liefert die Rails-API einen kompakten Snapshot aus den Materialized Views der Lerndatenmessung an das AI-Backend:

```json
{
  "position":     { "modul": 3, "kapitel": 3, "unterkapitel": "3.3" },
  "pace_ratio":   1.6,
  "zeit_im_uk":   { "ist_min": 22, "soll_min": 60 },
  "quiz":         { "letzter_score": 0.33, "versuche": 1,
                    "falsche_lernziele": ["LZ-2.3-b", "LZ-3.1-a"] },
  "historie":     { "score_7d": 0.61, "inaktiv_tage": 0 },
  "indikatoren":  ["quiz_schwierigkeiten"]
}
```

Entscheidend ist `falsche_lernziele`: Quiz-Items sind in der Content Factory bereits Lernzielen zugeordnet. Damit wird aus „58 % falsch" ein **inhaltlicher** Befund statt einer Zahl.

### Drei Verwendungen

**a) Kalibrierung der Antwort.** Der Snapshot geht in den System-Prompt und steuert Erklärtiefe, Sprachniveau und Beispieldichte.

| Signal | Wirkung auf die Antwort |
|---|---|
| Score < 60 % im aktuellen Unterkapitel | einfachere Sprache, ein Alltagsbeispiel vorschalten, kleinere Schritte |
| Pace-Ratio > 1.4 | kürzere Antworten, ein Fokusthema statt Vollständigkeit |
| Pace-Ratio < 0.7 und Score hoch | knapper, dafür Vertiefungs-/Transferfragen |
| 2+ Wiederholungsversuche am selben Quiz | Wechsel der Erklärmethode statt Wiederholung derselben Erklärung |

**b) Proaktive Meldungen.** Der ILA meldet sich von selbst. Die Trigger sind **regelbasiert**, nicht vom Modell entschieden — sonst wird das Verhalten unvorhersehbar und nervt.

| Trigger | Regel | Meldung |
|---|---|---|
| Quiz nicht bestanden | Score < 60 % | benennt das schwache Lernziel, verweist auf 1–2 frühere Content-Elemente |
| Steckengeblieben | Zeit im UK > 2× Soll | fragt nach, bietet Zusammenfassung des UK an |
| Wiederholt gescheitert | 3. Fehlversuch am selben Quiz | bietet Kontakt zum Lernbegleiter an |
| Wiederaufnahme | Login nach ≥ 3 Tagen Pause | kurzer Rückblick: „Wo du stehen geblieben bist" |
| Guter Lauf | 3 Quizze in Folge ≥ 80 % | Bestätigung, optionale Vertiefung |

Grenzen: **max. 1 proaktive Meldung pro Unterkapitel**, Cooldown 20 Minuten, und ein „Ich komm alleine weiter"-Button, der für dieses Unterkapitel alles stummschaltet. Ohne diese Bremsen wird der Assistent zum Störfaktor.

**c) Steuerung des Retrieval.** `falsche_lernziele` bestimmt, welche früheren Content-Elemente der ILA in den Kontext holt und als Leseempfehlung verlinkt. Voraussetzung ist eine Lernziel-Verknüpfung über Modulgrenzen hinweg — für V1 reicht die flache Zuordnung „Content-Element → Lernziel(e)" aus dem Feinkonzept.

### Der Rückkanal ist bewusst schmal

Laut Anforderung sind Aktionen und Inhalte des ILA nicht für Lernbegleiter sichtbar. Daher gilt eine **Einbahnstraße**:

```
Lerndatenmessung ──[voller Snapshot]──▶ ILA
Lerndatenmessung ◀──[nur: ila_opened, ila_message_sent, Zeitstempel]── ILA
```

Aus dem ILA fließt nur zurück, *dass* er genutzt wurde — nie *was* besprochen wurde. Diese Vertraulichkeit ist auch ein Feature: Lernende stellen einem Bot Fragen, die sie einem Menschen nicht stellen würden. Sie muss im UI explizit benannt werden.

---

## Frage 2: Allgemeine Fragen stellen *und* Unterstützung bekommen

Das ist kein Widerspruch, sondern ein Routing-Problem. Ein Eingabefeld, dahinter eine Intent-Klassifikation:

| Intent | Beispiel | Verhalten |
|---|---|---|
| **Wissensfrage** | „Was ist CIDR?" | Direkte Erklärung per RAG über die Kursinhalte, mit Verweis auf das Content-Element. **Keine Gegenfrage.** |
| **Aufgabenbezogen** | „Wie löse ich Frage 3?" | Übungsmodus, Hinweis-Leiter |
| **Organisatorisch** | „Wann ist die Abgabe?", „Wo finde ich das Arbeitsblatt?" | Faktenantwort aus Kursstruktur und Kurstermin |
| **Lernstrategie / Meta** | „Ich komme nicht klar", „Wie merke ich mir das?" | Lernstrategie-Repertoire, ggf. Eskalation zum Lernbegleiter |
| **Off-Topic** | alles andere | freundlich zurück zum Kurs, ein Satz |

### Sokratisch heißt nicht „nie antworten"

Die häufigste Fehlinterpretation. Ein Lernender, der wissen will, was ein Broadcast-Adressbereich ist, und stattdessen eine Gegenfrage bekommt, schließt den Chat und geht zu ChatGPT. Die Regel für V1:

> **Sokratik greift bei Lösungswegen, nicht bei Definitionen.**
> Begriffe, Fakten und Zusammenhänge werden erklärt. Ausgefragt wird nur dort, wo der Lernende gerade selbst denken soll — also im Übungsmodus und bei „Warum war meine Antwort falsch?".

### Unterstützung ist proaktiv, nicht aufdringlich

Die eigentliche Unterstützung passiert über die Trigger aus Frage 1 — nicht dadurch, dass jede Frage in ein Coaching-Gespräch umgebogen wird. Zusätzlich:

- **Quick-Actions** unter dem Eingabefeld als Intent-Abkürzung: „Einfacher erklären", „Beispiel", „Zusammenfassung", „Lerncheck wiederholen". Kontextabhängig — im Übungsmodus stattdessen „Tipp", „Aufgabe umformulieren".
- **Kontext-Strip** über dem Chat, der immer zeigt, worauf sich der ILA gerade bezieht (Modul › Kapitel › Unterkapitel). Ohne den weiß der Lernende nicht, was der Assistent eigentlich sieht.
- **Eskalation:** Der ILA kann den Lernbegleiter vorschlagen, aber nicht kontaktieren. Er formuliert einen Entwurf, der Lernende sendet ihn — sonst wäre die Vertraulichkeit gebrochen.

---

## Frage 3: Wie verhindern wir, dass der ILA Aufgaben löst?

Fünf Schichten, geordnet von deterministisch (wirksam) nach probabilistisch (unzuverlässig). Der Aufwand gehört in die ersten beiden.

### Schicht 1 — Musterlösungen sind nicht im Index

Der RAG-Index des ILA enthält Content-Elemente vom Typ Text und Bild. **Nicht enthalten:** Musterlösungen, korrekte Antwortoptionen, Assertions-Erwartungen, Lösungscode, Bewertungsraster. Für ein aktives interaktives Element bekommt der ILA nur die Aufgabenstellung und die verknüpften Lernziele.

Das ist die einzige Maßnahme, die auch gegen einen perfekten Jailbreak hält.

### Schicht 2 — Modus-Automatik

Sobald ein interaktives Element geöffnet und nicht abgegeben ist, ist der Übungsmodus aktiv. Diese Entscheidung trifft die Content Delivery, nicht das Modell.

### Schicht 3 — Aufgaben-Erkennung im Eingabefeld

Der Klassiker: der Lernende kopiert die Aufgabenstellung in den Chat und wechselt vorher das Unterkapitel. Gegenmaßnahme: Textähnlichkeit der Nachricht gegen den Aufgaben-Index des Kurses (Embedding-Vergleich, Schwellwert). Trifft es zu, greift der Übungsmodus unabhängig vom Navigationszustand.

Das gilt auch für eingefügten Code aus einem Code-Quiz.

### Schicht 4 — Hinweis-Leiter mit Budget

Im Übungsmodus eskaliert die Hilfe stufenweise. Das ist eine echte Sequenz: jede Stufe wird erst erreicht, wenn der Lernende auf der vorherigen etwas beigetragen hat.

1. **Rückfrage** – „Was hast du bisher probiert? Wo hakt es genau?"
2. **Konzept-Hinweis** – benennt das relevante Konzept und verlinkt das Content-Element, das es erklärt.
3. **Teilschritt** – zerlegt die Aufgabe in Schritte oder bietet eine Analogie mit *anderen* Zahlen/Daten.
4. **Gemeinsam durchgehen** – nur nach der Abgabe erreichbar.

Budget: **max. 3 Stufen und 4 Turns pro Aufgabe** vor der Abgabe. Danach: „Ich habe dir alles gegeben, was ohne Lösung geht. Lies nochmal Kapitel 5.3 — oder ich helfe dir, deine Lernbegleiterin zu fragen."

### Schicht 5 — Prompt-Härtung und Output-Filter

- Nutzereingaben werden im Prompt klar als Daten ausgezeichnet, nicht als Instruktionen.
- In Programmierkursen: bei offenem Code-Quiz kein zusammenhängender Code-Block im Output — Pseudocode und einzelne Syntaxzeilen mit anderem Anwendungsfall sind erlaubt. Regex-Filter auf die Ausgabe als letzte Bremse.
- Standardabweisung bei Jailbreak-Mustern.

Diese Schicht ist die schwächste. Sie ist Kosmetik, wenn Schicht 1 fehlt — und weitgehend entbehrlich, wenn Schicht 1 sitzt.

### Flankierend: Bewertungsdesign

- Quizze zählen in die Lerndatenmessung, **nicht** in eine Zertifizierung. Wer den ILA austrickst, manipuliert nur sein eigenes Dashboard. Das gehört einmal explizit gesagt — im Onboarding und beim ersten Übungsmodus-Kontakt.
- Das **Abschlussquiz** läuft im Prüfungsmodus, ILA deaktiviert. Damit gibt es genau eine Messung, die belastbar ist.

---

## Lernstrategien

In `edutech.md` steht „Aufzeigen von Lernstrategien" gleichberechtigt neben dem sokratischen Dialog. Das ist die Antwort auf eine Frage, die der Kursinhalt nicht beantworten kann: *nicht* „Was bedeutet CIDR?", sondern „Warum komme ich damit nicht weiter, obwohl ich es dreimal gelesen habe?"

Genau da ist der ILA im Vorteil gegenüber jedem allgemeinen Chatbot: Er sieht am Verhalten, **wie** jemand lernt, nicht nur was er weiß.

### Strategien sind kuratierte Objekte, keine generierte Prosa

Die zentrale Entscheidung: Das Modell **erfindet keine Strategien**. Es wählt aus einem gepflegten Katalog und formuliert den Vorschlag kontextspezifisch aus. Ohne diese Einschränkung produziert ein LLM beliebige Ratgeber-Sprache — freundlich, plausibel, wirkungslos.

Eine Strategie ist ein strukturiertes Objekt:

```json
{
  "id": "STR-K01",
  "name": "Abrufen statt Wiederlesen",
  "familie": "kognitiv",
  "wirksamkeit": "hoch",
  "kurskategorie": ["*"],
  "passt_zu": ["quiz_schwierigkeiten", "steckengeblieben"],
  "aufwand_min": 10,
  "pitch": "Wiederlesen fühlt sich nach Lernen an, ist aber das Schwächste, was du tun kannst. Abrufen aus dem Kopf ist das Stärkste.",
  "erster_schritt": "Schließ das Kapitel. Schreib mir in drei Sätzen, was hängen geblieben ist — ich gleiche es mit den Lernzielen ab.",
  "wirkung_messbar_an": "quiz_score_delta"
}
```

Das Feld `erster_schritt` ist das wichtigste:

> **Eine Strategie ohne ersten Schritt ist ein Ratschlag.**
> Der ILA nennt immer eine konkrete Mikro-Aktion, die *jetzt, in dieser Plattform* ausführbar ist — kein „achte künftig auf regelmäßige Wiederholung".

### Der Katalog

20 Strategien reichen für V1. Sie sind kursübergreifend wiederverwendbar und nur nach Kurskategorie gefiltert.

**Kognitiv — wie Inhalt verarbeitet wird**

| ID | Strategie | Erster Schritt im ILA |
|---|---|---|
| K01 | Abrufen statt Wiederlesen | „Schließ das Kapitel, schreib mir 3 Sätze aus dem Kopf." |
| K02 | Selbsterklärung | „Beantworte mir eine Warum-Frage zu dem Satz, den du gerade gelesen hast." |
| K03 | Eigenes Beispiel bilden | „Gib mir ein Beispiel aus deinem alten Beruf." |
| K04 | Struktur zeichnen | „Ordne diese fünf Begriffe — was gehört wozu?" |
| K05 | Gegenüberstellen | „Nenn mir einen Unterschied zwischen Scrum und Kanban, der dir wichtig erscheint." |

**Metakognitiv — wie das Lernen gesteuert wird**

| ID | Strategie | Erster Schritt im ILA |
|---|---|---|
| M01 | Vorab-Frage stellen | „Bevor du liest: Was willst du am Ende können?" |
| M02 | 3-Sätze-Check am Kapitelende | „Fassen wir zusammen — ich starte, du ergänzt." |
| M03 | Fehlerprotokoll führen | „Warum hast du bei Frage 2 so geantwortet? Das notiere ich mit." |
| M04 | Verteiltes Wiederholen | „Drei Fragen zu Modul 2 — 4 Minuten, jetzt gleich." |
| M05 | Stillstand abbrechen | „Nach 10 Minuten ohne Fortschritt: weiterblättern, Frage parken." |

**Ressourcen — unter welchen Bedingungen gelernt wird**

| ID | Strategie | Erster Schritt im ILA |
|---|---|---|
| R01 | Sitzungslänge begrenzen | „Du bist seit 95 Min. dran. Mach 10 Min. Pause — ich merke mir, wo wir waren." |
| R02 | Feste Lernslots | „Wann passt es dir morgen? Ich trag's im Tagesplan als Notiz ein." |
| R03 | Ablenkung ausschalten | „Du warst in dieser Sitzung 12× in anderen Tabs. Ein Durchgang ohne?" |
| R04 | Fragen sammeln statt hängenbleiben | „Park die Frage, ich erinnere dich am Kapitelende daran." |

**Kategoriespezifisch**

| Kurskategorie | Strategie | Anlass |
|---|---|---|
| Programmierkurs | Fehlermeldung zuerst lesen und übersetzen | viele Code-Quiz-Versuche in kurzer Zeit |
| Programmierkurs | Rubber Ducking — Code laut erklären | Code-Explanation-Quiz scheitert |
| Programmierkurs | Kleinste Änderung, dann testen | Trial-and-Error-Muster |
| Methodikkurs | Transfer auf den eigenen Arbeitsalltag | Score gut, Transferfragen falsch |
| Methodikkurs | Rollenperspektive wechseln | Verwechslung von Rollen/Verantwortlichkeiten |
| Tooltraining | Blinddurchlauf ohne Anleitung | schnelles Durchklicken, schwacher Score |
| Tooltraining | Fehlerfall provozieren und beheben | Oberflächenlernen |

Gepflegt wird der Katalog vom **DLD in der Content Factory** — als eigener Objekttyp neben den Content-Elementen, ohne Bindung an einen konkreten Kurs.

### Auswahl: von der Zahl zur Diagnose

Die Zuordnung passiert **regelbasiert vor** dem LLM-Aufruf. Der Snapshot liefert nicht nur Werte, sondern ein Muster — und das Muster ist die Diagnose.

| Muster in den Lerndaten | Diagnose | Strategie |
|---|---|---|
| Score < 60 %, Zeit im Soll | Verstehensproblem, kein Fleißproblem | K02, K03 |
| Score < 60 %, Zeit > 2× Soll | ineffiziente Verarbeitung — liest immer wieder | **K01**, M02 |
| Score im UK gut, in Wiederholungsfragen schlecht | Vergessenskurve | M04 |
| Pace > 1.4, Sitzungen > 90 Min. | Ressourcenproblem, nicht Verständnisproblem | R01, R03 |
| Inaktivitäts-Gaps, dann Marathonsitzungen | Zeitmanagement | R02 |
| Pace < 0.7, Score hoch, Transferfragen falsch | Oberflächenlernen | K05, Kategorie-Transfer |
| Viele Quizversuche in sehr kurzer Folge | Raten statt Denken | M03, M05 |

Der wichtigste Fall ist Zeile 2: **hohe Zeit plus schwacher Score** wird intuitiv als Fleiß gelesen und mit „schau's dir nochmal an" beantwortet — also mit genau der Strategie, die nicht funktioniert. Der ILA muss hier gegen die Intuition beraten.

### Der Regelkreis

Ein Tipp, der einmal ausgesprochen und nie wieder erwähnt wird, ist ein Glückskeks. Was den ILA von einem Ratgeber unterscheidet, ist die Rückkopplung über die Lerndaten:

```
Vorschlag ──▶ Zusage ──▶ Erinnerung ──▶ Rückfrage ──▶ Wirkungsprüfung
                                                            │
              ┌─────────────────────────────────────────────┘
              ▼
       bestätigen  oder  Strategie wechseln
```

| Schritt | Wann | Was passiert |
|---|---|---|
| Vorschlag | bei passendem Muster, max. 1× pro Unterkapitel | Strategie-Karte im Chat mit erstem Schritt als Button |
| Zusage | sofort | „Probier ich" heftet die Strategie an — sichtbar in der Sidebar |
| Erinnerung | beim nächsten Einstieg | ein Satz, kein Dialog: „Du wolltest 3 Sätze aus dem Kopf schreiben." |
| Rückfrage | nach dem nächsten Quiz | „Hat's was gebracht?" — Antwort ist optional |
| Wirkungsprüfung | automatisch | Score-Delta / Pace-Delta gegenüber dem Zeitraum davor |

Bleibt die Wirkung über zwei Quizze aus, schlägt der ILA eine Strategie aus einer **anderen Familie** vor — nicht eine Variante derselben. Wer mit Ressourcenstrategien nicht weiterkommt, hat vermutlich ein Verständnisproblem und umgekehrt.

### Leitplanken

| Regel | Begründung |
|---|---|
| **Max. eine aktive Strategie** | Wer fünf Strategien hat, hat keine. Neuer Vorschlag erst, wenn der alte quittiert oder abgelaufen ist. |
| **Nie im Übungsmodus als Ausweichmanöver** | „Ich darf dir nicht helfen, aber hier ein Lerntipp" ist zynisch. Strategien kommen im Erklärmodus oder nach der Abgabe. |
| **Keine Lerntypen** | Visuell / auditiv / kinästhetisch ist fachlich widerlegt. Gehört als explizites Verbot in den System-Prompt — es ist das Erste, was ein Modell sonst anbietet. |
| **Keine Motivationssprüche** | Der ILA benennt ein beobachtetes Muster und eine Handlung. Keine Aufmunterung ohne Befund. |
| **Strategie ersetzt nie eine Antwort** | Auf eine Wissensfrage folgt eine Erklärung. Strategien kommen zusätzlich, nie anstelle. |
| **Vertraulich wie alles andere** | Zusage, Verlauf und Fehlerprotokoll bleiben im ILA. An die Lerndatenmessung geht nur `ila_strategy_offered` / `ila_strategy_accepted`. |

### Was das für die Lerndatenmessung bedeutet

Zwei zusätzliche Ereignisse (Kategorie 1, Aktivitätsnachweise) und eine neue Auswertung: **Welche Strategien wirken?** Über alle Lernenden anonym aggregiert lässt sich das Score-Delta nach Annahme messen — Grundlage für die Katalogpflege durch die DLDs. Da nur IDs und Kennzahlen aggregiert werden, kollidiert das nicht mit der Vertraulichkeitszusage.

Ein neuer Problemindikator kommt dazu: **Strategie-resistent** — zwei angenommene Strategien aus verschiedenen Familien ohne messbare Wirkung. Das ist ein starkes Signal für die Lernbegleitung, und zwar ohne Preisgabe von Gesprächsinhalten.

---

## Scope V1

**Drin**
- Chat-Panel neben dem Kursinhalt, Kontext-Strip, Modus-Badge
- Kontext-Snapshot aus der Lerndatenmessung
- 5 proaktive Trigger mit Cooldown und Stummschaltung
- Intent-Routing über 5 Intents
- RAG über Storyboards des laufenden Kurses
- Modus-Automatik, Hinweis-Leiter, Aufgaben-Erkennung
- Vorschlag „Lernbegleiter kontaktieren" (Entwurf, kein Versand)
- Strategie-Katalog mit ~20 kuratierten Einträgen, ein aktiver Slot, Regelkreis über 2 Berührungen
- Events `ila_opened` / `ila_message_sent` / `ila_strategy_offered` / `ila_strategy_accepted`

**Raus (bewusst)**
- Sprachein- und -ausgabe
- Gedächtnis über Sessions hinweg (V1: Historie nur innerhalb der Session, plus Snapshot)
- Einsicht für Lernbegleiter in ILA-Inhalte
- Bild-/Diagrammgenerierung im Chat
- Kursübergreifender Zugriff (Umschulung: nur das laufende Lernfeld)
- Feintuning eines eigenen Modells
- Lernstil- oder Lerntypdiagnostik (fachlich widerlegt, bewusst ausgeschlossen)
- Generierte statt kuratierte Strategien
- Strategie-Profil über Lernfelder einer Umschulung hinweg

---

## Technische Einordnung

Passt in den bestehenden Stack, keine neue Komponente:

```
React (Content Delivery)
  │  Chat-Panel, Modus aus lokalem Zustand
  ▼
Rails API  /ila/message
  │  holt Kontext-Snapshot aus Materialized Views
  │  entscheidet Modus serverseitig (nicht dem Client vertrauen)
  ▼
Python AI-Backend
  │  Intent-Klassifikation → Prompt-Auswahl
  │  RAG über Storyboard-Index (ohne Lösungen)
  │  LLM-Aufruf
  ▼
Rails ─▶ Antwort + Content-Element-Verweise
     └─▶ Event an Lerndatenmessung (nur Metadaten)
```

- Der Storyboard-Index wird beim Freigeben der Storyboards in der Content Factory aufgebaut — dort ist ohnehin bekannt, welches Element Lösung ist und welches nicht. **Die Trennung passiert an dieser Stelle, nicht zur Laufzeit.**
- Modus-Entscheidung serverseitig, weil ein manipulierter Client sonst dauerhaft „Erklärmodus" behaupten könnte.
- Retention und Löschkonzept für Chat-Verläufe analog zur Lerndatenmessung, aber mit eigener Rechtsgrundlage — die Verläufe sind vertraulich und dürfen nicht in die 36-Monats-Auswertung.

---

## Offene Entscheidungen

| Punkt | Frage |
|---|---|
| Arbeitsblatt / Selbstreflexion | Es gibt keine automatische Bewertung — wann schaltet der ILA frei? Vorschlag: mit dem Absenden. |
| Missbrauchs-Statistik | Darf die Zahl abgewiesener Lösungsanfragen **anonym aggregiert** an DLDs gehen (Signal für zu schwere Aufgaben)? Berührt die Vertraulichkeitszusage. |
| Modellwahl | Generierung läuft heute über ChatGPT. Für den ILA ist Latenz kritischer als Qualität — eigenes Modell/Tier prüfen. |
| Session-Länge | Wie viel Historie geht in den Kontext? Vorschlag V1: letzte 10 Turns plus Snapshot. |
| Umschulung | Darf der ILA in Lernfeld 12 auf Inhalte aus Lernfeld 3 verweisen? Inhaltlich sinnvoll, erhöht aber Index-Größe und Kosten. |
| Strategie-Katalog | Wer pflegt ihn — jeder DLD für seine Kurse oder eine zentrale Redaktion? Kursübergreifende Wiederverwendung spricht für zentral. |
| Indikator „strategie-resistent" | Geht er ins Lernbegleiter-Dashboard? Er verrät keine Inhalte, macht aber sichtbar, dass jemand den ILA genutzt hat. |
