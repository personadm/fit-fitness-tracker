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

Von Emre bestätigt und in `datenschutz.html` eingearbeitet:

| Angabe | Wert |
|---|---|
| Supabase-Region | eu-central-1, Frankfurt – keine Verarbeitung in den USA |
| Row Level Security | auf allen sechs Tabellen aktiv |
| Löschung des Kontos | sofort und vollständig per CASCADE, aktuell keine Backups |
| Serverprotokolle | 1 Tag im Free Plan, 7 Tage im Pro Plan |
| HealthKit lesend | bodyMass (Körpergewicht) |
| HealthKit schreibend | HKWorkoutType, activeEnergyBurned |
| Standard-Sichtbarkeit | `friends` |

### Offen

**Mailversand.** Läuft noch über den Supabase-Standardversand. Sobald ein eigener
Anbieter feststeht (Resend, Postmark, Brevo – jeweils mit AVV und EU-Rechenzentrum),
muss Abschnitt 12 ergänzt werden.

**Bei Wechsel auf einen bezahlten Supabase-Plan** ändern sich zwei Angaben:
Es greifen tägliche Backups mit sieben Tagen Aufbewahrung, und die Log-Aufbewahrung
steigt auf sieben Tage. Beides ist in der Erklärung bereits mit „spätestens sieben
Tage" abgedeckt.

**Health-Daten müssen auf dem Gerät bleiben.** Die Erklärung sagt zu, dass aus Apple
Health gelesene Werte nicht an den Server gehen. Das stimmt aktuell, weil die
Lesefunktion nicht aufgerufen wird. Sobald sie aktiviert wird, landen die Werte über
die Synchronisation von `bodyData` automatisch auf dem Server – dann muss entweder
getrennt oder die Erklärung geändert werden.

## Fehlt noch

`nutzungsbedingungen.html` ist im Footer verlinkt, aber noch nicht erstellt. Bis dahin
entweder den Link entfernen oder die Seite ergänzen. Für die Community-Funktionen
(Profile, Feed, Meldungen, Sperrungen) sind eigene Nutzungsbedingungen nötig –
Apples Standard-EULA deckt das nicht ab.
