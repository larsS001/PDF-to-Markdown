# PDF-to-Markdown

Wandelt PDF-Dateien lokal auf einem Windows-Rechner in Markdown um: per Rechtsklick auf die Datei oder vollautomatisch über einen überwachten Ordner. Ein Precheck entscheidet pro PDF, welches Konvertierungstool die beste Qualität liefert. Es verlässt keine Datei den Rechner.

## Warum

Wer LLMs (Claude, ChatGPT und andere) mit Vorlesungsskripten, Papern oder Handbüchern füttert, lädt meist PDFs hoch. Das kostet nach meiner Erfahrung deutlich mehr Tokens als derselbe Inhalt als Markdown, weil Layout und Bilder mitverarbeitet werden; bei großen Dokumenten stößt man schnell an Kontext- oder Nutzungslimits. Markdown-Text ist kompakt, behält Überschriften, Listen und Tabellen und lässt sich vom Modell direkt lesen.

Online-Konverter erledigen das, haben aber Seiten- oder Dateilimits, kosten Geld oder verlangen, dass man seine Dokumente hochlädt. Dieses Projekt macht die Umwandlung deshalb lokal, schnell und ohne Terminal: Rechtsklick auf das PDF, fertig. Oder PDF in einen Ordner legen, Markdown erscheint im Nachbarordner.

## Voraussetzungen

- Windows 11 (getestet), Windows 10 sollte funktionieren, ist aber nicht getestet. Das Kernscript `pdf2md.py` läuft auch auf anderen Systemen, die Rechtsklick- und Autostart-Integration ist Windows-spezifisch.
- Python 3 von [python.org](https://www.python.org/downloads/) (getestet mit 3.14.7). Beim Installieren „Add python.exe to PATH" ankreuzen. Die Microsoft-Store-Version von Python bitte nicht verwenden (siehe Probleme unten).
- Etwa 3 GB freier Speicherplatz für die Python-Bibliotheken plus etwa 600 MB für die KI-Modelle von Docling.
- Internetverbindung einmalig für die Einrichtung und den ersten Docling-Lauf. Danach läuft alles offline.
- Keine Adminrechte nötig; alle Einträge landen im Benutzerprofil. Einzige Ausnahme ist `langpfade_aktivieren.reg`, das nur bei einem bestimmten Fehler gebraucht wird (siehe Probleme).

## Einrichtung

1. Repository herunterladen (Code > Download ZIP, entpacken) oder klonen. Den Ordner an einen dauerhaften Ort legen, z. B. `C:\Users\<Name>\Documents\PDF-to-Markdown`. Ein Ordner in OneDrive funktioniert ebenfalls. Wird der Ordner später verschoben, müssen Schritt 4 und 5 wiederholt werden.
2. Python installieren (siehe Voraussetzungen), falls noch nicht vorhanden.
3. `setup.bat` doppelklicken. Installiert `pymupdf4llm`, `docling` und `watchdog` (2 bis 3 GB, dauert einige Minuten). Am Ende muss „Setup abgeschlossen!" stehen. Die Datei ist gefahrlos wiederholbar.
4. `kontextmenu_hinzufuegen.bat` doppelklicken. Trägt „PDF zu Markdown umwandeln" ins Rechtsklick-Menü für PDF-Dateien ein; der Pfad zum Projektordner wird automatisch ermittelt.
5. `autostart_einrichten.bat` doppelklicken. Startet die Ordnerüberwachung (unsichtbar, ohne Fenster) und richtet sie so ein, dass sie künftig mit Windows startet. Dabei entstehen im Projektordner die Ordner `Eingang`, `Ausgang` und `Verarbeitet`.
6. Optional: `klassisches_menue_aktivieren.reg` doppelklicken, dann ab- und wieder anmelden. Windows 11 zeigt Einträge von Programmen nur unter „Weitere Optionen anzeigen"; mit dem klassischen Menü steht der Eintrag direkt im Rechtsklick-Menü. Betrifft das gesamte Kontextmenü, Rückgängig mit `klassisches_menue_deaktivieren.reg`.

Der erste Docling-Lauf lädt einmalig etwa 600 MB Modelle herunter und dauert entsprechend länger.

### Probleme, die auftreten können, und ihre Lösung

**`setup.bat` bricht mit „WinError 206: Der Dateiname oder die Erweiterung ist zu lang" ab.**
Ursache: Windows begrenzt Pfade auf 260 Zeichen. Die Microsoft-Store-Version von Python installiert Bibliotheken in einen sehr tiefen Pfad, und `torch` (eine Abhängigkeit von Docling) überschreitet damit die Grenze. Zwei Lösungen:
- Empfohlen: Store-Python deinstallieren (Einstellungen > Apps > „Python 3.x" deinstallieren), Python von python.org installieren (mit „Add python.exe to PATH"), `setup.bat` erneut ausführen. Der Store-Version fehlt außerdem der `pyw`-Starter, den die unsichtbare Ordnerüberwachung nutzt.
- Alternativ: `langpfade_aktivieren.reg` doppelklicken (hebt die 260-Zeichen-Grenze systemweit auf, braucht eine Adminbestätigung), PC neu starten, `setup.bat` erneut ausführen.

**`autostart_einrichten.bat` meldet „Zugriff verweigert" und „Verknuepfung konnte nicht erstellt werden".**
Ursache: Manche Virenscanner (bei mir Avira) blockieren Scripts, die Dateien in den Autostart-Ordner schreiben. Lösung von Hand:
1. `watch_unsichtbar.vbs` im Projektordner doppelklicken. Das startet die Überwachung sofort und legt die drei Ordner an. Es öffnet sich kein Fenster.
2. Windows-Taste+R drücken, `shell:startup` eingeben, Enter. Der Autostart-Ordner öffnet sich.
3. Dort Rechtsklick > Neu > Verknüpfung. Als Speicherort eintragen (Pfad anpassen): `wscript.exe "C:\Pfad\zum\Projektordner\watch_unsichtbar.vbs"`. Als Name `PDF-zu-Markdown-Watcher` verwenden, genau so, damit `autostart_entfernen.bat` die Verknüpfung später findet.

**`kontextmenu_hinzufuegen.bat` meldet einen Fehler.**
Die Datei erzeugt dann automatisch `kontextmenu_hinzufuegen.reg` mit dem richtigen Pfad. Diese doppelklicken und die Abfragen bestätigen.

**Der Eintrag „PDF zu Markdown umwandeln" fehlt im Rechtsklick-Menü.**
Unter Windows 11 zuerst auf „Weitere Optionen anzeigen" klicken. Wer den Eintrag direkt sehen will: Schritt 6 der Einrichtung.

**Beim Docling-Lauf erscheint „RapidOCR returned empty result!" oder eine Warnung zu Symlinks (huggingface_hub).**
Beides ist harmlos. Die erste Meldung bedeutet nur, dass die Texterkennung auf einem Bild (z. B. einem Logo) nichts gefunden hat. Die Symlink-Warnung heißt, dass die Modelle etwas mehr Speicherplatz belegen, weil Windows ohne Entwicklermodus keine Symlinks erlaubt.

**Das Fenster scheint bei Docling eingefroren.**
Docling zeigt keinen Fortschritt pro Seite an. Im Task-Manager sieht man, dass `python.exe` arbeitet. Bei langen Dokumenten dauert es entsprechend (siehe Laufzeiten unten).

## Verwendung

### Weg 1: Rechtsklick auf ein PDF

Rechtsklick auf die PDF-Datei > „Weitere Optionen anzeigen" > „PDF zu Markdown umwandeln". Ein Konsolenfenster zeigt, welches Tool gewählt wurde, und schließt sich nach der Umwandlung von selbst. Die Markdown-Datei erscheint mit gleichem Namen neben dem PDF. Bei einem Fehler bleibt das Fenster mit der Meldung offen.

### Weg 2: Überwachter Ordner

PDF in den Ordner `Eingang` legen (kopieren, verschieben oder direkt dort speichern). Die Überwachung wandelt es im Hintergrund um, legt die Markdown-Datei in `Ausgang` ab und verschiebt das Original nach `Verarbeitet`. Es öffnet sich kein Fenster; das Protokoll steht in `watch_log.txt`. Mehrere PDFs auf einmal sind möglich, sie werden nacheinander verarbeitet.

Stoppen: Task-Manager > `pythonw.exe` beenden. Autostart entfernen: `autostart_entfernen.bat`. Kontextmenü entfernen: `kontextmenu_entfernen.bat`.

### Weitere Möglichkeiten

- Eine oder mehrere PDFs auf `pdf2md.bat` ziehen (Drag & Drop): Umwandlung startet sofort, die Markdown-Dateien landen neben den PDFs.
- Tool erzwingen über die Kommandozeile: `pdf2md.bat datei.pdf --tool docling` oder `--tool pymupdf4llm`; Zielordner mit `--out Ordner`.
- Existiert der Zielname schon, wird `name_2.md`, `name_3.md` usw. verwendet. Es wird nie etwas überschrieben.

## Wie das Script entscheidet

Zwei Konverter sind eingebaut: **pymupdf4llm** (schnell, gut bei normalem Fließtext) und **Docling** (KI-basierte Layoutanalyse, deutlich langsamer, dafür bessere Struktur bei Folien, Tabellen und Scans). `pdf2md.py` analysiert jedes PDF in wenigen Sekunden und prüft die Regeln in dieser Reihenfolge:

| Bedingung | Tool | Grund |
|---|---|---|
| Weniger als 80 % der Seiten enthalten Text | Docling mit OCR | Scan, Text muss erkannt werden |
| Mindestens 50 % der Seiten im Querformat | Docling | Foliensatz; siehe unten |
| Mindestens 20 % der Seiten enthalten Tabellen | Docling | beste Tabellen-Rekonstruktion |
| sonst | pymupdf4llm | Fließtext im Hochformat, schnell |

Fällt Docling aus (nicht installiert, Absturz), springt automatisch pymupdf4llm ein.

Die Folien-Regel stammt aus einem Vergleichstest mit einem 359-seitigen Vorlesungsskript (Querformat, 13 % Tabellen-Seiten). pymupdf4llm setzte fast alle Überschriften auf Ebene 6, ließ Kopf- und Fußzeile jeder Folie im Text stehen und gab Buchungssätze und T-Konten als zusammengezogene Textblöcke aus. Docling entfernte Kopf- und Fußzeilen, behielt Aufzählungen und baute Buchungssätze und T-Konten als Tabellen. Da Foliensätze zuverlässig am Querformat erkennbar sind, gehen sie zu Docling; Hochformat-Dokumente bleiben beim schnellen Weg.

Gemessene Laufzeiten für dieses Skript auf einem Laptop ohne GPU: pymupdf4llm etwa 1 Sekunde pro Seite (6:20 Minuten), Docling etwa 7,5 Sekunden pro Seite (45 Minuten). Die Schwellwerte stehen oben in `pdf2md.py` (`TEXT_ANTEIL_SCHWELLE`, `QUERFORMAT_SCHWELLE`, `TABELLEN_SEITEN_SCHWELLE`) und lassen sich anpassen; `QUERFORMAT_SCHWELLE = 0` schaltet die Folien-Regel ab.

Bekannte Grenzen: Handschrift (z. B. GoodNotes-Exporte) erkennt auch Docling nur schlecht. Diagramme und Bilder werden zu `<!-- image -->`-Platzhaltern. Docling setzt alle Überschriften auf Ebene 2.

## Dateien im Repository

| Datei | Zweck |
|---|---|
| `pdf2md.py` | Kernscript: Precheck und Konvertierung |
| `pdf2md.bat` | Wrapper für Kontextmenü, Drag & Drop und Kommandozeile |
| `watch_ordner.py` | Ordnerüberwachung (`Eingang` > `Ausgang`, Original nach `Verarbeitet`) |
| `watch_unsichtbar.vbs` | startet die Überwachung ohne Fenster |
| `setup.bat` | installiert die Python-Bibliotheken |
| `kontextmenu_hinzufuegen.bat` / `kontextmenu_entfernen.bat` | Rechtsklick-Eintrag an / aus |
| `autostart_einrichten.bat` / `autostart_entfernen.bat` | Überwachung mit Windows starten an / aus |
| `klassisches_menue_aktivieren.reg` / `klassisches_menue_deaktivieren.reg` | optional: klassisches Rechtsklick-Menü an / aus |
| `langpfade_aktivieren.reg` | nur bei WinError 206: hebt das 260-Zeichen-Pfadlimit auf |
| `LICENSE` | Lizenztext (AGPL-3.0) |

## Verwendete Tools

| Tool | Aufgabe | Quelle | Lizenz |
|---|---|---|---|
| [pymupdf4llm](https://github.com/pymupdf/pymupdf4llm) | PDF zu Markdown, schneller Weg | [Dokumentation](https://pymupdf.readthedocs.io/en/latest/pymupdf4llm/) | AGPL-3.0 (kommerzielle Lizenz bei Artifex erhältlich) |
| [PyMuPDF](https://github.com/pymupdf/PyMuPDF) | PDF-Analyse im Precheck (Text, Tabellen, Seitenformat), Basis von pymupdf4llm | [Dokumentation](https://pymupdf.readthedocs.io/) | AGPL-3.0 (kommerzielle Lizenz bei Artifex erhältlich) |
| [PyMuPDF Layout](https://pypi.org/project/pymupdf-layout/) | Layoutanalyse-Modell, das pymupdf4llm automatisch nutzt | PyPI | AGPL-3.0 (kommerzielle Lizenz bei Artifex erhältlich) |
| [Docling](https://github.com/docling-project/docling) | PDF zu Markdown mit KI-Layoutanalyse, Tabellen und OCR | [Dokumentation](https://docling-project.github.io/docling/) | MIT |
| [RapidOCR](https://github.com/RapidAI/RapidOCR) | Texterkennung innerhalb von Docling (wird von Docling in diesem Setup verwendet) | GitHub | Apache-2.0 |
| [watchdog](https://github.com/gorakhargosh/watchdog) | Ordnerüberwachung | GitHub | Apache-2.0 |
| [Python](https://www.python.org/) | Laufzeitumgebung | python.org | PSF-Lizenz |

Docling lädt beim ersten Lauf seine Modelle von Hugging Face (u. a. `docling-project/docling-models` und `docling-project/docling-layout-heron`).

## Lizenz

Copyright (C) 2026 Lars Simon

Dieses Projekt steht unter der GNU Affero General Public License, Version 3 (AGPL-3.0). Der vollständige Lizenztext liegt in der Datei [`LICENSE`](LICENSE). Kurz gefasst: Jeder darf das Tool nutzen, verändern und weitergeben, auch kommerziell; wer eine veränderte Version verteilt oder als Dienst anbietet, muss den Quellcode ebenfalls unter AGPL-3.0 offenlegen. Die Lizenz wurde gewählt, weil die verwendeten Bibliotheken PyMuPDF und pymupdf4llm selbst unter AGPL-3.0 stehen. Es gibt keine Gewährleistung.

Mit dem Tool erzeugte Markdown-Dateien sind von der Lizenz nicht betroffen; sie gehören dem, der das PDF hat.

## Video

Ein kurzes Screenrecording, das beide Wege zeigt (Rechtsklick auf ein PDF und Ablegen im Eingang-Ordner), folgt in Kürze.
