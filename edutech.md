##Kontext
Wir setzen eine Lernplattform um. Diese wird in der
Erwachsenenfortbildung eingesetzt, um Weiterbildungen und Umschulungen als Selbstlernkurse zu generieren.

##Komponenten
* Content Factory: Generierung von Kursen durch Digital Learning Designer. Nutzung von Quelldokumenten (1-3 werden als PDF hochgeladen). Nach Generierung, Anpassung der generierten Kurse.
* Content Delivery: Ausspielung von Kursen aus der Content Factory für Lernende
* Lerndatenmessung: Messung von Lernfortschritt. Profiling von Lernenden. Realtime-Bewertung, ob Lernende langsam, normal oder schnell lernen und ob sie Unterstützung durch Lernbegleiter benötigen.
* Intelligent Learning Assistant: Chatbot der Lernende bei Fragen und Problemen in einem sokratischen Dialog unterstützt. 

##Kurs
* Wird in der Content Factory generiert.
* Findet an mehreren Terminen statt
* Jeder Kurs hat 1-3 Lernbegleiter (meistens 1) und 5-15 Lernende
* Dauer Kurstermin 3 Wochen bis 3 Monate

##Kurskategorien
* Programmierkurs
** Programmiersprachen wie Java, Python, JavaScript, aber auch HTML und CSS
** Dauer: 3 Wochen bis 3 Monate
* Methodikkurs
** Projektmanagementmethoden wie Scrum, Kanban, Prince2
** Dauer: 3 Wochen bis 3 Monate
* Tooltraining
** Office-Programme, Microsoft Server, Betriebssysteme
** Dauer: 3 Wochen bis 3 Monate

##Nutzer (getrennte Rollen)
* Digital Learning Designer (DLD)
** Läst Kurse generieren und bearbeitet sie
* Subject Matter Experts (SME)
** Bearbeitet Storyboards
* Lernende
** Konsumieren Kurse, immer nur einen zur Zeit
* Lernbegleiter
** Unterstützen Lernende
** Gibt Kurse der DLDs frei

##Technologien
* Content Factory
** Frontend: React
** Datenbank: Postgres
** Backend: Ruby on Rails
** Prozesse: Celery
** AI-Backend: Python
** Generierung: ChatGPT
* Content Delivery
** Frontend: React
** Datenbank: Postgres
** Backend: Ruby on Rails
** Prozesse: Celery
* Lerndatenmessung
** Technologie: noch offen

##Content Factory
* Phasen
** Grobkonzept
*** Generierung einer Kursgliederung bestehend aus Modulen, Kapiteln und Unterkapiteln
*** Parameter: 1-3 Quelldokumente, Kurskategorie, geplante Kursdauer
*** Trainer gibt Grobkonzept frei, bevor daraus das Feinkonzept generiert wird
** Feinkonzept
*** Generierung von Content Elementen (Titel, Typ, grober Inhalt) in den Unterkapiteln 
*** Ein Unterkapitel hat in der Regel 10-15 Content Elemente
*** DLD gibt Feinkonzept frei, bevor es zu Storyboards wird
** Storyboards
*** Generierung von Text, Quiz, Bild oder Video
*** SME bearbeitet Storyboards (Anpassung von generierten Storyboards und neu schreiben)
*** Lernbegleiter gibt Storyboards frei

##Content Delivery
* Komponenten
** Lernenden Dashboard
** Lernbegleiter Dashboard
** Kurs
*** Kursstruktur (Modul, Kapitel, Unterkapitel)
*** Kursinhalt (Content Elemente in Reihenfolge)
*** Lernende navigieren frei durch Reihenfolge Content Elemente
** Gamification
*** Punkte für abgeschlossene Quizze
*** Badges für erreichte Meilensteine
*** Leaderboard für Kurstermine im Dashboard für Lernende

##Lerndatenmessung
* Events aus der Content Delivery tracken
* Statistiken für Kurstermine, Lernende und Lernbegleiter aufbereiten
** Nutzung im Dashboard des Lernbegleiters
** Statistiken für DLDs außerhalb Lernbetrieb
** historische Daten über 36 Monate
* Near-Realtime
** Lernfortschritt von Teilnehmern ermitteln
** Probleme von Teilnehmern für Lernbegleiter ermitteln
*** Lernender über Zeitraum X langsamer als Baseline für Lerndauer
*** Lernender hat in Zeitraum X einen Prozentsatz an Quizzen falsch beantwortet
*** Lerndener zeigt seit Zeitraum X keine Aktivität

##Intelligent Learning Assistant
* Chatbot der Lernende bei Fragen und Problemen in einem sokratischen Dialog unterstützt
* Keine automatische Antwort auf Fragen
* Aufzeigen von Lernstrategien
* Referenz auf Content Elemente in Kursstruktur 

##Aufbau von Kursen
* Kurse sind in Module, Kapitel und Unterkapitel unterteilt.
* Jedes Unterkapitel verfügt über eine Anzahl von Content Elementen.
* Ein Content Element ist eine didaktische Methode für einen bestimmten Einsatzzweck. 
* Ein Content Element besteht aus Text, Quiz, Bild oder Video. Der eigentliche Inhalt an einem Content Element ist ein Storyboard.
* 1 Modul dauert ungefähr 1 Woche
* 1 Kapitel dauert ungefähr 1 Tag
* 1 Unterkapitel dauert ungefähr 1 Stunde