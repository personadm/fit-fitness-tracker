# fin – Rechtstexte und Support

Statische Seiten für die App **fin**. Kein Build, keine Abhängigkeiten – reines HTML und CSS.

## Dateien

| Datei | Zweck |
|---|---|
| `index.html` | Startseite mit Verweisen auf die drei Seiten |
| `datenschutz.html` | Datenschutzerklärung – URL für App Store Connect |
| `impressum.html` | Anbieterkennzeichnung nach § 5 DDG |
| `support.html` | Support-Seite – URL für App Store Connect |
| `style.css` | Gemeinsames Stylesheet |
| `fortschritt.js` | Fortschrittsbalken beim Scrollen |

## Veröffentlichen über GitHub Pages

1. Auf github.com ein neues Repository anlegen, z. B. `fin-web`. Sichtbarkeit **Public** – bei privaten Repos ist Pages kostenpflichtig.
2. Alle Dateien aus diesem Ordner hochladen. Am schnellsten per **Add file → Upload files**, die Dateien ins Fenster ziehen, dann **Commit changes**.
3. Im Repository auf **Settings → Pages**.
4. Unter *Source* **Deploy from a branch** wählen, Branch `main`, Ordner `/ (root)`, dann **Save**.
5. Nach ein bis zwei Minuten ist die Seite erreichbar unter:

```
https://<dein-github-name>.github.io/fin-web/
```

Die URLs für App Store Connect lauten dann:

```
https://<dein-github-name>.github.io/fin-web/datenschutz.html
https://<dein-github-name>.github.io/fin-web/support.html
```

## Eigene Domain (empfohlen)

Schöner ist eine Subdomain wie `fin.gymmittim.de`:

1. Beim Domain-Anbieter einen **CNAME**-Eintrag anlegen:
   `fin` → `<dein-github-name>.github.io`
2. In **Settings → Pages → Custom domain** `fin.gymmittim.de` eintragen und speichern.
3. **Enforce HTTPS** aktivieren, sobald das Zertifikat ausgestellt ist (dauert bis zu einer Stunde).

GitHub legt dabei automatisch eine Datei `CNAME` im Repository an. Nicht löschen.

## Vor der Einreichung prüfen

- [ ] Beide URLs im privaten Browserfenster öffnen – sie müssen ohne Login erreichbar sein
- [ ] Datenschutz-URL in App Store Connect unter *App-Datenschutz* eintragen
- [ ] Support-URL in App Store Connect bei der Version eintragen
- [ ] Beide Seiten aus der App heraus verlinken (Profil → Einstellungen)

## Noch mit Emre abzugleichen

Die folgenden Angaben stehen in der Datenschutzerklärung und müssen zur tatsächlichen Umsetzung passen:

| Angabe | Eingesetzt |
|---|---|
| Löschung nach Kontolöschung, inkl. Backups | 30 Tage |
| Aufbewahrung von Meldungen | 12 Monate |
| Aufbewahrung der Serverprotokolle | 30 Tage |
| Voreinstellung Sichtbarkeit neuer Einheiten | nur ich |
| Aus Apple Health gelesen | Schlafdaten, Herzfrequenz |
| Nach Apple Health geschrieben | abgeschlossene Trainingseinheiten |

Wenn davon etwas abweicht, in `datenschutz.html` anpassen.

## Fehlt noch

`nutzungsbedingungen.html` ist im Footer verlinkt, aber noch nicht erstellt. Bis dahin
entweder den Link entfernen oder die Seite ergänzen. Für die Community-Funktionen
(Profile, Feed, Meldungen, Sperrungen) sind eigene Nutzungsbedingungen nötig –
Apples Standard-EULA deckt das nicht ab.
