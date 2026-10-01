# Tankbon

Handy-Seite, die per Knopfdruck einen frischen Coop Pronto Tankbon holt, ihn ohne weissen
Rand anzeigt und beim Tippen nur den Barcode quer und gross zeigt.

```
Handy (GitHub Pages: index.html) ──► Cloudflare Worker (worker.js) ──► coop-pronto.ch
```

## Teil A: Zwei Adressen aus den DevTools holen (Computer)

1. Coop-Seite öffnen, `F12`, Tab **Network**, **Keep log** an.
2. Filter leeren, oben **Fetch/XHR** wählen, mit 🚫 die Liste leeren.
3. Auf der Seite **Jetzt herunterladen** klicken. Es erscheinen mindestens zwei Einträge.
4. **Download-Adresse:** Rechtsklick auf `download-coupon?url=…` → **Copy → Copy URL**.
   Irgendwo zwischenspeichern.
5. **Generieren-Adresse:** Den Eintrag **direkt davor** anklicken (Initiator meist ebenfalls
   `coupon-teaser.ts`). Im Tab **Response** sollte der Dateiname `….pdf` stehen. Notieren:
   - Tab **Headers**: *Request URL* und *Request Method*
   - Tab **Payload** (falls vorhanden): auf **view source** klicken und den Text kopieren

## Teil B: Cloudflare Worker

1. https://dash.cloudflare.com → kostenlos registrieren, E-Mail bestätigen.
2. Links **Workers & Pages** (bzw. **Compute → Workers & Pages**) → **Create** →
   **Start with Hello World!**.
3. Name `coop-tankbon` → **Deploy**. Beim ersten Mal fragt Cloudflare nach einer
   workers.dev-Subdomain: beliebigen Namen wählen.
4. **Edit code** → alles im Editor löschen → Inhalt von `worker.js` einfügen.
5. Oben in `CONFIG` eintragen:
   - `GENERATE_URL`, `GENERATE_METHOD`, `GENERATE_BODY` aus Teil A, Punkt 5
   - `DOWNLOAD_URL` aus Teil A, Punkt 4
6. **Deploy** klicken. Die Worker-Adresse steht oben, z.B.
   `https://coop-tankbon.deinname.workers.dev`
7. Test im Browser:
   - `…workers.dev/?debug=generate` zeigt die Antwort von Schritt 1 und den gefundenen Dateinamen
   - `…workers.dev/` sollte direkt ein PDF anzeigen ✅

Findest du in Teil A den Generieren-Request nicht: `…workers.dev/?debug=1` öffnen. Der Worker
durchsucht die JavaScript-Dateien der Coop-Seite und zeigt Code-Stellen mit «coupon».

## Teil C: GitHub Pages

1. https://github.com → Konto erstellen bzw. einloggen.
2. Oben rechts **+ → New repository**. Name: `tankbon`, **Public**, **Create repository**.
3. Auf der leeren Repo-Seite auf **uploading an existing file** klicken.
4. `index.html` und `README.md` hineinziehen → **Commit changes**.
5. In der Dateiliste auf `index.html` klicken → Stift-Symbol ✏️ → in Zeile mit
   `const WORKER_URL = "";` deine Worker-Adresse zwischen die Anführungszeichen setzen →
   **Commit changes**.
6. **Settings → Pages** (links). Unter *Build and deployment*: Source **Deploy from a branch**,
   Branch **main**, Ordner **/ (root)** → **Save**.
7. Nach 1–2 Minuten ist die Seite unter `https://DEINNAME.github.io/tankbon/` erreichbar
   (Link erscheint oben auf der Pages-Einstellungsseite).

## Teil D: Aufs Handy

- iPhone (Safari): Seite öffnen → Teilen → **Zum Home-Bildschirm**
- Android (Chrome): Seite öffnen → Menü ⋮ → **Zum Startbildschirm hinzufügen**

## Optional: Zugriff einschränken

Die Worker-Adresse steht in der öffentlichen `index.html`. Wer will, kann im Worker unter
**Settings → Variables and Secrets** ein Secret `ACCESS_KEY` anlegen und denselben Wert in
`index.html` bei `ACCESS_KEY` eintragen. Zusätzlich in `worker.js` `ALLOWED_ORIGINS` auf
`["https://DEINNAME.github.io"]` setzen.

## Fehlerhilfe

| Meldung | Lösung |
|---|---|
| Worker nicht erreichbar | `WORKER_URL` in index.html prüfen (mit `https://`) |
| GENERATE_URL / DOWNLOAD_URL ist leer | In Cloudflare eintragen, Deploy nicht vergessen |
| Generieren fehlgeschlagen (HTTP 400/403/419) | Payload prüfen, CSRF-Token durch `{CSRF}` ersetzen |
| Kein Dateiname gefunden | `?debug=generate` öffnen und Antwort ansehen |
| Barcode-Ansicht zeigt ganzen Bon | Erkennung fand keinen Barcode, Screenshot vom PDF schicken |

Coop kann die Seite jederzeit ändern, dann Teil A wiederholen. Jeder Bon gilt nur einmal
und bis 31.12.2026.
