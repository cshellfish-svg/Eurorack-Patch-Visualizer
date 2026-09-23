# SESSION HANDOVER — Eurorack Patch-Dokumentations-Tool

## 1. Anweisung an das Modell (bei Sitzungsstart zuerst lesen)

Der folgende Block ist der vollständige Kontext eines laufenden Projekts. Zugehörig zu live-small-rack_default.xml (mit Kommentaren in Head) und patch-tool.html. Lies **Abschnitt 2–5 vollständig**, bevor du irgendetwas tust. Fasse den Kontext NICHT von dir aus zusammen, stelle KEINE Rückfrage im Sinne von „Wie soll ich fortfahren?" — warte auf konkrete nächste Anweisung und knüpfe dann direkt an den in Abschnitt 3 gelisteten offenen Punkten an. Antworten grundsätzlich auf Deutsch.

---

## 2. Projekt-Übersicht & Tech-Stack

**Ziel:** Ein Web-Tool zur Dokumentation von Daniels Eurorack-Patch-Case "LIVE (small)" (ModularGrid: `modulargrid.net/e/modules_racks/data_sheet/3033116`, User "Shellfish"), als Alternative zur unübersichtlichen Kabel-Darstellung von modulargrid.net bei ~42 Modulen.

**Kernidee:** Module werden nach *Funktion* gruppiert (nicht physische Case-Position) in Reihen dargestellt. Verbindungen (Patch-Kabel) werden nach *Signal-Layer* (Audio/Pitch/Gate/Modulation/Clock) eingefärbt und lassen sich pro Layer ein-/ausblenden. Zusätzlich lässt sich eine "Stimme" (Voice, = ein Oszillator-Modul als Startpunkt) isolieren — die Zugehörigkeit anderer Module zu dieser Stimme wird zur Laufzeit per Graph-Traversal über die gesetzten Verbindungen berechnet, nicht statisch gespeichert.

**Tech-Stack:**
- Ein einzelnes, self-contained **HTML/CSS/Vanilla-JS**-File (`patch-tool.html`).
- Datenhaltung extern in einer **XML-Konfigurationsdatei** (`live-small-rack_default.xml`), die im Tool per `<input type="file">` geladen wird (kein Hardcoding von Daten im Tool).
- Verbindungen (`<connections>`) werden **im Tool selbst** interaktiv gesetzt (Drag&Drop oder Klick-Klick) und beim Export wieder mit in die XML zurückgeschrieben.
- Keine Frameworks, keine externen JS-Libraries — nur Google Fonts (Inter, JetBrains Mono) und natives DOM/SVG.

---

## 3. Aktueller Status

### ? Funktioniert (im Code aus Abschnitt 5 enthalten)
- XML-Import per Datei-Dialog, Parsing von `layers`, `voices`, `functions`, `modules`+`ports`, `connections`.
- XML-seitig: `<functions>` hat  eine `row`-Spalte, bei der sich mehrere `function`-Werte eine Reihe teilen können
- Rendering der Module als Boxen mit Ports (Dots links=in, rechts=out, Raute=bidirectional).
- Verbindungen setzen: Drag&Drop von Port zu Port, sowie Klick-Klick als Alternative.
- Verbindungen lösen: Doppelklick/Rechtsklick auf Kabel (löscht das Kabel) oder auf einen Port (löscht dessen letzte Verbindung).
- Stack-Cables: ein Port kann mehrere Kabel tragen.
- Kabelfarbe = **immer** die Layer-Farbe des tatsächlichen Signalgebers (Source), unabhängig von der Klick-Reihenfolge — gelöst über `resolveConnection()`, die anhand `direction` (`in`/`out`/`bidirectional`) die Quelle bestimmt statt anhand von "wer wurde zuerst angeklickt".
- Layer-Toggles (Sidebar) zum Ein-/Ausblenden von Kabeln nach Signaltyp.
- Stimmen-Filter (Dropdown) mit Graph-Traversal (`reachableFromVoice`), dimmt alles außerhalb der erreichbaren Module/Kabel.
- Export als XML mit frei wählbarem Dateinamen über die `downloads`-Runtime-Capability.
- Selbst-Verbindungen (gleiches Modul, zwei Ports) sind erlaubt.
- Neue Funktionsgruppe "EXT"** (externe Synths) — im XML-Schema als `<function id="ext" row="7" label="EXT"/>` ergänzt, aktuell aber ohne zugeordnete Module.
- Optionales Port-Attribut `elementary="false"`, wird ggf. erst später verwendet

### ? NICHT fertig — offene Baustellen
1. ** bei Status der Ports ('default_active="false"'): Beim Laden vom XML soll die Port Liste gerendert werden (Inaktiv=blau, Aktiv= Grün) und auch die Ports im Modul (entsprechend der tatsächlichen Funktion: Inaktiv=Nicht anzeigen, Aktiv=Anzeigen) **
2. ** Das Rot in den Port Listen bitte ersetzen durch dasselbe Grün (der Default-Active-Ports) **
3. ** Sobald versteckte Ports vorhanden sind soll der [+] Button farblich markiert werden (mit demelben Blau wie in den Port Listen verwendet bei Status inaktiv) **
4. ** Analog zum Stimmen-Filter sollen auch bei Klick auf den Modul Header alle anderen, nicht verbundenen Module gedimmt werden **
5. ** Graph-Traversal bei den Connections (Kabeln): gerade Linien anstatt gebogene **
6. ** XML Export (z.Z. ist nur eine Dummy Funktion vorhanden) **

### Sonstige bekannte Unsicherheiten in den Moduldaten (informativ, nicht blockierend)
Die ca. 40 Module wurden entweder aus Original-Manuals recherchiert oder selbst anhand von realen Module korrigiert/bestätigt.

### Nächster sinnvoller Schritt
1. Tool-Code umbauen für Punkte 1–5 oben (offene Baustellen).

---

## 4. XML-Datenschema (repräsentatives Beispiel, nicht der volle Modul-Datensatz)
** Siehe live-small-rack_default.xml - die enthaltenen Kommentare in Head beachten **

---
                     
## 5. Quellcode des Prototyps 
** Siehe patch-tool.html **
