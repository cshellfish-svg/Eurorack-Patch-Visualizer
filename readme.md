# SESSION HANDOVER — Eurorack Patch-Dokumentations-Tool

## 1. Anweisung an das Modell (bei Sitzungsstart zuerst lesen)

Der folgende Block ist der vollständige Kontext eines laufenden Projekts. Zugehörig zu `live-small-rack_default.xml` und `patch-tool.html` (bzw. der aktuellen erweiterten Variante in der Entwicklung). Lies Abschnitt 2–6 vollständig, bevor du weitere Änderungen am Tool vornimmst. Die nachfolgenden Punkte sind die zentrale Arbeitsbasis für die nächste Sitzung.

---

## 2. Projekt-Übersicht & Tech-Stack

**Ziel:**
Ein Web-Tool zur Dokumentation des Eurorack-Patch-Cases "LIVE (small)" von Daniel, als Alternative zur unstrukturierten Notiz- bzw. PDF-Doku. Das Tool soll Module semantisch nach Funktion gruppieren, Verbindungen visuell als Signal-Layer darstellen und Export/Weiterverarbeitung über XML erlauben.

**Kernidee:**
Module werden nach Funktion gruppiert und in Reihen dargestellt. Verbindungen werden nach Signal-Layer klassifiziert (z. B. Audio, Pitch, Gate, Modulation, Clock). Sichtbarkeits- und Fokuslogiken sollen es ermöglichen, einzelne Stimmen oder Layer isoliert zu betrachten, ohne die Gesamtstruktur zu verlieren.

**Tech-Stack:**
- Einzelnes, self-contained HTML/CSS/Vanilla-JS-File (`patch-tool.html`)
- Datenhaltung extern in einer XML-Datei (`live-small-rack_default.xml`)
- Im Tool interaktiv gesetzte Verbindungen werden beim Export wieder in XML geschrieben
- Keine Frameworks, keine Bibliotheken; nur natives DOM/SVG und Google Fonts

---

## 3. Aktueller Status

### Erfolgreich implementiert
- XML-Import per Datei-Dialog mit Parsing von `layers`, `voices`, `functions`, `modules`, `ports` und `connections`
- Funktionsbasierte Reihen-/Gruppenansicht mit Modulen als Boxen
- Ports als visuelle Dots: `in` links, `out` rechts, `bidirectional` als Raute
- Verbindungserstellung per Drag&Drop sowie Klick-Klick als Alternative
- Verbindungslöschung per Doppelklick/Rechtsklick auf Port oder Kabel
- Multi-Connection pro Port (Stack-Cables)
- Layer-Toggles in der Sidebar zum Ein-/Ausblenden von Signalpfaden
- Stimmen-Filter mit Graph-Traversal, der nicht erreichbare Module/Kabel dimmt
- Modul-Header-Interaktion mit Fokus-/Dim-Logik für Nachbarschaften
- Popover für Port-Sichtbarkeit und Einzel-Status der Ports
- Export als XML mit frei wählbarem Dateinamen
- Rack-Ansicht mit optimierter Reihenfolge/Verteilung der Module (ohne Labels, vertikales Layout)
- Rack-Optimierung mit Bewertungsfunktion („connections“, „functions“, „compactness“)
- Rack-spezifischer XML-Export inklusive `rack_rows`, `rack_hp`, `rack_row` und `rack_hp_start/end`
- Dynamische Rack-Parameter (`rack_rows`, `rack_hp`) im UI und beim XML-Import/Export
- Selbst-Verbindungen und Port-Status-Handling sind im Standardpfad unterstützt

### Noch offen / nächste Baustellen
1. Konsistenz zwischen sichtbaren/invisiblen Ports und Popover-Status weiter prüfen und ggf. vereinfachen
2. Finaler visueller Feinschliff der Port-Farben und der `+`-Markierung bei versteckten Ports
3. Optionales `elementary`-Handling im XML-Schema klar definieren, falls später erforderlich
4. Kabeldarstellung weiter verfeinern (z. B. gerade vs. gebogene Linien je nach Ansicht/Typ)
5. Exportroutine und Dateinamenlogik abschließend gegen Edge Cases abtesten
6. Zustandslogik beim Rack Export (Export trägt nur dann den Namen _optimized_N.xml, wenn tatsächlich eine Optimierung ausgeführt wurde)
7. Popover-Layout in Rack-Ansicht verbreitern
8. Modul Konfiguration im Tool ergänzen (XML lesen/schreiben)
9. Daten API / Import für Modul Metadaten ergänzen

### Sonstige bekannte Unsicherheiten in den Moduldaten
Die Module wurden größtenteils aus Original-Manuals, realen Referenzen oder manueller Korrektur übernommen. Die Datensätze sind für die UI-Logik ausreichend, aber nicht als vollständige Masterdatenbank für alle Eurorack-Module gedacht.

### Nächster sinnvoller Schritt
1. Zustandslogik beim Rack Export (nächste Baustellen: Punkt 6)
2. Popover-Layout in Rack-Ansicht verbreitern (nächste Baustellen: Punkt 7)
3. UI/UX und Render-Logik auf Konsistenz validieren
4. Rack- und XML-Workflow mit echten Beispieldateien durchlaufen
5. Eventuelle Redundanzen in der App-Struktur bereinigen und finalisieren

---

## 4. Architekturentscheidungen & Struktur

### Zielarchitektur
Das Tool ist bewusst als single-file Web-App mit Vanilla JS gebaut. Das hat drei Vorteile:
- Sehr geringe Einstiegshürde
- Keine Build- oder Paket-Abhängigkeiten
- Direkte Daten-/UI-Kopplung für schnelle Iteration im Browser

### Datenmodell
Das zentrale Domänenmodell basiert auf XML, nicht auf einer internen JSON-DB. Die XML-Datei liefert:
- `layers`
- `voices`
- `functions`
- `modules`
- `ports`
- `connections`

Das Tool liest die XML-Struktur beim Laden, wandelt sie in ein in-memory-Objekt um und rendert daraus die UI.

### App-Struktur (heutiger Stand)
Die Logik ist bewusst in funktionale Blöcke gegliedert:
- `parse()` – liest XML und erzeugt den internen State
- `render()` – baut die Funktions- oder Rack-Ansicht auf
- `makeModuleBox()` – erzeugt eine Modul-Box inklusive Ports und Popover
- `draw()` – zeichnet die SVG-Kabel auf Basis aktueller Port-Positionen
- `resolve()` – entscheidet Richtung und Farbe der Verbindung
- `exportXML()` – schreibt den aktuellen State zurück in XML
- `optimizeRack()` – erzeugt eine Rack-Positionierung nach Bewertungsfunktion

### Sichtbarkeits- und Fokuslogik
Das Tool arbeitet mit drei Ebenen:
1. Layer-Toggles: Ein-/Ausblenden von Signaltypen
2. Module/Voice-Filter: Fokus aus einer ausgewählten Stimme
3. Per-Modul-Status: Sichtbarkeit einzelner Ports durch `default_active`

Diese Schichten werden bewusst getrennt modelliert, damit Filterung und Visualisierung nicht in dieselbe Logik laufen.

### Warum XML als Arbeitsformat
- Einfaches Austauschformat mit externer Doku und manueller Pflege
- Keine Server- oder DB-Notwendigkeit
- Gut geeignet für semantische Projektdateien mit viele Metadaten pro Modul und Port

### Warum Render als DOM + SVG statt Canvas
- DOM ist für die Boxen, Labels und Port-Interaktion besser geeignet
- SVG ist für Kabel und Linien ideal, da es Interaktion und Styling leicht erlaubt
- Dadurch bleibt die App visuell leicht anpassbar und debugbar

### Designprinzipien
- UI-Logik und Datenlogik bewusst getrennt halten
- Keine komplexen Zustandsbibliotheken, weil die App klein bleibt
- Interaktionen über native Events, weil sie für diese Datei direkt und stabil sind

---

## 5. XML-Datenschema (repräsentatives Beispiel, nicht der volle Modul-Datensatz)

Das XML-Schema ist die Grundlage der gesamten App. Relevante Bestandteile sind:

```xml
<patch title="LIVE (small)" hp="265">
  <layers>
    <layer id="audio" label="Audio" color="#6fcf97" />
    <layer id="gate" label="Gate" color="#4f9fe0" />
  </layers>

  <voices>
    <voice id="v1" label="Voice 1" />
  </voices>

  <functions>
    <!-- row = feste Position im Diagramm-Layout; mehrere function-Ids
         können sich eine row + label teilen (z.B. fx+process -> "Processing",
         env+mod -> "Modulation"). function wird pro Modul referenziert,
         die Reihe selbst ergibt sich daraus (nicht separat am Modul gespeichert). -->
    <function id="osc" row="1" label="OSC" />
    <function id="filter" row="2" label="FILTER" />
  </functions>

  <modules>
    <module id="m1" name="VCO" function="osc" active="true" hp="6" collapsed="false" rack_hp_start="0" rack_hp_end="6">
      <ports>
        <port id="m1.out1" direction="out" layer="audio" default_active="true" />
        <port id="m1.in1" direction="in" layer="audio" default_active="true" />
      </ports>
    </module>
  </modules>

  <connections>
    <connection from="m1.out1" to="m2.in1" />
  </connections>
</patch>
```

Wichtige Punkte:
- `function`-Einträge können mehrere Module derselben Gruppe zuordnen
- `row` definiert die visuelle Reihenfolge der Funktionsgruppen
- `default_active` ist der Standard-Sichtbarkeitsstatus eines Ports
- `connections` werden nach dem Laden des Tools interaktiv ergänzt oder gelöst
- `rack_rows` und `rack_hp` können als Attribute auf Root-Ebene gesetzt werden, um Rack-Layouts zu definieren
- `collapsed` wird dynamisch beim Speichern gesetzt und gibt den letzten Aus- Eingeklappt Status eines Moduls wieder
- `rack_hp_start` und `rack_hp_end` werden nach dem Rack optimieren über "Rack-XML speichern" dynamisch gesetzt

---

## 6. Quellcode des Prototyps / Referenz

**Siehe `patch-tool.html`** und die jeweils aktuelle Entwicklungsvariante im Repository.

Der Prototyp dient als Referenz für die Ausgangsbasis; die aktuelle Entwicklung erweitert die UI-Logik, Rack-Ansicht und Export-Funktionalität weiter.

---

## 7. Kurzfazit für die nächste Sitzung

Das Projekt ist von einem reinen Prototyp zu einer funktionierenden, strukturierten Doku-/Patch-App gewachsen. Die zentrale Stärke liegt in der Kombination aus XML-basiertem Modell, semantischer Funktions- und Layer-Visualisierung sowie interaktivem Export. Die nächste Aufgabe sollte nicht mehr im Bereich „Grundlagen bauen“, sondern in der Konsolidierung, Feinabstimmung und Stabilisierung der vorhandenen Logik bestehen.

