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
