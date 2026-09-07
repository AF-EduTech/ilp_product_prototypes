# Risikobewertung

## Zusammenfassung

Ein etablierter Entwicklungsprozess und der gezielte Einsatz erprobter, passender Technologien halten die üblichen Risiken im ILP-Entwicklungsprozess niedrig. Ein Kriterium (Abhängigkeit von Drittanbietern) ist mit Gelb bewertet, alle übrigen mit Grün.

## Truck-Factor

Grün

**Erläuterung**

- **Grün - Content Factory:** keine Wissensinseln, Dokumentation aktuell; Frontend- wie Backend-/Infrastruktur-Erfahrung auf 4 Entwickler verteilt.
- **Grün - Content Delivery:** keine Wissensinseln, Dokumentation aktuell; Frontend- wie Backend-/Infrastruktur-Erfahrung auf 3 Entwickler verteilt.
- **Grün - Lerndatenmessung:** keine Wissensinseln, Dokumentation aktuell; Frontend- wie Backend-/Infrastruktur-Erfahrung auf 3 Entwickler verteilt.
- **Grün - ILA:** keine Wissensinseln, Dokumentation aktuell; Frontend- wie Backend-/Infrastruktur-Erfahrung auf 3 Entwickler verteilt.

## Wissens-/Dokumentationslücken

Grün

**Erläuterung**

- Alle Komponenten (Architektur, Deployment, Betrieb) sind dokumentiert; das Team hält die Dokumentation bei Änderungen aktuell.
- Jede Komponente und jedes Tool wird von mindestens drei Entwicklern beherrscht (Wartung, Weiterentwicklung, Analyse).

## Abhängigkeit von Drittanbietern

Gelb

**Erläuterung**

- **Gelb - OpenAI/ChatGPT:** Wird umfassend für die Kursgenerierung und die Validierung von Freitexteingaben der Lernenden genutzt. Bei Wegfall, Nutzungsänderung oder Preiserhöhung fällt der Lernbetrieb nicht aus: Kurse laufen weiter, Eingaben werden dann durch Lernbegleiter bewertet, und die Kurserstellung weicht vorübergehend auf weniger leistungsfähige lokale Modelle aus.
- **Grün - TC Manager:** Liefert Kurstermine, Anmeldungen und Trainer-Zuweisungen per Schnittstelle. Bei Ausfall entfällt nur die Automatisierung; die ILP erlaubt befähigten Nutzern weiterhin die manuelle Anlage und Zuweisung. Der Lernbetrieb ist überhaupt nicht gefährdet.

## Testabdeckung / Regressionssicherheit

Grün

**Erläuterung**

- Umfassende Unit-Tests vor jedem Commit. Quality Gate für jedes Deployment.
- Automatisierte Sicherheitstests Quality Gate für jedes Deployment.
- Automatisierte End-To-End Playwright-Tests für Content Delivery und Content Factory.

## Deploy- und Rollback-Fähigkeit

Grün

**Erläuterung**

- Infrastruktur und Systeme sind dank Azure Kubernetes Service und Terraform innerhalb von 60 Minuten wiederherstellbar.
- Fehlerhafte Releases lassen sich über versionierte Container innerhalb von 30 Minuten zurückrollen.

## Technische Schulden

Grün

**Erläuterung**

- Keine bekannten Altlasten oder technischen Workarounds

## Datenschutz-/Sicherheitsrisiko im Entwicklungsalltag

Grün

**Erläuterung**

- Content Factory und Content Delivery: Auf Entwicklungs- und Testsystemen werden ausschließlich Testdaten verwendet.
- Für Testzwecke eingesetzte Sachbücher enthalten keine Lernenden- oder Personendaten.

## Skalierungs-/Lastrisiko

Grün

**Erläuterung**

- Content Factory: Infrastruktur für speicherintensive Prozesse wie die Dokumentenverarbeitung ausgelegt und erprobt.
- Content Delivery: Infrastruktur für die gleichzeitige Ausspielung und Auswertung von Kursen für 4.000 Lernende ausgelegt und erprobt; weitere Skalierung über AKS vorbereitet.
