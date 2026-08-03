# Digitale Trauerkarte — Klaus Hochstrate

Statische Website, ein Verzeichnis, keine Build-Schritte. `index.html` ist der
Einstiegspunkt und muss im Wurzelverzeichnis des Repos liegen.

**Veröffentlichungsadresse (fest eingetragen):**
`https://steftai.github.io/trauerkarte/`

Diese Adresse steht bereits in `index.html` in `og:image` und `og:url`. Wird das
Repo umbenannt oder unter einem anderen Konto veröffentlicht, müssen beide
Werte angepasst werden — sonst zeigt WhatsApp keine Kachel.

## Dateien

| Datei | Zweck |
|---|---|
| `index.html` | Die Karte. Gebündeltes Artefakt, Inhalt liegt base64-kodiert im `<script>`-Block. Vor Änderungen `HANDOVER.md`, Abschnitt 3 lesen. |
| `manifest.json` | Web-App-Manifest für „Zum Startbildschirm hinzufügen". |
| `cover.jpg` | 1600 × 1600, Ziel von `og:image`. Erzeugt die WhatsApp-Kachel. |
| `icon-192.png`, `icon-512.png` | Aktives Startbildschirm-Symbol (voller Name). |
| `icon-mono-192.png`, `icon-mono-512.png` | Alternative mit Monogramm `KH`. Bei Bedarf über die aktiven Dateien kopieren. |
| `.nojekyll` | Schaltet die Jekyll-Verarbeitung von GitHub Pages ab. Nicht löschen. |
| `HANDOVER.md` | Technische Übergabe: warum Hosting nötig ist, Aufbau der Datei, offene Punkte. |

## Veröffentlichen (GitHub Pages)

1. Repo `trauerkarte` unter dem Konto `steftai` anlegen — **öffentlich**
   (Pages aus privaten Repos erfordert ein kostenpflichtiges Konto).
2. Alle Dateien dieses Verzeichnisses in den Branch `main` laden.
3. Settings → Pages → Source: `Deploy from a branch`, Branch: `main`, Ordner: `/ (root)`.
4. Ein bis zwei Minuten warten, dann `https://steftai.github.io/trauerkarte/` aufrufen.

## Abnahmeprüfung

- `https://steftai.github.io/trauerkarte/cover.jpg` lädt ohne Login.
- Im Quelltext der Live-Seite steht in `og:image` eine absolute `https`-URL.
- Link in WhatsApp an sich selbst senden — Kachel mit Bild und Titel erscheint.
  WhatsApp cacht pro URL; zum erneuten Testen `?v=2` anhängen.

## Offen vor dem Versand

Im Karteninhalt stehen im Abschnitt „Im Anschluss" (Café am Feld) noch die
Platzhalter `[Adresse]` und `[Uhrzeit]`. Sie liegen in der kodierten Nutzlast,
nicht im Klartext — Vorgehen siehe `HANDOVER.md`, Abschnitt 3.1.
