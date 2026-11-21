# Spieleabend-App (privat: Jakob & Emma)

## Zweck und Zielgruppe
- Web-App (PWA) zum Tracken von Spieleabenden zu zweit: Siege, Punkte, Statistiken.
- Nutzer: Jakob und seine Freundin Emma, am iPhone (Safari → „Zum Home-Bildschirm“). Optional Synchronisierung zwischen beiden Handys.
- Eine Nachbau-Anleitung samt Prompt für andere Paare existiert als privates Claude-Artifact: https://claude.ai/artifact/ENENi3Jz1VkcSD5PPDhkYA
- Geplant, noch nicht gebaut: Neubau als Geburtstagsgeschenk für Lotta und Paul (eigene Konten, nicht über Jakob) → `docs/geschenk-app-lotta-paul.md`.

## Technik
- Reines HTML/CSS/JavaScript, **kein Framework, kein Build für die App selbst**.
- Firebase JS SDK **12.19.0** (Auth E-Mail/Passwort + Firestore), lokal gebündelt in `vendor/firebase.js` (esbuild, ESM, minifiziert). Kein CDN.
- Hosting: GitHub Pages (Repo `JakobGreimel/Spiele-Tracker`, bis 11/2026 `JGHosting`, Branch `main`, root). Live: https://jakobgreimel.github.io/Spiele-Tracker/ (alte Adresse `jghosting.github.io` liefert 404)
- Firebase-Projekt `spieletracker-app` (Spark-Tarif). Config in `config.js` (nicht geheim, identifiziert nur das Projekt).
- Zielplattform: iPhone-PWA (Hauptziel); laut README auch Android/Chrome.
- UI-Sprache: Deutsch.

## Ordnerstruktur
- `index.html` – App-Shell: Header (Sync-Wolke, „+“), `<main id="view">`, Tab-Leiste, Bottom Sheet, Dialog, Toast. Lädt `config.js`, `app.js`, `sync.js` (Modul).
- `app.js` – gesamte App-Logik und UI (Tabs Spiele / Verlauf / Statistik / Mehr), lokaler Modus, Brücke zur Sync-Schicht. Enthält `APP_VERSION`.
- `sync.js` – Firebase Auth + Firestore, stellt `window.Cloud` bereit (inkl. `Cloud.ready`-Promise). Datenmodell steht als Kommentar am Dateianfang.
- `vendor/firebase.js` – gebündeltes SDK (Build-Artefakt, nicht von Hand ändern); `vendor/firebase-entry.js` – Einstieg für den Bundle-Build.
- `config.js` – `window.FIREBASE_CONFIG` (`null` = nur lokal).
- `firestore.rules` – Sicherheitsregeln (werden von Hand in der Firebase-Konsole veröffentlicht).
- `sw.js` – Service Worker, `VERSION = 'spieleabend-vN'`.
- `style.css`, `manifest.webmanifest`, `icon.svg`, `icon-180/192/512.png`, `.nojekyll`, `README.md` (inkl. Firebase-Einrichtung).
- `docs/geschenk-app-lotta-paul.md` – Plan für die Geschenk-App.

## Befehle
- App ändern: Dateien bearbeiten, nichts bauen. Deploy = Push auf `main` (live nach 1–2 Min.).
- **Bei jeder Änderung `APP_VERSION` (app.js) und `VERSION` (sw.js) gemeinsam hochzählen**, sonst lädt die installierte App das Update nicht.
- Firebase-Bundle neu bauen (nur bei SDK-Update; **immer in einem leeren Ordner**, sonst doppelte Firebase-Kerne → „Component auth has not been registered yet“):
  `npm i firebase@12.19.0 esbuild && npx esbuild vendor/firebase-entry.js --bundle --format=esm --minify --outfile=vendor/firebase.js`
  Danach im Browser prüfen, dass `initializeAuth` und `initializeFirestore` laufen.
- Keine automatischen Tests im Repo. Entscheidung (16.11.2026): Tests laufen weiterhin auf den Handys, keine neue Testumgebung.
- Lokaler Start (Vorschlag, nicht offiziell festgelegt): `python3 -m http.server 8080` im Projektordner, dann `http://localhost:8080`.

## Konventionen und Designregeln
- Spielerfarben: Lila `#7c5cff` (Jakob/p1), Pink `#ff5c8a` (Emma/p2); Hintergrund `#0d0d14`; automatisch dunkel/hell (`prefers-color-scheme`).
- Eigene Dialoge, Bottom Sheets und Toasts statt `alert/confirm`. Bedienung wie eine native App: kein Zoom, kein Markieren, kein Kontextmenü, keine Scrollbalken, Wischen zum Schließen.
- App-Shell: `body` ist `position: fixed`, nur `<main>` scrollt; Header und Tab-Leiste bleiben fest.
- Meta `apple-mobile-web-app-status-bar-style` = `black`.
- Datumsfeld: `appearance: none; width: 100%; min-width: 0`.
- Spielernamen werden bei jedem Tastendruck gespeichert. Versteckter Toast: `opacity: 0; visibility: hidden`.
- Auto-Update: Service Worker network-first mit `cache: 'no-cache'`, `updateViaCache: 'none'`, `reg.update()` bei Start und `visibilitychange`, Reload bei `controllerchange`. Der SW fängt Anfragen an fremde Domains (Firebase) nicht ab.
- Lokale Daten: `localStorage`-Key `spieleabend.v1`; im Konto-Modus bleiben sie als Sicherung unangetastet.
- Wertungsarten: Höchste Punkte / Wenigste Punkte / Verlierer bekommt Minuspunkte (Option: Gewinner bekommt sie als Plus) / Nur Sieg-Niederlage. Wertung ist gesperrt, sobald Partien existieren. Startspiele: Kniffel, Qwixx, Ganz schön clever, Rummikub.

## Sync-Architektur (Kurzfassung)
- `users/{uid}` Profil, `users/{uid}/games|matches` privat; `groups/{gid}` (+ `games|matches`) geteilt; `invites/{CODE}` (6 Zeichen, 7 Tage gültig).
- Neue Spiele sind privat (🔒); über ✎ → Sichtbarkeit teilbar (👥), Partien wandern mit.
- Erstanmeldung übernimmt lokale Daten (IDs bleiben, idempotent). Leere private Duplikate geteilter Spiele werden nur nach Server-Bestätigung entfernt, nie mit Partien.
- Offline: Firestore `persistentLocalCache` (IndexedDB). Status-Wolke: grün synchron / orange ausstehend / grau offline.
- Konflikte: jede Partie ein eigenes Dokument; Spiel-Änderungen last-write-wins; Partien gelöschter Spiele werden ausgeblendet.
- Rules: privat nur selbst; Gruppe nur Mitglieder; Beitritt nur mit gültigem Code < 7 Tage und nur freier Slot; Mitgliederliste nur durch eigenen Austritt änderbar; Invites nur per `get`, kein `list`.

## Entscheidungen und bewusst nicht
- Firebase statt Supabase (Supabase-Free pausiert nach 7 Tagen Inaktivität), kein eigenes Backend, kein GitHub als Datenbank.
- Kein Framework, kein Build für die App.
- Kein `backdrop-filter` (verursachte Blur-Probleme auf iOS); `black-translucent` nicht verwenden (Blur im Header, leerer Balken unten).
- Bundle nie in einem Ordner mit Resten alter Firebase-Versionen bauen.
- Die Geschenk-App für Lotta und Paul darf **nicht** über Jakobs Konten laufen (kein `JGHosting`-GitHub, kein Firebase-Projekt `spieletracker-app`).

## Aktueller Stand (App-Version 17, Commit `1116d94`, Stand 03.11.2026)
Funktioniert: Spiele, Partien, Verlauf (nach Monat, Filter), Statistik (Siegquoten, Serien, Form, Ø/Bestwert/Bilanz, Punkteverlauf), Mehr (Spieler, Konto, Gruppe, JSON-Backup, Versionsnummer), lokaler Modus, Firebase-Login.

Behobene Fehler (nicht wiederholen):
1. „Cloud.signUp is not a function“ → `Cloud.ready`-Promise, App wartet bis 20 s und zeigt echte Startfehler (v15).
2. „Component auth has not been registered yet“ → doppelte `@firebase/app` im Bundle, Bundle sauber neu gebaut (v16).
3. Einladungscode/Gruppe verschwand sofort → `groupId` erst nach Commit ins Profil, Listener verbinden neu, Gruppe nur entfernen, wenn Server mehrfach bestätigt (v17).

Am 16.11.2026 bestätigt Jakob: kein neuerer Stand als `1116d94`, Testergebnisse „passt alles“ (Details zu Gruppe/Code/Beitritt/Teilen nicht einzeln genannt).
Vermutlich verwaiste Gruppe auf dem Server vom fehlgeschlagenen ersten Versuch (harmlos).

## Offene Aufgaben (nach Priorität)
1. Geschenk-App für Lotta und Paul bauen, sobald Jakob das Signal gibt: eigener Ordner `../Spieleabend_Lotta` (Plan in `docs/geschenk-app-lotta-paul.md`). Nicht in diesem Ordner umbauen.
2. Entscheiden, ob das Repo für andere zum Forken freigegeben wird (Alternative: Nachbau-Prompt).

## Entscheidungen vom 16.11.2026
- Git-Remote `origin` bleibt (GitHub Pages deployt von dort). Nicht entfernen.
- Geschenk-App liegt in `../Spieleabend_Lotta` (`~/Software_Entwicklung/Spieleabend/`), nicht hier.
- Keine neue Testumgebung; getestet wird auf den Handys.

## Zu klären
- Firestore-Region in der Firebase-Konsole von `spieletracker-app` ist nicht dokumentiert (README empfiehlt `eur3`/`europe-west3`). Autorisierte Domains: `jakobgreimel.github.io` eingetragen (11/2026); ob `jghosting.github.io` noch drinsteht, ist ungeprüft.
- Geburtstag von Lotta/Paul ist unbekannt.
- Lokaler Start per `http.server` ist ein Vorschlag, noch nicht bestätigt.

## Ort (Stand 16.11.2026)
- Arbeitsordner: `~/Software_Entwicklung/Spieleabend/Spieleabend_Privat` (lokal, nicht mehr in iCloud Drive).
- Ältere Pfadangaben in dieser Datei oder in der Doku (iCloud `Developer/…`, `Projekte/…`, `~/Developer/…`) sind überholt; es gilt dieser Ordner.
