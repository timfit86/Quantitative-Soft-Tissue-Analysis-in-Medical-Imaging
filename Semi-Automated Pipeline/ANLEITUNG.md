# KSW Semi-Automatic Body Composition – Kurzanleitung (nur 3D Slicer, CSV-Export)

> **Forschungsprototyp.** Ersetzt keine radiologische Beurteilung oder klinische Entscheidung.

Diese Variante läuft **vollständig in 3D Slicer**. Es wird **kein Python**, **keine Zusatzpakete**, **kein Dashboard** und **keine Slicer-Erweiterung** benötigt. Die Resultate werden als CSV-Datei gespeichert. Geeignet für Laptops mit strengen Sicherheitseinstellungen.

---

## 1. Installation (einmalig)

1. **3D Slicer** (aktuelle stabile Version) von <https://download.slicer.org> installieren. Bei gesperrten Geräten über die IT installieren lassen.
2. Aus diesem Repository nur **eine Datei** herunterladen:
   [`KSW_SemiAutomatic_BCW_module.py`](KSW_SemiAutomatic_BCW_module.py) → auf der Dateiseite oben rechts **Download raw file**.
3. Die Datei in einen eigenen Ordner legen, z. B. `Dokumente\KSW_Slicer_Modul\`.
   Den Dateinamen **nicht ändern**.
4. 3D Slicer öffnen → **Edit → Application Settings → Modules**.
5. Bei **Additional module paths** auf **Add** klicken und den Ordner aus Schritt 3 wählen → **OK**.
6. Slicer neu starten.
7. Im Modulmenü oben links nach **KSW Semi-automatic MRI Body Composition Workflow** suchen und öffnen.

---

## 2. Ablauf pro Patient

| Schritt | Was tun | Button im Modul |
|---|---|---|
| 1 | Dixon-**Fettbild** laden (Datei oder DICOM-Ordner) | **Load Fat Image** |
| 2 | Dixon-**Wasserbild** laden | **Load Water Image** |
| 3 | Zuordnung prüfen: *Fat image* = Fett, *Water image* = Wasser | – |
| 4 | Patienten-ID, ggf. Jahrgang und **Körpergrösse in m** (z. B. `1.70`) eingeben – Grösse ist Pflicht | – |
| 5 | Modus wählen: **2D image mode** (ein Schnitt) oder **3D single-slice mode** (Volumen) | *Processing mode* |
| 6 | Nur 3D: im roten Viewer zur **Mitte L3** scrollen, dann speichern | **Auto-detect** |
| 7 | SAT erstellen und korrigieren | **1. Run SAT threshold** → **2. Edit SAT Segment** |
| 8 | VAT erstellen (SAT wird abgezogen) und korrigieren | **3. Run VAT threshold + subtract SAT** |
| 9 | Muskel erstellen und auf dem Wasserbild korrigieren | **4. Run Muscle threshold …** → **Edit Muscle Segment** |
| 10 | IMAT berechnen (Start: Min `53`, Max `233`) | **Run automatic IMAT segmentation** |
| 11 | Ausgabeordner prüfen, Metriken berechnen und speichern | **Compute Metrics + Export CSV** |
| 12 | Nächster Patient | **Start New Patient** |

### Korrigieren im Segment Editor
- Zuerst das Segment anklicken (**SAT**, **VAT**, **Muscle** oder **IMAT**).
- **Paint** = hinzufügen, **Erase** = entfernen. Pinselgrösse nach Bedarf.
- *Source volume*: Fettbild für SAT/VAT/IMAT, Wasserbild für Muskel.
- **SAT**: nur subkutanes Fett ausserhalb der Bauchwand.
- **VAT**: nur Fett innerhalb der Bauchhöhle.
- **Muskel (L3)**: Psoas, paraspinale Muskulatur, Bauchwandmuskulatur. Kein Knochen, keine Organe, kein Darm.
- **IMAT**: liegt nur innerhalb der Muskelmaske. Wirkt das Resultat falsch: im Segment Editor mit **Threshold** auf dem Fettbild passende Min/Max-Werte suchen, im Modul eintragen und IMAT erneut starten.

---

## 3. Resultate (CSV)

Ausgabeordner: Feld **Output folder** (Standard: `Dokumente\KSW_BodyComposition`). Der Ordner wird gespeichert und beim nächsten Start wieder vorgeschlagen. **Open output folder** öffnet ihn im Explorer.

Pro Export entstehen:

| Datei | Inhalt |
|---|---|
| `<PatientID>_<Datum-Zeit>_body_composition.csv` | Werte dieses Patienten |
| `body_composition_all.csv` | Sammeldatei – pro Export eine neue Zeile |
| `<PatientID>_<Datum-Zeit>.seg.nrrd` | nur wenn *Also save segmentation* angehakt ist – zur späteren Kontrolle in Slicer |

Format: Trennzeichen `;`, Dezimalpunkt, UTF-8 → öffnet sich per Doppelklick korrekt in Excel (Schweiz).

| Spalte | Bedeutung | Einheit |
|---|---|---|
| `sat_cm2` | Subkutanes Fett (SAT) | cm² |
| `vat_cm2` | Viszerales Fett (VAT) | cm² |
| `sma_cm2` | Skelettmuskelfläche (SMA) | cm² |
| `imat_cm2` / `imat_pct` | Intramuskuläres Fett, absolut / in % der Muskelfläche | cm² / % |
| `vat_sat_ratio` | VAT/SAT-Verhältnis | – |
| `smi` | Skeletal Muscle Index = SMA / Grösse² | cm²/m² |
| `sat_index` / `vat_index` | SAT bzw. VAT / Grösse² | cm²/m² |
| `height_m`, `patient_id`, `patient_name`, `birth_year` | Eingaben | |
| `slice_index`, `processing_mode`, `fat_image`, `water_image`, `export_time` | Nachvollziehbarkeit | |

> **Datenschutz:** Die CSV enthält die eingegebene Patienten-ID bzw. den Namen. Ausgabeordner nur an einem dafür freigegebenen Speicherort wählen, oder eine pseudonymisierte ID verwenden.

---

## 4. Probleme

| Problem | Lösung |
|---|---|
| Modul erscheint nicht | Ordner (nicht Datei) bei *Additional module paths* eingetragen? Dateiname exakt `KSW_SemiAutomatic_BCW_module.py`? Slicer neu gestartet? |
| „Please enter the patient height“ | Körpergrösse in Metern eingeben (z. B. `1.70`, nicht `170`). |
| „No write permission for the output folder“ | Anderen Ordner wählen, z. B. unter *Dokumente*. |
| Werte unplausibel | Fett- und Wasserbild vertauscht? Richtiger L3-Schnitt gewählt? Segmente kontrolliert? |

Weitere Details zu den Segmentierungsschritten: Original-Anleitung [`Semi_Automated_Pipeline_Anleitung.pdf`](Semi_Automated_Pipeline_Anleitung.pdf) (die Abschnitte zu Python und Dashboard entfallen in dieser Variante).
