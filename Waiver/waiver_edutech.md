# Beurteilung Azure Kubernetes Service statt Azure Container Apps

## Hintergrund
In der ILP setzen wir für den Betrieb von Content Factory und Content Delivery ein. Insbesondere die Analyse und Verarbeitung von Quelldokumenten ist ein sehr aufwändiger Prozess und erfordert viel RAM.

## Fragestellung

Ist es sinnvoll, von AKS auf ACA umzusteigen? 

## Analyse aus dem Entwicklungsteam

Ein Umstieg von AKS auf Azure Container Apps (ACA) bietet aktuell keinen ausreichenden technischen oder wirtschaftlichen Mehrwert, da die bestehende AKS-Lösung alle erforderlichen Anforderungen bereits zuverlässig erfüllt. Insbesondere Autoscaling sowie individuelle Anforderungen an CPU, Memory, GPU und Storage können in AKS flexibel und servicespezifisch konfiguriert und bei Bedarf erweitert werden. ACA verfolgt ein anderes Ressourcen- und Skalierungsmodell und führt dadurch insbesondere bei spezialisierten Workloads zu zusätzlichen Einschränkungen und technischen Abhängigkeiten. Der Wechsel würde daher erheblichen Migrations- und Testaufwand verursachen, ohne einen entsprechenden funktionalen Vorteil gegenüber der bestehenden Plattform zu schaffen.

## Aufwand für eine Migration AKS → ACA
Last- und Skalierungstests: 5 PD
Deployment auf ACA umstellen: 3 PD
GHCR-Anbindung
Deployment-Skripte
Secrets und Konfiguration
Stabilisierung und Sicherstellung des zuverlässigen Betriebs aller Services: 5 PD
Queuing und Autoscaling über KEDA/Flex zuverlässig aufsetzen und testen: 10 PD
Rechte-/Berechtigungsmanagement sowie Schulung/Dokumentation: 2 PD
Gesamtaufwand: ca. 25 PD


## Beispiele für bereits erstellte Dateien

2026-09-01 Waiver-Antrag 1 GitHub (ILP) - Waiver für die Nutzung von GitHub im ILP-Team
2026-09-01 Waiver-Antrag 2 DragonflyDB (ILP) - Waiver für die Nutzung von DragonfylDB im ILP-Team
2026-09-01 Vorstandsvorlage Waiver ILP - Vorstandsvorlage zur Entscheidung über alle Waiver
2026-09-01 Anlage 3 Betriebsuebernahme-Erklaerung AF EduTech (ILP) - Beschreibung des Betriebs durch die edutech, in Hinblick auf die beiden Waiver.



