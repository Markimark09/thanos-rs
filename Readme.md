# Thanos-RS

**Verkleinert Minecraft-Welten, indem Chunks entfernt werden, in denen kaum jemand war.**
Schnell, in Rust geschrieben, mit futuristischer GUI (Drag and Drop, Fortschrittsbalken) und optionaler ZIP-Ausgabe.

> Die Original-Welt bleibt unverändert. Thanos-RS schreibt immer in einen neuen Ausgabe-Ordner.

---

## Features

- GUI mit Ordnerauswahl und Drag and Drop
- Fortschrittsbalken für Verkleinerung und ZIP-Erstellung
- Unterstützt Overworld, Nether und End
- Unterstützt Paper 26.1.x (Dimensions-Ordner `dimensions/minecraft/...`)
- Unterstützt LZ4-komprimierte Regionsdateien
- Mehrere Threads für hohe Geschwindigkeit
- Einstellbarer Schwellenwert (`InhabitedTime`)
- Optionale ZIP-Datei zum direkten Teilen
- Eine einzelne `.exe`, keine Installation von Zusatzprogrammen nötig

---

## Download

Fertige Version unter **Releases** im Repository herunterladen und `thanos-rs.exe` starten.

---

## Benutzung (GUI)

1. `thanos-rs.exe` starten.
2. **Welt-Ordner** auswählen oder per Drag and Drop in das Fenster ziehen.
   Der Ordner muss eine `level.dat` enthalten.
3. **Ausgabe-Ordner** wählen. Er darf noch nicht existieren.
4. Schwellenwert und Threads einstellen (Standardwerte passen meistens).
5. Optional **ZIP erstellen** aktivieren.
6. **Start** klicken und den Fortschrittsbalken abwarten.

---

## Benutzung (Kommandozeile)

```bash
thanos-rs --input "C:\Pfad\zur\welt" --output "C:\Pfad\neue_welt" --threshold 600 --threads 8
```

| Option | Beschreibung |
|---|---|
| `--input` | Quell-Weltordner (mit `level.dat`) |
| `--output` | Neuer Ausgabe-Ordner (darf nicht existieren) |
| `--threshold` | Mindest-`InhabitedTime` in Ticks, darunter wird der Chunk entfernt |
| `--threads` | Anzahl der Worker-Threads |

---

## Wie funktioniert es?

Jeder Chunk speichert in `InhabitedTime`, wie lange Spieler dort waren. Chunks unter dem Schwellenwert werden nicht in die neue Welt übernommen und vom Spiel bei Bedarf neu generiert. Bauwerke und besuchte Gebiete bleiben erhalten.

Zeitangaben: 20 Ticks = 1 Sekunde, also 1200 Ticks = 1 Minute.

---

## Welt nutzen

### Auf einem Paper-Server

Ordner einfach als Weltordner verwenden.

### Im Vanilla-Singleplayer (Version 26.1.2)

Paper-Welten brauchen eine Anpassung, sonst erscheint der Fehler `Unable to read or access the world gen settings file`:

1. In Minecraft 26.1.2 eine neue Welt mit **demselben Seed** erstellen und beenden.
2. Aus dieser Welt `data/minecraft/world_gen_settings.dat` kopieren.
3. In die verkleinerte Welt nach `data/minecraft/` einfügen.
4. Die Welt in `.minecraft/saves` legen und starten.

Die Warnung `Missing data pack paper` kann ignoriert werden.

---

## Welt weitergeben

1. Kopie der Welt anlegen.
2. `session.lock` und `level.dat_old` löschen, bei Bedarf auch `playerdata`, `stats`, `advancements`.
3. Den Weltordner als ZIP packen, sodass `level.dat` direkt im Ordner in der ZIP liegt.
4. ZIP über Google Drive, MEGA oder OneDrive teilen.
5. Empfänger entpacken die ZIP nach `.minecraft/saves` und starten mit **Minecraft 26.1.2**.

---

## Selbst bauen

Voraussetzung: aktuelles Rust (`rustup update`).

```bash
git clone https://github.com/DEIN-NAME/thanos-rs.git
cd thanos-rs
cargo build --release
```

Die fertige Datei liegt unter `target/release/thanos-rs.exe`.

Tests ausführen:

```bash
cargo test
```

---

## Hinweise

- Vor dem Verkleinern immer ein Backup anlegen.
- Server vorher stoppen, damit keine Dateien gesperrt oder halb geschrieben sind.
- Je höher der Schwellenwert, desto kleiner wird die Welt.
- Die ZIP-Größe hängt vom Inhalt ab. Regionsdateien lassen sich kaum weiter komprimieren.

---

## Lizenz

MIT, siehe `LICENSE`.
