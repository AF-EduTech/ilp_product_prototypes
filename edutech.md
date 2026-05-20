## Kontext
Wir setzen eine Lernplattform um. Diese wird in der
Erwachsenenfortbildung eingesetzt, um Weiterbildungen und Umschulungen als Selbstlernkurse zu generieren.

## Komponenten
- Content Factory: Generierung von Kursen durch Digital Learning Designer. Nutzung von Quelldokumenten (1-3 werden als PDF hochgeladen). Nach Generierung, Anpassung der generierten Kurse.
- Content Delivery: Ausspielung von Kursen aus der Content Factory für Lernende
- Lerndatenmessung: Messung von Lernfortschritt. Profiling von Lernenden. Realtime-Bewertung, ob Lernende langsam, normal oder schnell lernen und ob sie Unterstützung durch Lernbegleiter benötigen.
- Intelligent Learning Assistant: Chatbot der Lernende bei Fragen und Problemen in einem sokratischen Dialog unterstützt. 
- Learning Management: Synchronisiert Kurstermine, Lernende und Lernbegleiter aus dem Geschäftsbetrieb (Verwaltung von allen Kursen, Kursterminen, Lernenden und Lernbegleitern)

## Weiterbildungskurs
- Wird in der Content Factory generiert.
- Jeder Kurs findet an einem Kurstermin statt
- Jeder Kurs hat 1-3 Lernbegleiter (meistens 1) und 5-15 Lernende
- Dauer Kurstermin 3 Wochen bis 3 Monate
- Es gibt Weiterbildungskurse und Umschulungskurse
- Weiterbildungkurse sind nach einem Termin abgeschlossen. Umschulungskurse können über mehrere Termine laufen.

## Umschulungskurse
- Eine Umschulung wird Lernenden angeboten, die ihren Beruf nicht mehr ausüben können oder wollen.
- Es gibt mehrere Umschulungen, vor allem aber zum Fachinformatiker
- Eine Umschulung dauert 2-3 Jahre
- Eine Umschulung besteht aus 15-20 Lernfeldern, die über einen Umschulungsplan zusammengehalten werden
- Ein Lernfeld ist ein spezieller Kurs, z.B. zu einer Methodik oder einer Programmiersprache und hat eine Laufzeit wie ein normaler Kurs
- Über die gesamte Umschulung hinweg gibt es mehrere Lernbegleiter, die die Lernenden unterstützen

## Kurskategorien
- Programmierkurs
    - Programmiersprachen wie Java, Python, JavaScript, aber auch HTML und CSS
    - Dauer: 3 Wochen bis 3 Monate
- Methodikkurs
    - Projektmanagementmethoden wie Scrum, Kanban, Prince2
    - Dauer: 3 Wochen bis 3 Monate
- Tooltraining
    - Office-Programme, Microsoft Server, Betriebssysteme
    - Dauer: 3 Wochen bis 3 Monate

##  Aufbau von Kursen
- Kurse sind in Module, Kapitel und Unterkapitel unterteilt.
- Jedes Unterkapitel verfügt über eine Anzahl von Content Elementen.
- Ein Content Element ist eine didaktische Methode für einen bestimmten Einsatzzweck. 
- Ein Content Element besteht aus Text, Quiz, Bild oder Video. Der eigentliche Inhalt an einem Content Element ist ein Storyboard.
- 1 Modul dauert ungefähr 1 Woche
- 1 Kapitel dauert ungefähr 1 Tag
- 1 Unterkapitel dauert ungefähr 1 Stunde

## Nutzer (getrennte Rollen, unterschiedliche Personen)
- Digital Learning Designer (DLD)
    - Lässt Kurse generieren und bearbeitet sie
    - Arbeitet eng mit dem SME zusammen
    - Plant zusammen mit SME und Lernbegleiter die Kursstruktur
- Subject Matter Experts (SME)
    - Erstellt und bearbeitet Storyboards
    - Gibt dem DLD in enger Zusammenarbeit Feedback zu Storyboards und Kursstruktur
- Lernende
    - Konsumieren Kurse, immer nur einen zur Zeit
- Lernbegleiter
    - Unterstützen Lernende
    - Gibt Grobkonzepte der DLDs frei
    - Gibt Storyboards der DLDs frei

## Technologien
- Content Factory
    - Frontend: React
    - Datenbank: Postgres
    - Backend: Ruby on Rails
    - Prozesse: Celery
    - AI-Backend: Python
    - Generierung: ChatGPT
- Content Delivery
    - Frontend: React
    - Datenbank: Postgres
    - Backend: Ruby on Rails
    - Prozesse: Celery
- Lerndatenmessung
    - Technologie: noch offen

## Content Factory Phasen
- Grobkonzept
    - Generierung einer Kursgliederung bestehend aus Modulen, Kapiteln und Unterkapiteln
    - Parameter: 1-3 Quelldokumente, Kurskategorie, geplante Kursdauer
    - Lernbegleiter gibt Grobkonzept frei, bevor daraus das Feinkonzept generiert wird
- Feinkonzept
    - Generierung von Content Elementen (Titel, Typ, grober Inhalt) in den Unterkapiteln 
    - Ein Unterkapitel hat in der Regel 10-15 Content Elemente
    - SME und DLD geben Feinkonzept frei, bevor es zu Storyboards wird
- Storyboards
    - Generierung von Text, Quiz, Bild oder Video
    - SME bearbeitet Storyboards (Anpassung von generierten Storyboards und neu schreiben)
    - Lernbegleiter gibt Storyboards frei

## Content Delivery Komponenten
- Lernenden Dashboard
    - Startseite für Lernende, auch nach einer Pause oder am nächsten Tag
    - Einstieg in den Kurs
    - Wiederaufnahme nach Pause
    - Übersicht über den eigenen Fortschritt
    - Übersicht zur Gamification (Punkte, Badges, Leaderboard)
    - Zugang zum Lernbegleiter für Hilfe und Unterstützung
    - Kursplan (Tagesplan)
- Lernbegleiter Dashboard
    - Übersicht über den aktuellen Kurs
    - Übersicht über die Lernenden
    - Übersicht über den Fortschritt der Lernenden
    - Übersicht über die Probleme der Lernenden
    - Übersicht über die Fragen der Lernenden
    - Übersicht über die Unterstützung der Lernenden
- Kurs
    - Kursstruktur (Modul, Kapitel, Unterkapitel)
    - Kursinhalt (Content Elemente in Reihenfolge)
    - Lernende navigieren frei durch Reihenfolge Content Elemente
    - Alle Content Elemente eines Unterkapitels werden auf einer Seite angezeigt
    - Ein Unterkapitel ist in die didaktischen Sequenzschritte unterteilt: Vorher, Während, Nachher, Schluss
- Gamification
    - Punkte für abgeschlossene Quizze
    - Badges für erreichte Meilensteine
    - Leaderboard für Kurstermine im Dashboard für Lernende

## Lerndatenmessung
- Events aus der Content Delivery tracken
- Statistiken für Kurstermine, Lernende und Lernbegleiter aufbereiten
- Nutzung im Dashboard des Lernbegleiters
- Statistiken für DLDs außerhalb Lernbetrieb
- historische Daten über 36 Monate
- Near-Realtime
- Lernfortschritt von Teilnehmern ermitteln
- Probleme von Teilnehmern für Lernbegleiter ermitteln
    - Lernender über Zeitraum X langsamer als Baseline für Lerndauer
    - Lernender hat in Zeitraum X einen Prozentsatz an Quizzen falsch beantwortet
    - Lerndener zeigt seit Zeitraum X keine Aktivität

## Learning Management
- Synchronisiert Kurstermine, Lernende und Lernbegleiter aus dem Geschäftsbetrieb (Verwaltung von allen Kursen, Kursterminen, Lernenden und Lernbegleitern)
- Die Daten kommen aus einem bestehenden ERP System. Dort werden die Daten gepflegt
- Im Learning Management werden die Daten aus dem ERP System in die Content Factory und die Lerndatenmessung übergeben, werden aber nicht bearbeitet

##  Intelligent Learning Assistant
- Chatbot der Lernende bei Fragen und Problemen in einem sokratischen Dialog unterstützt
- Keine direkte Antworten auf Fragen, sondern sokratische Fragen
- Aufzeigen von Lernstrategien
- Referenz auf Content Elemente in Kursstruktur 
- Aktionen und Inhalte des Chatbots sind nicht für Lernbegleiter sichtbar

