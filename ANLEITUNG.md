# fin Zahlen: Dashboard einrichten

Zweig `dashboard` (d9031aae, Funde der Gegenprüfung in d3ab7a3c). Drei Teile:
`app/supabase/134_dashboard.sql`, die Edge Function `dashboard` und die Seite
`dashboard.html` aus diesem Ordner. Nichts davon ist eingespielt oder deployt.

## 1. 134 einspielen

1. Supabase, SQL-Editor, neue Abfrage.
2. Den ganzen Inhalt von `app/supabase/134_dashboard.sql` einfügen und einmal
   ausführen. Die Datei hat `begin`/`commit` und einen Selbsttest; schlägt er
   an, wird alles zurückgerollt. Ein zweiter Lauf ist folgenlos.
3. Probe (nur lesen):

   ```sql
   select jsonb_pretty(public.dashboard_zahlen(14));
   ```

   Steht unter `fehlt` ein Tabellenname, gibt es die Tabelle in der
   Datenbank nicht; die Zahl dazu bleibt leer, sonst passiert nichts.

## 2. Geheimnis setzen

Im Terminal, im Ordner `app` (Projekt ist verlinkt):

```sh
SCHLUESSEL=$(openssl rand -hex 24)
echo "$SCHLUESSEL"          # in den Passwortmanager, nirgends sonst
supabase secrets set DASHBOARD_SCHLUESSEL="$SCHLUESSEL"
```

Mindestens 24 Zeichen, sonst antwortet die Function mit 503.

Dazu `DASHBOARD_HERKUNFT`: die Seiten, die im Browser abfragen dürfen
(Komma-Liste, jeder Eintrag exakt, kein Muster, keine Unterdomain). Fest
eingetragen ist nichts, auch claude.ai nicht. Ohne die Variable bekommt jede
Seite im Browser 403; curl ohne Origin entscheidet allein der Schlüssel.

**Empfohlen:** die Seite unter einem nicht verlinkten Pfad auf fin-tracker.de
ablegen (Schritt 4) und genau diese Herkunft freigeben:

```sh
supabase secrets set DASHBOARD_HERKUNFT="https://fin-tracker.de"
```

**Lokal geöffnet** (Doppelklick auf die Datei) schickt der Browser die
Herkunft `null`. Dafür `DASHBOARD_HERKUNFT="null"` setzen, aber ehrlich:
`null` steht nicht für eine Seite, sondern für jede opake Herkunft (lokale Dateien,
`data:`-Adressen, Sandbox-iframes auf beliebigen fremden Seiten). CORS ist
damit für alle diese offen, und **nur noch der Schlüssel schützt**. Das geht,
solange der Schlüssel lang ist und nur im eigenen Browser liegt. Wer es
enger will, nimmt fin-tracker.de.

## 3. Function deployen

```sh
supabase functions deploy dashboard --no-verify-jwt
```

`--no-verify-jwt`, weil die Seite kein Supabase-Konto hat; sie schickt nur den
Schlüssel im Kopf `x-dashboard-schluessel`. Kurz prüfen:

```sh
curl -s -H "x-dashboard-schluessel: $SCHLUESSEL" \
  "https://feylbgjusxpvaxvcwssd.supabase.co/functions/v1/dashboard?tage=14" | head -c 300
```

Ohne oder mit falschem Schlüssel kommt `{"fehler":"schluessel"}` (401).

## 4. Seite öffnen und Schlüssel eintragen

Beim ersten Öffnen fragt die Seite nach Adresse (ist vorbelegt) und
Schlüssel. Beides bleibt nur im Browser (localStorage), nicht in der Seite.
Danach lädt sie alle 60 s neu und mit dem Knopf **Neu laden**. **Zugang**
ändert oder löscht den Schlüssel.

Wo die Seite liegt (zu Schritt 2 passend):

- **Eigene Domain (empfohlen):** `dashboard-eigenstaendig.html` (die Seite
  als ganzes HTML-Dokument) unter einem nicht verlinkten Pfad auf
  fin-tracker.de ablegen und `DASHBOARD_HERKUNFT="https://fin-tracker.de"`
  setzen. Im HTML steht kein Schlüssel; ohne Schlüssel sieht dort niemand
  etwas. Der Pfad ist kein Schutz, nur der Schlüssel.
- **Lokal:** `dashboard-eigenstaendig.html` per Doppelklick öffnen und
  `DASHBOARD_HERKUNFT="null"` setzen. Dann ist CORS für jede opake Herkunft
  offen (siehe Schritt 2), es schützt nur der Schlüssel.
- **claude.ai:** nicht freigegeben. Eine Artifact-Seite darf nach den
  Artifact-Regeln ohnehin keine fremden Adressen abrufen und zeigt dann
  **Keine Antwort**. Ist Supabase in claude.ai als Connector verbunden, ginge
  es über den Connector (`select public.dashboard_zahlen(14)`), ganz ohne
  Function und Schlüssel; dafür müsste die Seite umgebaut werden.

## Was die Seite zeigt

- **Heute:** neue Konten (iOS, Android, ohne Push), aktiv heute, Abo-Starts
  mit Probe, erste Zahlungen, Kündigungen, Lifetime (gekauft, darunter
  geschenkte Lifetime), Erstattungen, Trainings im Feed, Rückmeldungen,
  Gewinnspiel-Lose; je Kachel 14 Tage als Säulen, gestern und Schnitt.
- **Bestand jetzt:** Konten, Pro gesamt (davon nur Werbung), aktiv 7 und 30
  Tage, Lifetime (gekauft, geschenkt), zahlende Abos je Produkt und Store,
  laufende Proben, Gekündigt läuft aus, Verbundene Apps, offene Rückmeldungen,
  heute geschenktes Pro (eigene Zeile unter Weiteres).
- **Verlauf:** neue Konten je Plattform, Abo-Starts gegen erste Zahlungen,
  Tabelle aller Tage.

Keine Store-Downloads, nur Supabase. Neue Konten sind Anmeldungen, nicht
Installationen.

Abo-Zahlen: Ein Zahlungsausfall mit späterer Zahlung (Apple ohne Nachfrist,
Play ON_HOLD) zählt nicht als neuer Start; Wechsel von Geschenk zu Kauf oder
zwischen Apple und Play schon. Die Historie beginnt am 23.09.2026 (085);
Abos aus dem Anfangsbestand zählen nie als Probe, sondern als bezahlt.

## Creator (Affiliate, 148)

Neu auf der Seite: Abschnitt **Creator** unter den Tageszahlen (Variante 3).
Creator anlegen, Code an/aus, Auszahlungen buchen, Liste als CSV. Creator
sehen davon nichts. Ohne Zugang zeigt der Abschnitt erfundene Beispiel-Creator
und schreibt nichts. Vertrag: `store/AFFILIATE-BAU.md` (Zweig `affiliate`).
Die Seite vor diesem Umbau liegt als `zahlen-67fdcdcee0fb-vor-creator.html`
daneben.

Nichts davon ist eingespielt oder deployt. Reihenfolge:

1. **148 einspielen.** SQL-Editor, ganzen Inhalt von
   `app/supabase/148_affiliate.sql` einfügen, einmal ausführen. Bricht der
   Selbsttest ab, ist nichts passiert (Meldung beginnt mit `148:`). Probe:

   ```sql
   select jsonb_pretty(public.dashboard_affiliate());
   ```

   Erwartet: `"creator": []` und leere Listen.

2. **Function deployen, nur nach deinem Go.** Im Ordner `app`:

   ```sh
   supabase functions deploy dashboard --no-verify-jwt
   ```

   Geheimnisse bleiben. Probe:

   ```sh
   curl -s -H "x-dashboard-schluessel: $SCHLUESSEL" \
     "https://feylbgjusxpvaxvcwssd.supabase.co/functions/v1/dashboard?teil=creator" | head -c 300
   ```

   Die Tageszahlen laufen vorher und nachher gleich. Vor dem Deploy zeigt
   der Abschnitt „Function ohne Creator-Teil“, vor 148 „148 ist noch nicht
   eingespielt.“

3. **Seite hochladen.** `zahlen-67fdcdcee0fb.html` auf fin-tracker.de unter
   demselben Pfad ersetzen. Der Schlüssel bleibt im Browser.

4. **Ersten Creator anlegen.** „Creator anlegen“: Name, fin-Handle, Code,
   Sätze (Vorgabe 25 % Jahr, 15 % Monat), Haken „Creator-Profil in der App
   freischalten“ (nur mit Handle). Die Seite fragt vor jedem Schreiben nach.
   Der Code wirkt erst, wenn die Paywall mit Code-Feld draußen ist.

5. **Auszahlen am 15. für den Vormonat.** Auf der Karte „Auszahlung buchen“,
   Monat wählen, Betrag überweisen, Haken „Überwiesen“, buchen. „Als CSV“ in
   der Auszahlungsliste ist für Überweisung und Gutschrift. Minus aus
   Erstattungen wird vorgetragen.

6. **Notfälle per SQL.** Die Nr. einer Auszahlung steht im Detail des
   Creators unter „Auszahlungen“.

   ```sql
   -- Code sofort aus
   update public.affiliates set aktiv = false, aktiv_seit = now() where code = 'MARA';
   -- Auszahlung Nr. 7 zuruecknehmen
   begin;
   update public.affiliate_provisionen set auszahlung_id = null where auszahlung_id = 7;
   delete from public.affiliate_auszahlungen where id = 7;
   commit;
   -- Creator-Profil entziehen
   select public.creator_entziehen('@handle', 'Grund');
   ```

7. **Erst vor der Paywall mit Code:** Datenschutzseite (`store/website/`) um
   Creator-Code ergänzen (Zuordnung zum Creator, automatisches Folgen),
   Play-Datensicherheit prüfen (`store/PLAY-FORMULARE.md`), Apple-Angebot
   `fin365-c33` und Play-Angebot `creator-33` (erst nach dem AAB, mit
   Tag-Filter).

Gezählt wird so: Provision nur auf die erste Zahlung (25 % Jahr, 15 % Monat)
vom geschätzten Netto (brutto / 1,19 × 0,85). Auszahlung erst nach
Abbuchung, Erstattungen gehen von der nächsten Auszahlung ab. Nur neue
Konten, ein Creator je Konto, der erste Code gilt. Details auf der Seite
unter „So wird gezählt“.

## Aktionen (162)

Neu auf der Seite: Abschnitt **Aktionen** unter den Creatorn (Link „Aktionen“
oben neben „Creator“). Eine Aktion ist ein Code ohne Creator, zum Beispiel
BLACKFRIDAY: auf der Paywall eingeben wie einen Creator-Code, 30 Tage gratis,
dann 50 % aufs erste Jahr (24,99 € statt 49,99 €). Kein Monatsrabatt, keine
Provision, kein Folgen. **Zielgruppe** ist ein Feld der Aktion (seit 11.11.,
Tim: „für alle“), Vorgabe **Alle**:

- neue Konten: 30 Tage gratis, dann 24,99 € im ersten Jahr,
- abgelaufene Abos und verbrauchte Proben: 24,99 € sofort, kein Gratismonat,
- Monats- und Quartalsabo: Wechsel aufs Jahr zu 24,99 € (iOS je nach Stufe
  der Abo-Gruppe, Vorgabe ab der nächsten Verlängerung, siehe Schritt 3;
  Android sofort, die Restzeit wird angerechnet),
- Jahresabo (auch in der Probe, Frage an Tim offen), Lifetime, geschenktes
  Pro, Abo an einem anderen Konto oder im anderen Store: ein ehrlicher Satz
  statt eines Kaufs,
- wer schon einen Creator-Code hat, behält ihn.

„Nur neue Konten“ verhält sich wie ein Creator-Code. Ohne Zugang zeigt der
Abschnitt eine erfundene Beispiel-Aktion und schreibt nichts.
Bauplan: `store/PAKET-1111/03-blackfriday.md` (Zweig `p1111-blackfriday`).
Die Seite vor diesem Umbau ist die aus `~/Downloads/fin-dashboard-3110/`.

Nichts davon ist eingespielt oder deployt. Reihenfolge:

1. **162 einspielen** (nach 148 und 150). SQL-Editor, ganzen Inhalt von
   `app/supabase/162_aktionen.sql` einfügen, einmal ausführen. Bricht der
   Selbsttest ab, ist nichts passiert (Meldung beginnt mit `162:`). Probe:

   ```sql
   select jsonb_pretty(public.dashboard_aktionen());
   ```

   Erwartet: `"aktionen": []` und `"rabatte": [33, 50]`. Wird 148 oder 150
   später noch einmal eingespielt, danach 162 erneut einspielen.

2. **Functions deployen, nur nach deinem Go.** Im Ordner `app`:

   ```sh
   supabase functions deploy abo --no-verify-jwt
   supabase functions deploy dashboard --no-verify-jwt
   ```

   `abo` braucht `--no-verify-jwt`, sonst weist es Apples Webhooks mit 401 ab
   (Gegenprüfung 11.11.2026, `store/PAKET-1111/03-blackfriday.md`).
   `abo` erkennt danach die Apple-Kampagne `fin365-c50`, `dashboard` den Teil
   `?teil=aktionen`. `play` und `beleg` bleiben, wie sie sind. Probe:

   ```sh
   curl -s -H "x-dashboard-schluessel: $SCHLUESSEL" \
     "https://feylbgjusxpvaxvcwssd.supabase.co/functions/v1/dashboard?teil=aktionen" | head -c 300
   ```

   Vor dem Deploy zeigt der Abschnitt „Function ohne Aktionen“, vor 162
   „162 ist noch nicht eingespielt.“

3. **Apple (App Store Connect, Abo fin365, Angebotscodes).**
   - Neuer Angebotscode mit Referenzname **`fin365-c50`** (genau so, der
     Server erkennt die Stufe daran). Kunden: **Neue, Bestehende und
     Abgelaufene Abonnenten** (alle drei Haken, für Zielgruppe Alle),
     Einführungsangebot: **Ja** (neue Konten bekommen zuerst die 30 Tage
     gratis). Art **Vorauszahlung**, **1 Jahr**, **24,99 €** (Deutschland),
     andere Länder angleichen lassen.
   - **Stufen der Abo-Gruppe prüfen, bevor der nächste Build zu Apple geht.**
     Monetarisierung → Abonnements → Gruppe: `fin30`, `fin90` und `fin365`
     müssen auf **derselben Stufe** stehen. Dann stellt Apple Monats- und
     Quartalsabonnenten zur nächsten Verlängerung aufs Jahr um, und genau das
     sagt die App. Steht `fin365` höher, stellt Apple sofort um und bucht
     24,99 € gleich ab: dann in `app/src/creator.js` `APPLE_WECHSEL =
     'sofort'` setzen, bevor gebaut wird. Steht `fin365` tiefer, lehnt Apple
     ab („This offer can't be redeemed with your current subscription“) und
     der Einmalcode ist weg: so nicht starten, Stufen angleichen.
   - **Vor Black Friday in TestFlight:** Sandbox-Konto mit laufendem
     Monatsabo, Test-Aktion mit kurzem Fenster, in den Einstellungen „Aufs
     Jahr wechseln“, Code einlösen, Apples Blatt bis zum Ende. Erwartet: die
     Umstellung wie oben, nie „can't be redeemed“.
   - Einmalcodes erzeugen, Vorschlag 3.000, Ablauf 15.12.2026. CSV laden,
     dann im Ordner `app`:

     ```sh
     node scripts/affiliate-codes-sql.mjs <csv> fin365 2026-12-15 --stufe c50 > ~/Downloads/fin-affiliate-codes/c50.sql
     ```

     und die Datei im SQL-Editor ausführen. Codes nie ins Repo.
   - Preise je Land:

     ```sh
     node scripts/asc-creator-preise.mjs --rabatt 50 > supabase/abfragen/affiliate-preise-c50.sql
     ```

     und im SQL-Editor ausführen.
   - **Kein** `fin30-c50`.

4. **Google Play (Abo fin365, Basis-Abo `jahr`).** Zwei Angebote, beide mit
   Tag **`gezielt`** am Angebot (nie am Basis-Abo), kein Angebot an fin30:
   - **`creator-50`**, Berechtigung **Neukunden**: Phase 1 kostenlos
     **P30D**, Phase 2 **24,99 €** für **P1Y** (1 Zyklus), danach der
     Basispreis.
   - **`creator-50-sofort`**, Berechtigung **Vom Entwickler festgelegt**:
     eine Phase **24,99 €** für **P1Y** (1 Zyklus), danach der Basispreis.
     Für Abgelaufene und den Wechsel vom Monats- oder Quartalsabo.

   Länderpreise aus 24,99 € umrechnen lassen. **Aktivieren erst**, wenn der
   nächste Build bei Play zu 100 % live ist, und frühestens Do 26.11. abends;
   `creator-50-sofort` nach dem Ende sofort deaktivieren. Apps bis 2.7.8
   kennen den Tag nicht und könnten die Angebote als normale Probe zählen.
   Android 2.7.9 lehnt BLACKFRIDAY ab („Hat nicht geklappt“), nie mit
   falschem Preis.

5. **Datenschutzseiten hochladen.** `store/website/datenschutz.html` und
   `privacy.html` (Absatz „Aktionscodes“), vorher das Stand-Datum setzen.

6. **Seite hochladen.** `zahlen-67fdcdcee0fb.html` aus diesem Ordner auf
   fin-tracker.de unter demselben Pfad ersetzen. Der Schlüssel bleibt im
   Browser.

7. **Aktion anlegen.** „Aktion anlegen“: Name **Black Friday**, Code
   **BLACKFRIDAY**, Rabatt **50 %**, Start **27.11. 00:00**, Ende **01.12.
   00:00** (Berliner Zeit, höchstens 92 Tage), „Nachlauf in Stunden“ **0**,
   Plattformen **iOS und Android**, Zielgruppe **Alle** (Vorgabe), Haken
   „Gleich an“. Die Seite fragt vor jedem Schreiben nach. Bis zum Start sagt
   die App „Code gerade nicht aktiv“.
   Rückfall ohne Seite: `app/supabase/abfragen/aktion-blackfriday-anlegen.sql`.

8. **Code ändern.** Auf der Karte „Code ändern“, neuen Code eintragen
   (3 bis 16 Zeichen, A bis Z und 0 bis 9). Der alte Code geht sofort nicht
   mehr, wer ihn schon eingelöst hat, behält den Preis. „Bearbeiten“ ändert
   Name, Zeiten, Nachlauf, Plattformen, Zielgruppe, Notiz. Bei Zielgruppe
   Alle zeigt die Karte zusätzlich „Wer“ (neu, zurück, Wechsel) und
   „Zurück bezahlt“ (erste Jahreszahlung ohne Gratismonat). Der Schalter auf der Karte
   macht die Aktion sofort aus und wieder an.

9. **Während der Aktion täglich ansehen.** Die Karte warnt selbst, wenn
   Apple-Codes knapp sind (unter 200 frei) oder leer, bei Play-Leck und wenn
   Android-Versuche ein Update brauchten. Per SQL:
   - `app/supabase/abfragen/aktion-c50-vorrat.sql`: freie Apple-Codes c50.
     Knapp: neue Charge in App Store Connect, wieder mit `--stufe c50`
     einspielen.
   - `app/supabase/abfragen/aktion-leck.sql`: Käufe von `creator-50` ohne
     eingelösten Code (letzte 24 h). Hohe Zahl: Angebot in der Play Console
     pausieren.
   - `app/supabase/abfragen/aktion-stand.sql`: Stand wie auf der Seite.

10. **Notfall per SQL.**

    ```sql
    -- Code sofort aus
    update public.affiliates set aktiv = false, aktiv_seit = now() where code = 'BLACKFRIDAY';
    ```

11. **Nach dem Ende** (Ende plus Nachlauf): Play-Angebot `creator-50-sofort`
    deaktivieren. `fin365-c50` und `creator-50` nur, wenn kein Creator und
    keine Aktion mit 50 % mehr aktiv ist (sie tragen auch c50 und den
    Rückfall von d50). Laufende Abos behalten 24,99 €
    fürs erste Jahr. Wer eingelöst und nicht gekauft hat, ist danach frei für
    einen Creator-Code oder die nächste Aktion.

Durchgestrichen stehen 49,99 €. Das muss der niedrigste Preis der letzten 30
Tage bleiben (Preisangabe), den Grundpreis vor Black Friday also nicht senken.

## Sofort zahlen (174)

Neu auf der Seite: Kästchen **Gratismonat** beim Anlegen und Bearbeiten, je
Creator und je Aktion. Vorgabe an, also wie bisher 30 Tage gratis. Aus heißt:
Das Jahresabo wird ohne Gratismonat sofort bezahlt, der Monat behält seinen
Gratismonat. Geht bei 33 und 50 % (vier Stufen: c33, d33, c50, d50). Die
Provision eines sofort bezahlten Jahres ist erst 15 Tage
nach der Abbuchung auszahlbar (Haltefrist). Vertrag: `store/SOFORT-BAU.md`
(Zweig `sofort-zahlen`). Die Seite vor diesem Umbau liegt in
`~/Downloads/fin-dashboard-1111/`.

- Das Kästchen erscheint erst, wenn 174 eingespielt ist (die Daten tragen
  dann `gratismonat`). Vorher sieht die Seite aus wie 1111.
- Umlegen wirkt nur auf neue Einlösungen. Wer schon zugeordnet ist, behält,
  was er beim Einlösen bekam.
- Sofort zahlt nur, wer mit der neuen App (iOS 339, AAB 88) einlöst und ein
  neues Store-Konto hat. Ältere Apps und fehlende d33- bzw. d50-Codes oder
  Play-Angebote geben bei einem Creator ohne Gratismonat weiter den
  Gratismonat (c33 bzw. c50), nie einen falschen Preis. Das gilt auch für
  schon Zugeordnete, wenn d33 bzw. d50 später leer oder abgelaufen ist: Sie
  bekommen den Rabatt mit Gratismonat, die Provision bleibt 15 Tage gehalten.
  Bei 50 % auf iOS nur, solange c50-Codes frei sind. Sind c50 und d50 leer,
  gibt es dort keinen Rabatt mehr (wie 162 bei leerem c50).
- Aktion ohne Gratismonat: sofort nur für neue Konten. Abgelaufene und der
  Wechsel laufen wie bisher, auch wenn jemand erst als neues Konto
  eingelöst hat und danach monatlich zahlt oder abgelaufen ist.
- Karte und Zeile zeigen „Ohne Gratismonat“. Ist Provision gehalten, steht auf
  der Karte „Gehalten X bis Datum“ (das früheste Datum, an dem etwas frei
  wird), die Kachel Offen nennt den gehaltenen Teil.
- **Auszahlung buchen:** „Fällig“ zählt nur Freies, darunter „Davon
  gehalten“. Nach Ablauf der 15 Tage zählt es beim nächsten Buchen mit.
  Wurde in der Frist erstattet, aber erst im Folgemonat, wartet die
  Provision auf ihren Abzug: Sie zählt erst, wenn du bis zu diesem Monat
  buchst, dann beide zusammen.
- Emi bleibt an. BLACKFRIDAY bleibt an, lässt sich jetzt aber umstellen.
  Einen Creator oder eine Aktion nur auf dein Wort umstellen.

Reihenfolge, nichts davon ist eingespielt oder deployt:

1. **174 einspielen** (nach 148, 150 und 162). SQL-Editor, ganzen Inhalt von
   `app/supabase/174_sofort_zahlen.sql` einfügen, einmal ausführen. Bricht der
   Selbsttest ab, ist nichts passiert. Wird 148, 150 oder 162 später noch
   einmal eingespielt, sofort danach 174; dazwischen keine Auszahlung buchen.
2. **Functions deployen, nur nach deinem Go.** Im Ordner `app`:

   ```sh
   supabase functions deploy abo --no-verify-jwt
   supabase functions deploy dashboard --no-verify-jwt
   ```

   Vor 174 sagt die Seite beim Umstellen „Erst 174 einspielen.“, vor dem
   Deploy „Die Function kennt das noch nicht. Erst deployen.“
3. **Store:** Apple-Kampagnen `fin365-d33` und `fin365-d50` mit eigenen
   Codes, Play-Angebote `creator-33-sofort` und `creator-50-sofort-neu`,
   dazu weitere `fin365-c50`-Codes mit späterem Ablauf, nach
   `store/SOFORT-STORE-SCHRITTE.md` (legt Claude an). `creator-50-sofort`
   bleibt das Black-Friday-Angebot.
4. **Seite hochladen.** `zahlen-67fdcdcee0fb.html` aus diesem Ordner auf
   fin-tracker.de unter demselben Pfad ersetzen.
5. **Umstellen.** Im Creator „Details“, unten „Bearbeiten“, Haken
   „Gratismonat“ weg, Speichern. Die Seite fragt vorher nach. Zurück geht
   genauso. Bei 50 % erst, wenn Schritt 3 ganz steht: d50-Codes, weitere
   c50-Codes und `creator-50-sofort-neu`.

Scheitert beim Anlegen nur das Umstellen, steht „Angelegt, Gratismonat bleibt
an. Nochmal umstellen.“: Der Creator ist da, mit Gratismonat; dann wie in
Schritt 5 umstellen.

Notfall per SQL:

```sql
-- Gratismonat wieder an (nur neue Einloesungen)
update public.affiliates set gratismonat = true where code = 'CODE';
```
