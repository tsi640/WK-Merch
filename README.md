# Waldkirche · Merch-Abstimmung

Eine touchoptimierte Abstimmungs-Web-App für gemeinsame iPads: eine Ja/Nein-Frage zum kleinen Waldkirche-Stick, je 2–3 **reine Designideen** für Hirte/Schaf und Baum/Wurzeln. React + TypeScript + Vite, GitHub Pages, Supabase (Postgres, Auth, Storage). Alle drei Antworten werden in **einem** Stimmzettel gespeichert. Keine Besucher-Accounts, keine E-Mail-Erfassung bei Abstimmungen.

## 1 · Supabase einrichten (einmalig)

1. Das Supabase-Projekt `wrmwzesptikclqzbzwte` ist bereits in `.env.example` und `.env.production` hinterlegt. **Es wurden dadurch noch keine Tabellen angelegt oder Änderungen in Supabase durchgeführt.**
2. Im **SQL Editor** die vollständige Datei [`supabase/schema.sql`](supabase/schema.sql) ausführen. Sie legt Tabellen, Regeln, Bild-Bucket und die transaktionale Abstimmungsfunktion an.
3. In **Authentication → Users → Add user** für das Merch-Team einen Benutzer mit E-Mail/Passwort erstellen und die E-Mail als bestätigt markieren. Falls Supabase stattdessen eine Einladung verschickt, die Einladung zuerst annehmen.
4. Im **SQL Editor** folgenden Befehl ausführen. Die Adresse durch die E-Mail des gerade erstellten Benutzers ersetzen:

   ```sql
   insert into public.voting_admins (user_id)
   select id from auth.users where email = 'DEINE-ADMIN-EMAIL@BEISPIEL.DE';
   ```

   Für weitere Team-Mitglieder dieselben Schritte wiederholen. **Keine Passwörter und keinen service_role/secret key in Dateien oder GitHub speichern.**
5. Projekt-URL und **öffentlicher** Publishable Key sind bereits konfiguriert. Der Schlüssel wird in der fertigen Website sichtbar sein; die eigentliche Zugriffsprüfung erfolgt durch die RLS-Regeln in `schema.sql`. Niemals einen Secret-/Service-Role-Key in `.env.production` eintragen.

## 2 · Lokal starten

Node.js 22 oder neuer wird empfohlen.

```bash
npm install
cp .env.example .env
```

Die öffentliche Projektkonfiguration ist bereits eingetragen. Anschließend:

```bash
npm run dev
```

Die angezeigte Adresse im Browser öffnen. Für ein iPad im selben WLAN die im Terminal angezeigte Netzwerkadresse benutzen und die Firewall des Entwicklungsrechners beachten. Die App nutzt stets die Supabase-Cloud-Datenbank.

## 3 · Motive einpflegen und Abstimmung freischalten

1. `#/admin` aufrufen (Link **Team** oben rechts), als freigeschalteter Benutzer anmelden.
2. Unter **Hirte & Schaf** und **Baum & Wurzeln** jeweils 2–3 Motive als PNG, JPEG oder WebP (maximal 8 MB) hochladen. Je Motiv einen klaren Titel und bei Bedarf einen kurzen Erklärungstext eintragen. **Keine Shirts oder erfundenen Platzhalterbilder erforderlich.**
3. Mit **Abstimmung öffnen** veröffentlichen. Erst dann sind die Motive im öffentlichen Fragebogen abrufbar. Die iPads öffnen die normale Startseite, ohne `#/admin`.
4. Das Team kann die Abstimmung jederzeit schließen und im Ergebnis-Tab die Zahlen ansehen und als CSV exportieren. Nach der ersten abgegebenen Stimme bleiben die Motive absichtlich gesperrt – ein nachträglicher Austausch würde das Ergebnis verfälschen.

**Kiosk-Ablauf:** Eine Person beantwortet alle drei Fragen, drückt „Stimme abgeben“ und sieht eine Bestätigung. Danach „Neue Abstimmung starten“ für die nächste Person tippen. Bei Verbindungsproblemen bleibt die Auswahl erhalten und der Sendeversuch kann mit derselben Stimmzettel-ID wiederholt werden; so wird ein doppeltes Speichern derselben Übertragung verhindert.

## 4 · Auf GitHub Pages veröffentlichen

1. Diesen Projektordner als GitHub-Repository hochladen. Der Branch heißt `main`. `.env` **nicht** committen (`.gitignore` greift). Die Workflow-Datei `.github/workflows/deploy.yml` ist bereits enthalten.
2. **Keine GitHub-Variablen erforderlich:** `.env.production` enthält bereits die öffentliche Supabase-URL und den Publishable Key. Die Datei darf zusammen mit dem Quellcode hochgeladen werden. Verwende dort niemals einen `service_role`-, Secret- oder anderen privaten Schlüssel. Vite schreibt `VITE_*`-Werte in die öffentliche Website.
3. Unter **Settings → Pages → Build and deployment → Source** „GitHub Actions“ einstellen.
4. Push nach `main`: der Workflow führt `npm install`, TypeScript-Prüfung, Vite-Build und Pages-Deployment aus. Die URL steht danach unter **Settings → Pages**. Durch relative Asset-Pfade und Hash-Navigation funktioniert sie auch unter `benutzer.github.io/repository/`.
5. Die veröffentlichte Website auf beiden iPads öffnen, eine Testabstimmung durchführen und die Ergebnisse im Team-Bereich kontrollieren. **Zum Zurücksetzen der Teststimmen vor der echten Abstimmung:** bei geschlossener Abstimmung im Supabase SQL Editor `truncate table public.ballots;` ausführen. Dieser Befehl löscht unwiderruflich alle Stimmen. Danach können bei Bedarf auch die Motive wieder bearbeitet werden.

### Testen

```bash
npm run test
npm run build
```

Die automatisierten Tests prüfen Vollständigkeit und Auszählung. Der Live-Test von Login, Upload, Schreiben und Berechtigungen benötigt eure eigenen Supabase-Zugangsdaten und ist erst nach Einrichtung möglich.

## Grenzen, die bewusst transparent bleiben

- **Vertrauensbasis:** Ein Mensch kann ohne Identifizierung absichtlich mehrmals abstimmen. Das wird nicht fälschlich als „eine Stimme pro Person technisch garantiert“ verkauft. Jede Übertragung besitzt jedoch eine UUID, die bei einem Verbindungsfehler nicht doppelt gezählt wird.
- **Kein Live-Synchronisieren offener iPad-Bildschirme:** Änderungen erscheinen beim erneuten Öffnen bzw. bei „Neue Abstimmung starten“. Der Server verhindert Stimmen nach Schließung, selbst wenn ein iPad noch die vorherige Seite anzeigt.
- **Hosting:** GitHub Pages veröffentlicht nur die Oberfläche. Ohne eigenes Supabase-Projekt, SQL-Schema und öffentliche Zugangsdaten können echte Stimmen weder gespeichert noch abgerufen werden. Die **öffentlichen** Verbindungsdaten sind enthalten; das SQL-Schema und das Team-Konto müssen im Supabase-Dashboard noch eingerichtet werden.
- **Datenschutz:** Der Stimmzettel enthält nur die Antworten, eine zufällige technische UUID und einen Zeitstempel; keine Namen oder E-Mail-Adressen. Supabase und GitHub können im Rahmen des technischen Betriebs eigene Serverprotokolle führen. Vor einer öffentlichen Nutzung die passende Datenschutzhinweis-Seite für eure Gemeinde ergänzen.
- **Einsatzzweck:** Eine einzige Abstimmungsrunde pro Datenbestand. Zum Start einer neuen Kollektion nach Sicherung der Ergebnisse die alten Stimmen und anschließend Motive bewusst in Supabase entfernen.

## Eine weiße Seite statt der App?

Die React-Anwendung benötigt einen Vite-Build; die Datei `index.html` darf **nicht direkt** lokal geöffnet oder als roher Quellcode per GitHub Pages „Deploy from a branch“ ausgeliefert werden. Unter **Settings → Pages → Build and deployment → Source** muss **GitHub Actions** ausgewählt sein. Der Workflow `.github/workflows/deploy.yml` muss im Repository enthalten sein (auf versteckte Ordner beim Upload achten). Im Tab **Actions** den Lauf „Website veröffentlichen“ prüfen. Nur die dort veröffentlichte Pages-URL aufrufen. Eine erfolgreiche Veröffentlichung allein ersetzt noch nicht das Supabase-SQL-Schema und die Motive im Adminbereich.
