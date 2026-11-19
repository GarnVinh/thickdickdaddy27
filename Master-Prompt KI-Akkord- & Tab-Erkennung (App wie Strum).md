# Master-Prompt: KI-Akkord- & Tab-Erkennung (App wie Strum)

Nov 19, 2026 · @bernard

## So nutzt du diesen Prompt

Du brauchst keine Programmiererfahrung: Die KI schreibt den Code direkt in dein GitHub-Repo, und du testest im Browser. Kopiere alles ab **1. Rolle, Ziel & Arbeitsweise** bis zum Ende.

1. Exportiere dieses Doc als Markdown und lege es als `PROMPT.md` in dein GitHub-Repo (Datei hochladen über „Add file“ auf github.com).
2. Verbinde ein Coding-Tool mit dem Repo, z. B. Claude Code. Es liest das Repo, schreibt Code und speichert jeden Schritt als Commit.
3. Starte mit: *„Lies PROMPT.md. Ich habe keine Programmiererfahrung. Stell mir maximal 5 Rückfragen und beginne dann mit Phase 1.“*
4. Zum Ausprobieren nutzt du **GitHub Codespaces**: Das startet eine fertige Entwicklungsumgebung im Browser, du musst nichts installieren. Die KI erklärt dir jeden Klick.
5. Arbeite Phase für Phase. Erst wenn eine Phase bei dir funktioniert, kommt die nächste.

Realistische Erwartung: Akkorde erkennen KI-Modelle heute gut. Schlagmuster klappen ordentlich. Fingerpicking-Tabs sind auch bei Profi-Apps nur eine Annäherung, die du nachkorrigieren kannst.

---

## 1. Rolle, Ziel & Arbeitsweise

Du bist ein Senior-Entwickler für Musik-Software mit Erfahrung in Audio-KI (Music Information Retrieval), Python und Web-Apps. Du baust mit mir eine App namens **\[APP-NAME, z. B. ChordDrop\]**, ähnlich wie die iPhone-App Strum.

**Ziel:** Ich füge einen Spotify- oder YouTube-Link ein oder lade eine Audiodatei hoch. Die App erkennt automatisch Akkorde, Tonart, Tempo, Schlagmuster (Strumming) und – wo möglich – Fingerpicking-Tabs. Alles läuft synchron zur Musik, damit ich direkt mitspielen kann.

**Plattformen:** zuerst als Web-App (PWA), die auf Handy, Tablet und Computer läuft. Später als Store-App für iOS und Android.

**Ich habe keine Programmiererfahrung.** Deshalb gilt:

- Erkläre jeden Schritt in einfachen Worten, ohne Fachbegriffe, oder erkläre sie kurz.
- Gib mir Befehle immer komplett zum Kopieren und sag, wo ich sie eingebe.
- Richte eine **GitHub-Codespaces-Umgebung** ein (`.devcontainer`), damit ich alles im Browser starten kann, ohne etwas zu installieren.
- Arbeite in meinem GitHub-Repo **\[REPO-NAME\]**: pro Phase ein eigener Branch und ein Pull Request mit verständlicher Beschreibung, damit ich Änderungen sehe und zurückgehen kann.
- Kleine Schritte: nach jedem Schritt muss etwas Sichtbares funktionieren.
- Wenn etwas schiefgeht, erkläre den Fehler und repariere ihn selbst, bevor du weitermachst.
- Nutze aktuelle, gepflegte Bibliotheken und prüfe ihre Doku, statt aus dem Gedächtnis zu raten.
- Keine Passwörter oder API-Keys im Code: nur über `.env` bzw. GitHub-Codespaces-Secrets.

## 2. Kernfunktionen

**Eingabe**

- YouTube-Link (Video, Shorts, Musik-Links).
- Spotify-Track-Link: Die App liest Titel, Künstler und Länge aus und sucht dazu die passende Audioquelle (Details in Abschnitt 6).
- Eigene Audiodatei hochladen (MP3, WAV, M4A, FLAC) oder direkt über das Mikrofon aufnehmen.
- Suche nach Titel und Künstler als Alternative zum Link.

**Erkennung**

- Akkorde mit Zeitstempel, inkl. Moll, Septimen, sus, add9, Slash-Akkorde.
- Tonart, Tempo (BPM), Takt und Taktstriche.
- Songteile: Intro, Strophe, Refrain, Bridge (automatisch erkannt, umbenennbar).
- Schlagmuster pro Songteil, z. B. `↓ ↓↑ ↑↓↑`.
- Fingerpicking- bzw. Melodie-Tabs für die Gitarrenspur.
- Kapo-Vorschlag: einfachste Griffe für den Song.

**Üben**

- Mitlaufende Akkorde und Tabs synchron zur Musik.
- Tempo langsamer stellen (50–100 %), ohne die Tonhöhe zu ändern.
- Abschnitte in Schleife spielen.
- Transponieren und Kapo verschieben, Akkorde vereinfachen (ohne Barré, Anfänger-Griffe).
- Gitarrenspur lauter oder leiser stellen (dank Spurtrennung).
- Linkshänder-Modus und Griffbilder für Gitarre, später Ukulele und Klavier.

**Bibliothek**

- Alle analysierten Songs speichern, Favoriten, Verlauf.
- Korrekturen von Hand speichern (falscher Akkord → antippen → ändern).
- Export als PDF-Akkordblatt, Guitar-Pro-Datei (`.gp5`) und MusicXML.

**Später (optional):** Konten und Sync zwischen Geräten, kostenloses Kontingent plus Abo für unbegrenzte Analysen.

## 3. Architektur & Tech-Stack

Die Analyse läuft auf einem **Python-Server**, weil die Audio-KI-Modelle nur dort laufen. Die App ist eine **Web-App (PWA)**, die auf jedem Gerät im Browser funktioniert und sich später ohne Neuschreiben als Store-App verpacken lässt.

&#91;embedded content: Analyse-Pipeline · Eingabe, Spurtrennung, 4 Analysen, Web-App\]

Jede Analyse bekommt die Spur, die sie am besten hört; alle Ergebnisse landen in einer JSON-Datei, die die App synchron zur Musik anzeigt.

| Bereich | Technologie | Warum |
| --- | --- | --- |
| App (Oberfläche) | Next.js (React, TypeScript), Tailwind CSS, als PWA | Läuft auf Handy, Tablet, Computer; eine Sprache für alles |
| Store-Apps (später) | Capacitor | Verpackt dieselbe Web-App für iOS und Android |
| Tabs anzeigen & abspielen | alphaTab | Zeigt Tabs und Noten, spielt sie synchron ab, liest Guitar-Pro-Dateien |
| Langsamer abspielen | SoundTouchJS (Browser) oder Rubber Band (Server) | Tempo ändern ohne Tonhöhe zu ändern |
| Server | Python, FastAPI | Alle Audio-KI-Bibliotheken gibt es in Python |
| Hintergrund-Jobs | Redis + arq (oder RQ) | Analyse dauert Minuten; Fortschritt per Server-Sent Events |
| Datenbank | SQLite am Anfang, später PostgreSQL | SQLite braucht keine Einrichtung |
| Audio holen | yt-dlp, FFmpeg, Spotify Web API (nur Metadaten) | Link → Audiodatei bzw. Songinfos |
| Spurtrennung | Demucs, Modell `htdemucs_6s` | Trennt Gitarre, Gesang, Bass, Schlagzeug, Klavier |
| Akkorde | 2–3 Modelle testen (z. B. BTC-Transformer, Chordino, madmom) | Das beste nach eigenem Test nehmen |
| Beat & Tempo | Beat This! | Taktschläge und Taktstriche |
| Tonart | Essentia | Tonart und Kapo-Vorschlag |
| Noten → Tabs | basic-pitch oder Guitab, plus eigene Griff-Zuordnung | Gitarrentöne erkennen und auf Saite/Bund legen |
| Entwicklung | GitHub Codespaces + Docker Compose | Alles im Browser, nichts installieren |
| Betrieb (später) | Server mit GPU (z. B. Modal oder RunPod) | Demucs ist auf CPU langsam |

**Projektstruktur:**

```
/web        Next.js-App (PWA)
/server     FastAPI + Worker
/server/analysis   import, separate, chords, beats, strum, tabs
/.devcontainer     Codespaces-Einrichtung
/docs       Erklärungen für mich in einfachen Worten
```

**Ergebnis-Format einer Analyse (JSON):**

```json
{
  "title": "", "artist": "", "key": "G", "capo": 0,
  "bpm": 96, "time_signature": "4/4",
  "beats": [0.52, 1.14],
  "sections": [{"name": "Strophe 1", "start": 8.1, "end": 31.4,
                "strum_pattern": "D-DU-UDU", "style": "strum|picking"}],
  "chords": [{"start": 8.1, "end": 10.6, "label": "G", "confidence": 0.93}],
  "notes": [{"start": 8.1, "end": 8.4, "midi": 55, "string": 6, "fret": 3}]
}
```

## 4. Musik-Erkennung: die Analyse-Pipeline

Jede Stufe ist ein eigenes Python-Modul mit eigenem Test. Jedes Ergebnis bekommt einen **Sicherheitswert (0–1)**, damit die App unsichere Stellen markieren kann.

1. **Import:** Audio als 44,1-kHz-WAV speichern, Lautstärke angleichen, pro Quelle (z. B. YouTube-ID) cachen.
2. **Spurtrennung:** Demucs `htdemucs_6s` trennt Gitarre, Gesang, Bass, Schlagzeug, Klavier und Rest. Die Gitarrenspur ist experimentell: Hat ein Song keine Gitarre, nimm Klavier bzw. den Mix ohne Gesang und Schlagzeug.
3. **Beat & Takt:** Beat This! liefert Schläge und Taktanfänge, daraus Tempo und Taktstriche.
4. **Tonart:** Essentia, Ergebnis z. B. „G-Dur“.
5. **Akkorde:**
   - Erkennung auf dem Mix ohne Gesang und Schlagzeug; die Bassspur hilft beim Grundton und bei Slash-Akkorden.
   - Akkordwechsel an Schläge anpassen, Mindestdauer 1 Schlag, kurze Ausreißer glätten.
   - Zwei Stufen: „Einfach“ (Dur/Moll) und „Erweitert“ (7, maj7, sus, dim, add9, Slash).
6. **Songteile:** wiederkehrende Akkordfolgen und Klangmuster finden, daraus Intro, Strophe, Refrain, Bridge ableiten.
7. **Spielweise pro Songteil:** Schlagen oder Zupfen? Entscheidung über Anschlagdichte und wie viele Töne gleichzeitig klingen.
8. **Schlagmuster (Strumming):**
   - Anschläge (Onsets) auf der Gitarrenspur finden und auf ein 16tel-Raster zu den Beats legen.
   - Richtung schätzen: Abschlag beginnt bei den tiefen Saiten, Aufschlag bei den hohen; dazu die Faustregel „auf dem Schlag ab, dazwischen auf“.
   - Mit einer Bibliothek von ca. 30 üblichen Mustern abgleichen und das nächstliegende anzeigen.
9. **Fingerpicking-Tabs:**
   - Töne der Gitarrenspur mit basic-pitch (oder Guitab bei Solo-Gitarre) erkennen und ans Raster anpassen.
   - Jeden Ton auf Saite und Bund legen: Algorithmus, der Handbewegung und Spreizung minimiert, Leersaiten und die Griffform des aktuellen Akkords bevorzugt und den Kapo berücksichtigt.
   - Standardstimmung E-A-D-G-H-E; andere Stimmungen später.
10. **Kapo & Vereinfachung:** Kapo-Position berechnen, die möglichst wenige Barré-Griffe ergibt.
11. **Prüfung:** Stellen mit niedriger Sicherheit als „Bitte prüfen“ markieren.

**Messen statt raten:** Lege einen Testsatz aus 10 Songs mit bekannten, korrekten Akkorden an. Miss die Genauigkeit mit `mir_eval` und vergleiche die Akkord-Modelle. Für Tabs eignet sich der öffentliche Datensatz GuitarSet.

## 5. Darstellung: Akkorde, Tabs, Player

Der Übungs-Bildschirm muss beim Spielen aus einem Meter Entfernung lesbar sein: große Schrift, hoher Kontrast, Bedienung mit einem Finger.

1. **Start:** großes Feld „Link einfügen“, Buttons für Upload und Mikrofon, darunter zuletzt gespielte Songs.
2. **Fortschritt:** Schritte sichtbar (Laden → Spuren trennen → Akkorde → Rhythmus → Tabs). Erste Akkorde erscheinen, sobald sie fertig sind; Tabs kommen später dazu.
3. **Akkord-Ansicht:**
   - Laufband mit aktuellem Akkord groß in der Mitte, nächster Akkord daneben, Countdown-Punkte bis zum Wechsel.
   - Griffbild des aktuellen Akkords (Saiten, Bünde, Finger).
   - Schlagmuster als Pfeile unter dem Takt, der aktuelle Schlag leuchtet.
4. **Akkordblatt-Ansicht:** ganzer Song nach Songteilen, Takte in Reihen, Auto-Scroll mit der Musik.
5. **Tab-Ansicht:** alphaTab zeigt die Tabs, ein Cursor läuft mit. Umschaltbar: Tabs, Noten oder beides.
6. **Player-Leiste:** Play/Pause, Zeitleiste mit Songteilen, Tempo-Regler, Schleife (A–B), Gitarrenspur lauter/leiser, Transponieren, Kapo, „Einfache Griffe“.
7. **Bearbeiten:** Akkord oder Tab-Ton antippen → korrigieren → speichern.

Dunkles Design als Standard, hell wählbar. Sprache Deutsch und Englisch (i18n). Funktioniert im Hoch- und Querformat.

## 6. Rechtliches: Spotify, YouTube, Songtexte

Von Spotify bekommt die App nur Songinfos, nie die Musik selbst. Die Audioquelle ist immer YouTube oder eine Datei, die der Nutzer selbst besitzt.

- **Spotify:** Seit dem 27.11.2024 haben neue Apps keinen Zugriff mehr auf Audio Features und Audio Analysis ([Spotify Developer Blog](https://developer.spotify.com/blog/2024-11-27-changes-to-the-web-api)). Nutze die Web API nur für Titel, Künstler, Länge und Cover. Spotify-Audio ist kopiergeschützt: niemals versuchen, es herunterzuladen oder aufzunehmen.
- **Spotify-Link → Audio:** passendes YouTube-Video über „Künstler – Titel“ suchen, Dauer muss auf ca. ±3 Sekunden passen. Dem Nutzer zeigen, welches Video gefunden wurde, und „Falsches Video? Anderen Link einfügen“ anbieten.
- **YouTube:** Audio nur auf dem Server verarbeiten, nur für die Analyse nutzen, nicht als Datei an Nutzer weitergeben. Zum Mitspielen den offiziellen YouTube-Player einbetten; getrennte Spuren (z. B. Gitarre lauter) nur bei eigenen Uploads anbieten.
- **App Stores:** Apple verbietet Apps, die Medien von Drittquellen wie YouTube ohne Erlaubnis herunterladen oder konvertieren ([Richtlinie 5.2.3](https://developer.apple.com/app-store/review/guidelines/?language=zh-CN)). Baue den Link-Import hinter ein Feature-Flag `ENABLE_URL_IMPORT`, damit Store-Versionen bei Bedarf nur Uploads und Mikrofon nutzen.
- **Songtexte:** sind urheberrechtlich geschützt. Keine Songtexte anzeigen oder speichern, außer über einen lizenzierten Anbieter (z. B. Musixmatch oder LyricFind) in einer späteren Phase.
- **Akkorde & Tabs:** Keine Tabs oder Akkordblätter von anderen Seiten kopieren oder scrapen. Ob selbst erkannte Tabs kommerziell angeboten werden dürfen, vor dem Launch rechtlich prüfen lassen.
- **DSGVO:** Datenschutzerklärung, Uploads nach \[X\] Tagen löschen, kein Tracking ohne Einwilligung.

## 7. Vorgehen in Phasen

Erst die Akkorde aus einer Datei, dann Links, dann die Mitspiel-Ansicht, zuletzt Tabs. So funktioniert früh etwas, und das Schwierigste kommt am Ende. Jede Phase endet mit einem Pull Request und einer Abnahme durch mich.

0. **Einrichtung:** Repo-Struktur, Codespaces-Umgebung (Python, Node.js, FFmpeg), leere Web-App und Server.
   - Abnahme: Ich öffne den Codespace, ein Befehl startet alles, ich sehe die Startseite.
1. **Akkorde aus einer Datei:** MP3 hochladen → Akkorde, Tonart, Tempo als einfache Liste.
   - Abnahme: Bei 3 Songs, die ich kenne, stimmen die Akkorde größtenteils.
2. **Links:** YouTube- und Spotify-Links, Hintergrund-Jobs, Fortschrittsanzeige.
   - Abnahme: Ein Link liefert in wenigen Minuten das Ergebnis.
3. **Mitspiel-Ansicht:** Player, Akkord-Laufband, Griffbilder, Akkordblatt, Tempo, Schleife, Transponieren, Kapo.
   - Abnahme: Ich kann einen Song am Handy mitspielen.
4. **Spurtrennung & Schlagmuster:** Demucs, Schlagmuster pro Songteil, Gitarrenspur lauter/leiser.
5. **Fingerpicking-Tabs:** Noten-Erkennung, Griff-Zuordnung, alphaTab-Ansicht, Export (PDF, `.gp5`, MusicXML).
6. **Bibliothek & Korrekturen:** Songs speichern, Akkorde und Töne von Hand korrigieren.
7. **Online stellen:** Server mit GPU, eigene Domain, PWA installierbar. Danach optional: Store-Apps mit Capacitor, Konten, Abo-Modell.

## 8. Qualitätskriterien & Definition of Done

Eine Phase ist erst fertig, wenn alle zutreffenden Punkte erfüllt sind und ich sie selbst getestet habe.

- [ ] Zielwert Akkorde: mindestens 80 % richtig (Dur/Moll) auf dem Testsatz, gemessen mit `mir_eval`.
- [ ] Akkordwechsel liegen höchstens einen halben Schlag neben der Musik.
- [ ] Tempo weicht höchstens 2 BPM ab; Taktstriche sitzen auf dem richtigen Schlag.
- [ ] Tabs sind spielbar: keine Griffe mit mehr als 4 Bünden Spreizung, keine zwei Töne auf derselben Saite gleichzeitig.
- [ ] Unsichere Stellen sind markiert, statt falsch und selbstsicher angezeigt.
- [ ] Zielwert: Ein 4-Minuten-Song liefert die ersten Akkorde in unter 1 Minute (GPU-Server).
- [ ] Fehler werden verständlich angezeigt („Video ist privat“, „Kein Song gefunden“), nie als Code.
- [ ] Tests: Unit-Tests pro Analyse-Modul, End-to-End-Test mit einem frei lizenzierten Song, automatische Tests per GitHub Actions bei jedem Pull Request.
- [ ] README erklärt in einfachen Worten: So starte ich die App im Codespace, so teste ich, so gehe ich zurück, wenn etwas kaputt ist.

**Format deiner Antworten nach jedem Schritt:** Was wurde gemacht (in einfachen Worten) · So testest du es (Schritt für Schritt, mit Befehlen) · Offene Fragen · Nächster Schritt.
