# Thanos-rs – Wiki

Tool zum Entfernen ungenutzter Chunks aus Minecraft-Welten, basierend auf `InhabitedTime`. Geschrieben in Rust.

## Was macht das Tool

Server-Welten wachsen mit der Zeit vor allem durch Chunks, die zwar generiert, aber nie wirklich besucht wurden (Preloading, Anti-Xray, Karten-Mods, Elytra-Überflüge etc.). Thanos-rs liest jede Region-Datei, prüft pro Chunk den `InhabitedTime`-Wert (Zeit, die Spieler dort tatsächlich verbracht haben) und lässt Chunks unterhalb eines Schwellenwerts beim Schreiben der neuen Welt einfach weg. Minecraft generiert sie beim nächsten Betreten neu.

Der Rest der Welt (level.dat, playerdata, stats, ...) wird 1:1 in den Zielordner kopiert. Die Original-Welt bleibt unangetastet.

## Build

Voraussetzung: Rust + Cargo installiert.

```
git clone https://github.com/byquanton/thanos-rs.git
cd thanos-rs
cargo build --release
```

Ergebnis liegt unter `target/release/thanos-rs.exe` (Windows) bzw. `target/release/thanos-rs` (Linux/macOS).

## Benutzung

```
thanos-rs <input_dir> <output_dir> [-i TICKS]
```

- `input_dir` – Ordner der Original-Welt (enthält `region/`, `level.dat`)
- `output_dir` – Zielordner für die reduzierte Welt, sollte noch nicht existieren
- `-i / --inhabited-time` – Schwellenwert in Ticks, Standard 0

Ticks zur Orientierung: 20 Ticks = 1 Sekunde, 1200 = 1 Minute, 6000 = 5 Minuten, 72000 = 1 Stunde.

Beispiele:

```
thanos-rs "C:\Server\world" "C:\Server\world_optimiert" -i 0
thanos-rs "C:\Server\world" "C:\Server\world_optimiert" -i 2400
```

## Ablauf im Detail

1. Region-Dateien im `region/`-Ordner werden gefunden, alle anderen Dateien direkt kopiert.
2. Jeder Chunk wird entpackt (zlib/gzip via flate2) und als NBT geparst (simdnbt).
3. `InhabitedTime` wird ausgelesen, mit Fallback auf `Level.InhabitedTime` bzw. Groß-/Kleinschreibvarianten für ältere Formate.
4. Liegt der Wert unter dem Schwellenwert, fliegt der Chunk aus der neuen Region-Datei raus.
5. Rayon verteilt die Arbeit auf mehrere CPU-Kerne, dadurch ist auch das Durchgehen großer Welten schnell.
6. Am Ende liegt die fertige, reduzierte Welt komplett neu im `output_dir`.

## Abhängigkeiten

- clap – Parsen der Kommandozeilen-Argumente
- simdnbt – NBT-Parsing
- flate2 – Dekomprimieren der Chunk-Daten
- rayon – Multithreading

## Sicherheitshinweise

- Vor dem ersten Einsatz Backup der Welt anlegen.
- Server/Client beim Ausführen geschlossen halten.
- Zielordner sollte leer/nicht vorhanden sein.
- Erst an einer Testwelt ausprobieren, nicht direkt am Live-Server.
- Gebäude in Chunks mit niedrigem `InhabitedTime` (z. B. durchgeflogen statt betreten) gehen beim Entfernen verloren, da der Chunk komplett neu generiert wird.

## Bekannte Build-Probleme

**E0308 bei `simdnbt::borrow::Nbt`**
Neuere simdnbt-Versionen (ab 0.6) liefern `Nbt::Some(...)` / `Nbt::None` statt `Option<Nbt>`. Match-Pattern entsprechend anpassen, `.as_compound()` verwenden um an die Tags zu kommen.

**`failed to select a version for simdnbt`**
Version in `Cargo.toml` existiert nicht mehr auf crates.io, auf verfügbare Version (z. B. 0.6) heben, dann:

```
cargo clean
cargo build --release
```

## FAQ

**Wird die Originalwelt verändert?**
Nein, es wird nur gelesen. Die neue Welt entsteht komplett separat im Zielordner.

**Funktioniert es mit jeder Minecraft-Version?**
Es unterstützt das normale Anvil-Format und deckt verschiedene InhabitedTime-Pfade ab, dadurch auch ältere Formate.

**Wie viel bringt es an Ersparnis?**
Kommt stark auf die Welt an. Bei Servern mit viel unbesuchtem, aber generiertem Terrain sind 30-70% weniger Weltgröße realistisch.

**Kann ich mehrere Schwellenwerte testen?**
Nicht in einem Durchlauf, einfach mehrfach mit unterschiedlichem `-i` und jeweils neuem Zielordner laufen lassen und vergleichen.

Sources:
- https://wiki.vg/NBT
- https://wiki.vg/Region_Files
