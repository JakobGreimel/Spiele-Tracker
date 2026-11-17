# Geschenk-App für Lotta und Paul – Stand 03.11.2026 (18:05)

(Übernommen aus dem Claude-Projekt „Spieleabend App“, Datei GESCHENK_APP_LOTTA_PAUL.md. Zeilenangaben beziehen sich auf App-Version 17.)

Neubau der Spieleabend-App (siehe `CLAUDE.md`) als Geburtstagsgeschenk für eine Freundin von Jakob (Lotta) und ihren Freund (Paul). Wichtig: Die App darf **nicht über Jakobs Konten laufen** (kein JGHosting-GitHub, kein Firebase-Projekt `spieletracker-app`).

## Entscheidungen
- Sync zwischen beiden Handys: **ja** (Firebase, aber eigenes Projekt der beiden)
- Hosting/Konten: **eigenes Konto für die beiden**; Jakob übergibt die Zugangsdaten als Teil des Geschenks.
- Spielernamen: **Lotta** und **Paul**
- Farben: von Claude passend gewählt (noch offen, nicht festgelegt)
- Startspiele: vorerst nur **Kniffel** (Wertung: Höchste Punkte)
- Anpassung: Namen, Farben, Startspiele; kein Begrüßungsbildschirm, kein eigenes Icon geplant
- Jakob hat gesagt: **noch nichts bauen**, Umbau erst auf sein Signal (wenn Wochenlimit wieder da ist)

## Stand der Konten (03.11.2026)
- Neues Google-Konto angelegt (dient für GitHub per „Mit Google fortfahren“ und für Firebase)
- Neues GitHub-Konto angelegt; Benutzername noch nicht an Claude übermittelt
- Neues Firebase-Projekt angelegt; Projekt-ID und Web-App-Config noch nicht an Claude übermittelt
- Noch zu tun in der Firebase-Konsole: Web-App hinzufügen (Config-Block holen), Authentication → E-Mail/Passwort aktivieren, Firestore Database erstellen (Europa, Produktionsmodus). Regeln trägt Claude später ein.
- Empfehlung an Jakob: im GitHub-Konto ein eigenes Passwort setzen (Settings → Password), Benutzername ohne Jakobs Namen (steht in der App-Adresse `benutzername.github.io/...`)

## Basis
- Quelle: Klon von `JGHosting/Spiele-Tracker` (main), Firebase SDK 12.19.0 gebündelt in `vendor/firebase.js`.
- Persönliche Stellen im Code, die ersetzt werden müssen:
  - `app.js` Z. 21–22: Spieler `Jakob`/`Emma` mit `#7c5cff` / `#ff5c8a`
  - `app.js` Z. 25ff: Startspiele (nur Kniffel behalten)
  - `app.js` Z. 926: Migrations-Zeile `Freundin` → `Emma` entfernen
  - `sync.js` Z. 47: `DEFAULT_PLAYERS` (Namen + Farben)
  - `sync.js` Z. 267: Kommentar mit „Emma“
  - `style.css` Z. 8–9: `--accent`, `--accent2` auf neue Farben
  - `config.js`: Firebase-Config von `spieletracker-app` ersetzen durch die neue Config (authDomain, projectId, storageBucket usw.)
  - `README.md`, `manifest.webmanifest`: auf Verweise zu Jakob/JGHosting prüfen
  - `sw.js` `VERSION` und `app.js` `APP_VERSION` beim Neubau zurücksetzen/hochzählen
- Bekannte Stolpersteine aus dem Original (siehe `CLAUDE.md`): Bundle nur in leerem Ordner bauen; iOS-Shell-Lösungen beibehalten.

## Nächste Schritte
1. Jakob schickt GitHub-Benutzername und Firebase-Web-App-Config und schaltet Auth (E-Mail/Passwort) und Firestore ein.
2. Code anpassen (Namen, Farben, Kniffel, neue Config, keine Verweise auf Jakob).
3. Schritt-für-Schritt-Anleitung für das neue Konto beilegen: GitHub-Repo + Pages, `firestore.rules` veröffentlichen, Config prüfen.
4. Als ZIP/Repo übergeben (Jakob lädt in das neue GitHub-Konto hoch), auf dem iPhone mit „Zum Home-Bildschirm“ testen.
5. Nach Möglichkeit Test mit Lotta und Paul vor dem Geburtstag; Datum des Geburtstags ist noch nicht bekannt.
