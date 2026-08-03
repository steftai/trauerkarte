# HANDOVER — Digitale Trauerkarte Klaus Hochstrate

Stand: 3. August 2026
Übergabe an: nachfolgende Coding-Instanz (Claude Code o. ä.)

---

## 1. Ziel des Projekts

Eine digitale Trauerkarte existiert als **einzelne, in sich geschlossene
HTML-Datei** und soll per WhatsApp verschickt werden.

Der Auftraggeber möchte, dass der Empfänger in WhatsApp **kein graues
`HTM`-Dokumentsymbol** sieht, sondern **ein farbiges Bild / eine Kachel,
die sich per Antippen öffnet** — und dass sich die Karte dabei **ohne
App-Auswahldialog direkt im Standard-Browser** öffnet.

Zusätzlich gewünscht: ein echtes, farbiges **Symbol auf dem
Startbildschirm** des Telefons, hinter dem die Karte liegt.

---

## 2. Wichtigste technische Erkenntnis (bitte nicht erneut untersuchen)

**WhatsApp erzeugt für `.html`-Anhänge grundsätzlich keine Vorschau.**
Miniaturbilder werden clientseitig nur für Bilder, Videos, PDF (erste
Seite) und Office-Dokumente erzeugt. Das graue `HTM`-Symbol wird von
WhatsApp anhand der Dateiendung gezeichnet — es stammt nicht aus der
Datei. Kein Meta-Tag, kein eingebettetes Bild und kein Trick im Markup
ändert das.

**Einziger funktionierender Weg:** Die Datei muss unter einer URL
erreichbar sein. Dann greifen:

| Wunsch | Umsetzung |
|---|---|
| Farbige Kachel in WhatsApp, die sich per Klick öffnet | Link-Vorschau über Open-Graph-Tags (`og:image` etc.) |
| Öffnet direkt im Standard-Browser, kein Auswahldialog | Ergibt sich automatisch — Links umgehen den Dialog, Datei-Anhänge nicht |
| Farbiges Symbol auf dem Startbildschirm | `manifest.json` + `apple-touch-icon`, dann „Zum Startbildschirm hinzufügen" |

Alternativen, die verworfen wurden:
- **PDF-Variante** — bekäme in WhatsApp eine echte Seitenvorschau,
  verliert aber Interaktivität (Öffnen-Button, „IBAN kopieren",
  Animationen). Bleibt als Rückfalloption offen.
- **Bild + Datei nacheinander senden** — funktioniert sofort ohne
  Hosting, ist aber zwei Nachrichten statt einer Kachel.

---

## 3. Aufbau der HTML-Datei — bitte vor jeder Änderung lesen

`index.html` ist **kein normales Dokument**. Es ist ein gebündeltes
Artefakt:

- Alle Assets (Schriften, Bilder, QR-Code) liegen **base64-kodiert in
  einer JS-Nutzlast** im `<script>`-Block.
- Beim Laden entpackt ein Bootstrapper diese Nutzlast und **ersetzt das
  komplette Dokument** (DOM-Austausch). Der sichtbare Inhalt entsteht
  also erst zur Laufzeit.

Konsequenzen:

1. **Der sichtbare Text steht nicht im Klartext im `<body>`.**
   Textänderungen (siehe offene Punkte, Abschnitt 6) müssen in der
   kodierten Nutzlast erfolgen — Nutzlast dekodieren, ändern, neu
   kodieren. Nicht per einfachem Suchen-und-Ersetzen im Markup.
2. **Alles, was im `<head>` steht, wird beim DOM-Austausch entfernt** —
   inklusive `<title>`, `<link rel="manifest">` und der Icon-Links.
   Open-Graph-Tags sind davon **nicht** betroffen, weil WhatsApp und
   andere Scraper den rohen Quelltext lesen und kein JavaScript
   ausführen.
3. Für Punkt 2 wurde ein **Wachhund-Skript** im `<head>` ergänzt (siehe
   Abschnitt 4). Es darf nicht entfernt werden, sonst verliert
   „Zum Startbildschirm hinzufügen" das eigene Symbol.

---

## 4. Was bereits umgesetzt ist

Alle Änderungen liegen ausschließlich im `<head>` von `index.html`,
direkt am Dateianfang zwischen den Kommentarzeilen. Der Karteninhalt
selbst wurde **nicht** angefasst.

- **Open-Graph-Tags** — `og:title`, `og:description`, `og:image`
  (1600 × 1600), `og:url`, `twitter:card`. Erzeugen die WhatsApp-Kachel.
- **`<link rel="manifest">`, `icon`, `apple-touch-icon`,
  `theme-color`** — für das Startbildschirm-Symbol.
- **`<title>`** von `Bundled Page` auf den richtigen Titel geändert.
- **Wachhund-Skript** — setzt Titel, Manifest- und Icon-Links nach dem
  DOM-Austausch wieder ein. Nutzt einen `MutationObserver` auf
  `document` mit 100-ms-Entprellung, zusätzlich `DOMContentLoaded` und
  `load`. Getestet: Titel und alle drei `<link>`-Elemente sind nach dem
  Entpacken vorhanden, keine Konsolenfehler.

---

## 5. Dateien in diesem Paket

| Datei | Zweck |
|---|---|
| `index.html` | Die Karte, ergänzt um Head-Tags und Wachhund. Einstiegspunkt. |
| `manifest.json` | Web-App-Manifest, `display: standalone`, verweist auf `icon-192/512`. |
| `cover.jpg` | 1600 × 1600, Titelseite der Karte. Ziel von `og:image`. |
| `icon-192.png`, `icon-512.png` | Symbol mit vollem Namen und Lebensdaten. Aktiv. |
| `icon-mono-192.png`, `icon-mono-512.png` | Alternative mit Monogramm `KH`. Bei kleinen Anzeigegrößen besser lesbar — bei Bedarf über die aktiven Dateien kopieren. |
| `HANDOVER.md` | Dieses Dokument. |

Farbwelt der Karte: Hintergrund `#FAF7F0`, Gold `#A8842C`, Schrift
`#1a1a1a`, Serifenschrift im Stil von Georgia.

---

## 6. Offene Punkte

**Blockierend für die Veröffentlichung:**

1. **Platzhalter-URL ersetzen.** In `index.html` steht an zwei Stellen
   `https://DEINE-ADRESSE.netlify.app` (in `og:image` und `og:url`).
   Beide müssen auf die tatsächliche Adresse zeigen — **absolute URLs,
   relative Pfade werden von WhatsApp nicht zuverlässig aufgelöst.**
2. **Zwei unausgefüllte Platzhalter im Karteninhalt.** Im Abschnitt
   „Im Anschluss" (Café am Feld) stehen wörtlich `[Adresse]` und
   `[Uhrzeit]`. Diese müssen vor dem Versand gefüllt werden — Änderung
   erfolgt in der kodierten Nutzlast, siehe Abschnitt 3.1. Beim
   Auftraggeber rückfragen, welche Angaben dort hineingehören.

**Optional:**

3. PDF-Variante als Rückfalloption erzeugen (Seitenvorschau in
   WhatsApp, dafür ohne Interaktivität).
4. Icon-Auswahl bestätigen — voller Name (aktiv) oder Monogramm.
   Der volle Name ist bei 48 px Anzeigegröße auf dem Startbildschirm
   nur noch als Muster erkennbar.

---

## 7. Veröffentlichung

Beliebiger Anbieter für statische Seiten, alle fünf Dateien im selben
Verzeichnis, `index.html` im Wurzelverzeichnis:

- **Netlify Drop** (`app.netlify.com/drop`) — Ordner oder Archiv per
  Drag-and-drop, kostenloses Konto nötig.
- **Cloudflare Pages** oder **GitHub Pages** — gleichwertig.

Reihenfolge beachten: erst hochladen, dann die erhaltene Adresse in die
beiden Platzhalter eintragen, dann erneut hochladen. Vorher greift die
Vorschau nicht.

**Abnahmeprüfung:**

- Quelltext der veröffentlichten Seite abrufen, prüfen, dass `og:image`
  eine absolute, öffentlich erreichbare URL enthält.
- `cover.jpg` direkt im Browser aufrufen — muss ohne Login laden.
- Link in WhatsApp an sich selbst senden: Kachel mit Bild und Titel
  muss erscheinen. WhatsApp cacht Vorschauen pro URL; zum erneuten
  Testen einen veränderten Parameter anhängen.
- Auf Android: Link antippen → Karte öffnet sich im Browser, dann
  Menü → „Zum Startbildschirm hinzufügen" → Symbol und Name prüfen.

---

## 8. Randnotiz zum Standard-Browser

Lässt sich nicht aus der Datei heraus erzwingen, das ist eine
Geräteeinstellung. Bei einer heruntergeladenen HTML-Datei erscheint
unter Android der App-Auswahldialog; dort einmal „Immer" wählen, oder
unter Einstellungen → Apps → Standard-Apps festlegen. **Links umgehen
den Dialog ohnehin** — ein weiteres Argument für den Hosting-Weg.
