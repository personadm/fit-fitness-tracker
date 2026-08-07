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

## Stand der technischen Angaben

In `datenschutz.html` eingearbeitet:

| Angabe | Wert |
|---|---|
| Supabase-Region | eu-central-1, Frankfurt |
| Row Level Security | auf allen Tabellen aktiv |
| Löschung des Kontos | sofort und vollständig per CASCADE |
| Serverprotokolle | max. 7 Tage |
| HealthKit lesend | Schlaf inkl. Phasen, Herzfrequenz, HRV, Ruhepuls, Schritte, Aktivenergie, Körpergewicht, Puls während des Trainings (Watch) |
| HealthKit schreibend | Trainingseinheiten, aktive Kalorien |
| Health-Daten auf dem Server | keine – Verarbeitung ausschließlich lokal |
| Sichtbarkeit | zwei Stufen: privat / geteilt |
| Soziale Daten | Kommentare, Reaktionen, Folgeanfragen, In-App-Benachrichtigungen, Meldungen, Blockierungen |
| Geschlecht | für Hyrox-Zusatzgewichte, nur für den Nutzer sichtbar |

### Muss in der App dazu passen

**Sichtbarkeit auf zwei Stufen umbauen.** Der Text kennt nur noch *privat* und *geteilt*.
Solange die App drei Optionen anzeigt, stimmen Text und Produkt nicht überein.

**Benachrichtigungen löschen.** Der Text sagt zu, dass gelesene In-App-Benachrichtigungen
nach 90 Tagen entfernt werden. Das braucht einen Job oder eine Löschregel.

**Health-Daten müssen lokal bleiben.** Die Erklärung sagt ausdrücklich zu, dass kein aus
Apple Health gelesener Wert den Server erreicht. Sobald ein Wert mitsynchronisiert wird,
ist die Aussage falsch.

### Offen

**Mailversand.** Läuft über den Supabase-Standardversand. Sobald ein eigener Anbieter
feststeht, muss Abschnitt 12 ergänzt werden.

**Push über Server.** Der Text beschreibt Benachrichtigungen als rein lokal bzw. in-app.
Sobald echte Push-Nachrichten dazukommen, müssen der Apple Push Notification service als
Empfänger und der Gerätetoken ergänzt werden.

**Ausdauer- und Hyrox-Daten.** Falls Distanz, Zeit und Pace erfasst werden, gehören sie
in die Aufzählung in Abschnitt 4.

## Fehlt noch

`nutzungsbedingungen.html` ist im Footer verlinkt, aber noch nicht erstellt. Bis dahin
entweder den Link entfernen oder die Seite ergänzen. Für die Community-Funktionen
(Profile, Feed, Meldungen, Sperrungen) sind eigene Nutzungsbedingungen nötig –
Apples Standard-EULA deckt das nicht ab.
