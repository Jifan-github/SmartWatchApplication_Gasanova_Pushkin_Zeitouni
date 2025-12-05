# SmartWatchApplication_Gasanova_Pushkin_Zeitouni
Hier wird der Papierprotoyp mit der Figma AI umgesetzt'

# Link zum [Figma Prototyp](https://sheen-fond-12214453.figma.site/)


## Prompt:
'''
wir haben ein prototyp für eine Smartuhr App auf figma. nun wollen wir diese app entwickeln. Beispielsweise für eine Garmin uhr. Wie machen wir das? Auf was ist zu achten? Wie gehen wir da vor? Kannst du schritt für Schritt anleitung machen. Wenn du noch informationen brauchst sag bescheid.
'''

### ChatGPT:
Super — klasse, dass ihr schon ein Figma-Prototyp habt. Ich mache euch jetzt eine praktische, schritt-für-schritt-Anleitung, wie ihr daraus eine echte Garmin-App (Connect IQ) entwickelt — inklusive technischer Anforderungen, Best-Practices (Battery, UI) und Checklisten. Ich verlinke die wichtigsten Quellen an den Stellen, wo es sinnvoll ist.

# Kurzüberblick (in 1 Satz)

Garmin-Apps werden mit dem **Connect IQ**-Ökosystem entwickelt, in der Sprache **Monkey C**, mit dem SDK Manager + VS Code (Monkey C Extension). Ihr exportiert Assets aus Figma, passt das UI an begrenzte Uhren-Screens & Batterie-Constraints an, programmiert, testet im Emulator / auf Geräten und reicht die App im Connect IQ Store ein. ([developer.garmin.com][1])

---

# Schritt-für-Schritt Anleitung

## 1) Projektplanung & Entscheidungen (vor dem Coden)

1. **Zielgeräte festlegen**: Welche Garmin-Modelle wollt ihr unterstützen? (z. B. Venu, Fenix, Epix, Forerunner). Geräte unterscheiden sich in Auflösung, Touch/Buttons, Sensoren.
2. **App-Typ wählen**: Watch Face, App (activity/standalone), Widget, Data Field — das beeinflusst APIs & Lebenszyklus.
3. **Funktionalität & Datenfluss**: Benötigt ihr GPS, Herzfrequenz, Sensoren, Hintergrund-Laufen, Verbindung zum Smartphone/Internet oder nur lokale Anzeige?
4. **Non-functional**: Speicher-, CPU-, Batterie-Limits; Offline/Online; Datenschutz/Permissions; Release/Monetarisierung.
   ➡️ *Warum*: unterschiedliche Gerätetypen + App-Typen haben verschiedene APIs, Speicher- und Batterie-Auswirkungen. ([developer.garmin.com][2])

---

## 2) Entwicklungsumgebung einrichten

1. Installiert **Garmin SDK Manager** und ladet das aktuellste Connect IQ SDK über den SDK Manager herunter. ([developer.garmin.com][3])
2. Installiert **Visual Studio Code** + die **Monkey C Extension** (von Garmin). Die Extension hilft beim Projekt-Template, Build & Emulator. Die Extension benötigt Java JRE (1.8+) laut Dokumentation. ([developer.garmin.com][4])
3. Optional: Git einrichten, CI/CD (z. B. automatischer Build), Issue Tracker.

---

## 3) Design → Assets exportieren aus Figma

1. Exportiert Icons, Grafiken und einzelne UI-Elemente in passenden Größen (PNG, SVG). Achtet auf mehrere Auflösungen, weil Watches unterschiedliche Pixel-Dichten haben. Figma unterstützt Bulk-Export. ([Figma Hilfezentrum][5])
2. **Wichtig**: Vermeidet detailreiche Grafiken — kleine Bildschirme brauchen klare, hohe Kontrast-Icons und möglichst wenige Farben/Komplexität.
3. Legt ein Design-Spec an: Schriftgrößen (oder Bitmap-Schriften), Abstände, Interaktionsmuster (Button via Touch vs. Buttons).

---

## 4) UI/UX-Anpassungen für Wearables & Batterie-Optimierung

1. **Informationsdichte reduzieren**: nur das Nötigste zeigen; zu viele Updates (Sekunden, Live-HR sehr oft) killen die Batterie. ([developer.garmin.com][6])
2. **Refresh-Frequenz minimieren**: Aktualisiert nur, wenn nötig; gruppiert Updates.
3. **Animations-Sparsamkeit**: begrenzte oder keine Animationen für stromsparende Geräte.
4. **Fallbacks**: Wenn Sensoren nicht vorhanden sind, zeigt sinnvolle Alternativen.
5. Testet immer Akku-Impact (s. Schritt 8). ([developer.garmin.com][6])

---

## 5) Erstes Projekt & Prototyping in Monkey C

1. In VS Code: `Monkey C: Create New Project` → wählt passenden Project Type (Watchface/App/Widget/Datafield). (Die Extension führt euch durch.) ([developer.garmin.com][3])
2. Grundlegende Ordnerstruktur: `source/`, `resources/`, `manifest` etc. Schreibt modular (UI-Views, Models, Services).
3. Nutzt vorhandene Beispiel-Repos (Garmin hat Referenz-Apps auf GitHub) als Vorlage. ([GitHub][7])

---

## 6) Implementierung: APIs & Dinge, auf die zu achten ist

1. **Monkey C Sprache** — ähnlich zu JavaScript/C/Java, eigene Laufzeit. Lernt das Event-Model. ([developer.garmin.com][8])
2. **Geräte-APIs**: Sensor/Health, System Clock, GPS, Bluetooth, Storage — nutzt nur, was ihr wirklich braucht.
3. **Storage & Speicherplatz**: begrenzt — keine großen lokalen Datenmengen.
4. **App Lifecycle**: Widgets/Apps können suspendiert werden — speichert Zustand sauber.
5. **Permissions & Datenschutz**: Beim Zugriff auf Health-Daten oder GPS braucht ihr explizite Permission/Erklärung.
6. **Optimierungen**: Vermeidet häufige Timer mit kurzen Intervallen; bündelt I/O; nutz native Grafikroutinen sparsam.

---

## 7) Testen: Emulator und echtes Gerät

1. Startet im **Connect IQ Emulator** in VS Code (Device Target = euer gewünschtes Modell). Testet Auflösungen, Buttons, Touch-Interaktion. ([developer.garmin.com][3])
2. Testet auf mindestens einem echten Gerät — Performance & Battery sind im Emulator nicht 1:1.
3. Testfälle: UI Scaling, Low-Battery Szenario, No-Sensor Szenario, Airplane Mode, Background/Resume.
4. Messung: Protokolliert Energieverbrauch bei typischen Nutzungsfällen. Community-Tips und Garmin Support behandeln Batterieprobleme häufig. ([Garmin-Foren][9])

---

## 8) Profiling & Battery-Feintuning

1. Verwendet Logging, misst CPU/Ticks und beobachtet wie oft ihr Sensoren abfragt.
2. Reduziert Abfragen (z. B. HR alle 5–15s statt 1s, wenn möglich), vermeidet Web-Requests vom Gerät (teuer für Batterie).
3. Wenn eure App Background-Tasks braucht, erläutert klar im Design, warum — viele Apps, die im Hintergrund stark arbeiten, werden schlecht bewertet oder lehnen Battery-sensitive Geräte ab. ([developer.garmin.com][6])

---

## 9) Packaging & Einreichen in den Connect IQ Store

1. Prüft **App Review Guidelines** (Inhalte, Berechtigungen, Monetarisierung). ([developer.garmin.com][10])
2. Erstellt eine saubere Manifest-Datei, Icon, Screenshots (für Store) & Beschreibung.
3. Signiert und reicht die App über das Garmin Developer Portal / Store-Workflow ein. Achtet auf Datenschutz-Docs, falls Health-Daten genutzt werden.
4. Plant Zeit für Review & mögliche Nachbesserungen ein.

---

## 10) Post-Release: Monitoring & Updates

1. Sammelt Feedback/Crash-Reports, überwacht Batteriemeldungen im Support.
2. Releases: Bugfixes, Optimierungen, neue Geräte-Support.
3. Pflege eines Upgrade-Plans (API-Änderungen in SDK, neue Geräte).

---

# Checkliste (zum Abhaken)

* [ ] Zielgeräte Liste (Modelle & Screen-Specs)
* [ ] App-Typ (Watchface / App / Widget / DataField)
* [ ] Figma → exportierte Assets (größen + Formate)
* [ ] SDK Manager + VS Code + Monkey C Extension installiert. ([developer.garmin.com][3])
* [ ] Minimales MVP Feature Set definiert
* [ ] Batterie-Budget & Refresh-Frequenz definiert
* [ ] Testgeräte verfügbar (mind. ein echtes Gerät)
* [ ] Privacy/Permissions Plan (falls Health/GPS)
* [ ] Store-Listing Texte & Screenshots vorbereitet

---

# Häufige Fehler & Tipps

* **Zu viele Live-Updates** → Akku-Killer. Reduziert Update-Frequenz. ([developer.garmin.com][6])
* **Vergessen, verschiedene Bildschirmgrößen zu testen** → UI-Brüche.
* **Netzwerk-Abfragen direkt vom Gerät** → teuer; nach Möglichkeit via Smartphone/Companion App synchronisieren.
* **Keine Fallbacks für fehlende Sensoren** → Abstürze oder leere Views.

---

# Was ich von euch noch bräuchte, um ein konkretes Implementierungs-Plan / Tasks-Board zu erstellen

Wenn ihr möchtet, erstelle ich euch daraus ein detailliertes Sprint-Plan (Tasks + Zeitaufwand) oder ein technisches Spec-Dokument. Dazu brauche ich kurz diese Angaben (ein Satz pro Punkt reicht):

1. Welches(e) Garmin-Modell(e) wollt ihr priorisieren?
2. Welcher **App-Typ** (Watchface, App, Widget, DataField)?
3. Welche Haupt-Features (z. B. Schrittzähler, Trainingsmodus, Benachrichtigungen, Messung mit HR/GPS)?
4. Braucht ihr Backend / Server / Benutzerkonto?
5. Wer programmiert — internes Team oder externe Agentur? (falls externe: gewünschte Technologien/CI)

Wenn ihr mir diese 5 Punkte gebt, erstelle ich euch sofort (hier) ein konkretes Implementierungs-Roadmap + Taskliste.

---

Wenn ihr möchtet, kann ich auch:

* Ein Starter-Repository Template vorschlagen (Files/Folder Struktur) und Beispielcode für ein kleines Watchface.
* Eine Liste mit empfohlenen Testgeräten (Modelle) zusammenstellen.
* Ein Figma → Export Checklist-Dokument erstellen.

Möchtet ihr, dass ich direkt ein MVP-Taskboard (z. B. Tickets nach Priorität) erstelle? Dann schreibt kurz die 5 Punkte oben — ich leg sofort los.

[1]: https://developer.garmin.com/connect-iq/?utm_source=chatgpt.com "Connect IQ SDK"
[2]: https://developer.garmin.com/connect-iq/connect-iq-basics/?utm_source=chatgpt.com "Connect IQ Basics"
[3]: https://developer.garmin.com/connect-iq/sdk/?utm_source=chatgpt.com "Get the SDK | Connect IQ"
[4]: https://developer.garmin.com/connect-iq/reference-guides/visual-studio-code-extension/?utm_source=chatgpt.com "Monkey C Visual Studio Code Extension"
[5]: https://help.figma.com/hc/en-us/articles/360040028114-Export-from-Figma-Design?utm_source=chatgpt.com "Export from Figma Design"
[6]: https://developer.garmin.com/connect-iq/user-experience-guidelines/?utm_source=chatgpt.com "User Experience Guidelines"
[7]: https://github.com/garmin/connectiq-apps?utm_source=chatgpt.com "garmin/connectiq-apps: A collection of Connect IQ apps."
[8]: https://developer.garmin.com/connect-iq/monkey-c/?utm_source=chatgpt.com "Monkey C"
[9]: https://forums.garmin.com/developer/connect-iq/f/discussion/364926/any-leads-to-improve-battery-consumption?utm_source=chatgpt.com "Any leads to improve battery consumption - Connect IQ ..."
[10]: https://developer.garmin.com/connect-iq/reference-guides/?utm_source=chatgpt.com "Reference Guides"
