# Webinar-Landingpage · Leadway × Synthesia × Kantonsspital Baden

Statische Landing Page (Single-File `index.html`) für das Webinar „KI-Video im
Gesundheitswesen" am 8. Juli 2026.

- **Live:** https://leadway.ch/webinar-synthesia-ki-video-im-ksb/
- **Hosting:** GitHub Pages, Deploy automatisch aus dem `main`-Branch (Root).
- **Stack:** reines HTML/CSS/JS, kein Build-Schritt.

## Anmeldeformular & Datenfluss

Das Anmeldeformular ist **kein** HubSpot-Standard-Embed, sondern ein natives
Formular im Seitenstil, das per **HubSpot Forms Submissions API** direkt an
HubSpot übermittelt (Portal `147665510`, Form-GUID `e4132e4e-…`).

Bei jedem Submit passiert Folgendes (Reihenfolge):

1. **HubSpot** (Quelle der Wahrheit): Der Lead wird via Submissions-API
   angelegt. Erst wenn dieser Call erfolgreich ist (`response.ok`), …
2. **Make.com-Webhook** (Automation, nice-to-have): … wird die Anmeldung
   zusätzlich an ein Make-Szenario weitergereicht. Dieses registriert den
   Teilnehmer in Zoom, verschickt eine personalisierte Bestätigungs-Mail
   (Gmail), sendet eine Google-Calendar-Einladung und reichert UTM-Daten in
   HubSpot an.

Der Webhook-Call ist **fire-and-forget**: Er wird nicht awaitet und blockiert
weder die Danke-Ansicht noch den HubSpot-Erfolg. Schlägt er fehl, gibt es nur
einen `console.error`, kein User-facing Error. `keepalive: true` stellt sicher,
dass der Request auch bei sofortiger Navigation (z. B. Danke-Seite) zu Ende
läuft.

### UTM-Tracking

Beim Page-Load werden `utm_source`, `utm_medium`, `utm_campaign` aus der URL in
`sessionStorage` zwischengespeichert (Init-Script im `<head>`). So bleiben sie
erhalten, falls der User vor dem Absenden noch navigiert. Im Webhook-Payload
werden die UTMs aus der aktuellen URL gelesen, mit Fallback auf `sessionStorage`.

## Webhook-URL für künftige Webinare austauschen

Pro Webinar wird **nur eine Konstante** getauscht:

- **Datei:** `index.html`
- **Konstante:** `MAKE_WEBHOOK_URL` (im Submit-Script-Block am Ende der Datei,
  direkt unter der HubSpot-`ENDPOINT`-Definition)

```js
var MAKE_WEBHOOK_URL = "https://hook.eu1.make.com/…";
```

Neue Webhook-URL aus dem Make-Szenario kopieren, hier eintragen, committen,
pushen — fertig.

## Lokal testen

1. Make-Szenario im Editor öffnen und auf **„Run once"** stellen (wartet auf
   einen einzelnen eingehenden Webhook).
2. `index.html` lokal öffnen (z. B. `python3 -m http.server` im Repo-Root) und
   das Formular ausfüllen + absenden.
3. In der **Make-History** prüfen, dass das Szenario grün durchgelaufen ist und
   alle Felder + UTMs im Payload ankamen.

**UTMs mittesten:** Seite mit `?utm_source=test&utm_medium=test&utm_campaign=test`
öffnen → die Werte müssen im Webhook-Payload erscheinen.

**Persistenz mittesten:** Seite mit UTM-Parametern öffnen, auf eine andere Seite
navigieren, zurück zur Landing Page (jetzt ohne UTM in der URL), Formular
absenden → die UTMs müssen aus `sessionStorage` trotzdem im Payload sein.

Der Webhook-Payload sieht so aus:

```json
{
  "firstName": "…",
  "lastName": "…",
  "email": "…",
  "company": "…",
  "jobtitle": "…",
  "utm_source": "…",
  "utm_medium": "…",
  "utm_campaign": "…"
}
```

## TODO / Offen

- [ ] **Datenschutzerklärung ergänzen:** Hinweis aufnehmen, dass zur
  Webinar-Organisation Daten an **Make.com (Celonis SE, EU-Region)** und
  **Zoom Video Communications** übermittelt werden. (Nicht Teil der
  Webhook-Integration — separate Änderung an der Datenschutz-Seite.)
