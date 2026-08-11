# Mobile App Notification Group

Ein wiederverwendbarer **Home-Assistant-Skript-Blueprint** für Benachrichtigungen an eine feste Gruppe von Geräten mit der Home Assistant Companion App.

Die Empfänger und Zustellungsoptionen werden einmal pro Skriptinstanz eingerichtet. Automationen müssen danach nur noch Titel, Nachricht und bei Bedarf Bestätigungsbuttons übergeben. Dadurch bleibt die komplette Empfänger- und Mobile-App-Logik an einer zentralen Stelle.

[![Open your Home Assistant instance and import this blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/osmosBe/homeassistant-scripts-blueprints/refs/heads/main/mobile_app_notification_group.yaml)

> [!IMPORTANT]
> Vor der Veröffentlichung musst du im Link des Buttons `DEIN-BENUTZERNAME` und `DEIN-REPOSITORY` durch den tatsächlichen GitHub-Benutzernamen und Repositorynamen ersetzen. Die Blueprint-Datei muss im Beispiel im Root des Branches `main` liegen.

## Funktionen

- Eine feste Empfängergruppe je Skriptinstanz
- Mehrere Mobile-App-Geräte als Empfänger
- Normale und bestätigbare Benachrichtigungen über dieselbe Schnittstelle
- Zwei frei beschriftbare Aktionsbuttons
- Strukturierte Rückgabe an die aufrufende Automation
- Erkennung, welches Gerät bestätigt hat
- Eindeutige Action-IDs für parallele Aufrufe
- Stabile Notification-Tags und automatisches Entfernen von anderen Geräten
- Erkennung weggewischter Benachrichtigungen unter Android
- Konfigurierbarer Timeout
- Android-Priorität, Sticky-Modus, Channel, Icon und Farbe
- iOS-Unterbrechungsstufe und Sound
- Deep Links zu Home Assistant oder externen URLs
- Ausnahmen für Notify-Dienste, die nicht aus dem Gerätenamen abgeleitet werden können
- Bis zu 100 parallele Aufrufe, standardmäßig 20

## Voraussetzungen

- Home Assistant **2024.10.0 oder neuer**
- Home Assistant Companion App auf allen Empfängergeräten
- Die Geräte müssen über die Integration `mobile_app` in Home Assistant registriert sein
- Funktionierende `notify.mobile_app_*`-Aktionen für die ausgewählten Geräte

## Grundidee

Aus dem Blueprint wird pro Empfängergruppe eine eigene Skriptinstanz erstellt, zum Beispiel:

- `script.benachrichtigung_haushalt`
- `script.benachrichtigung_obergeschoss`
- `script.benachrichtigung_admins`

Die Geräte werden nur bei der Einrichtung dieser Skriptinstanz ausgewählt. Eine Automation ruft anschließend lediglich das passende Gruppenskript auf:

```yaml
action: script.benachrichtigung_haushalt
data:
  title: "Waschmaschine"
  message: "Die Wäsche ist fertig."
```

## Installation

### Variante 1: Importbutton

1. Auf den Button **Open your Home Assistant instance and import this blueprint** am Anfang dieser README klicken.
2. Die angezeigte Blueprint-URL kontrollieren.
3. **Blueprint-Vorschau** öffnen und den Blueprint importieren.
4. Unter **Einstellungen → Automatisierungen & Szenen → Skripte** ein neues Skript aus dem Blueprint erstellen.

Der Importbutton funktioniert erst, wenn dieses Repository öffentlich erreichbar und seine tatsächliche Raw-URL im Button hinterlegt ist.

### Variante 2: Manuelle Installation

1. [`mobile_app_notification_group.yaml`](mobile_app_notification_group.yaml) herunterladen.
2. Die Datei in Home Assistant speichern, zum Beispiel unter:

   ```text
   /config/blueprints/script/<dein-name>/mobile_app_notification_group.yaml
   ```

3. In Home Assistant die Skript-Blueprints neu laden oder Home Assistant neu starten.
4. Unter **Einstellungen → Automatisierungen & Szenen → Skripte → Skript erstellen → Blueprint verwenden** den Blueprint **Mobile-App-Benachrichtigungsgruppe** auswählen.

## Empfängergruppe einrichten

Beim Erstellen einer Skriptinstanz werden die Einstellungen der Gruppe einmalig festgelegt.

### Mobile-App-Geräte

Wähle alle Geräte aus, die Benachrichtigungen dieser Gruppe erhalten sollen. Die Auswahl zeigt nur Geräte der Home-Assistant-Integration `mobile_app` an.

Der Blueprint leitet den Notify-Dienst standardmäßig aus dem Gerätenamen ab:

```text
Corinna Handy → notify.mobile_app_corinna_handy
```

### Abweichende Notify-Dienste

Gerätename und Notify-Dienst stimmen nicht immer überein, etwa nach einer Umbenennung oder Migration. Solche Ausnahmen werden unter **Erweitert → Abweichende Notify-Dienste** einmalig als Schlüssel/Wert-Zuordnung eingetragen.

Als Schlüssel sind die Home-Assistant-Geräte-ID oder der exakte Gerätename möglich. Als Wert wird die vollständige Notify-Aktion angegeben:

```yaml
Mauricios Phone: notify.mobile_app_moro
```

Die tatsächlich vorhandenen Aktionen findest du unter **Entwicklerwerkzeuge → Aktionen**, indem du nach `notify.mobile_app` suchst.

> [!TIP]
> Die Geräte-ID ist stabiler als der Gerätename. Für eine einfacher lesbare Konfiguration reicht der exakte Gerätename, solange er nicht geändert wird.

## Einstellungen der Skriptinstanz

### Zustellung

| Einstellung | Plattform | Wirkung |
| --- | --- | --- |
| Android – hohe Priorität | Android | Setzt `ttl: 0` und `priority: high`, damit die Meldung möglichst unmittelbar zugestellt wird. |
| Android – dauerhaft sichtbar | Android | Setzt die Meldung auf `sticky`. Sie bleibt sichtbar, bis sie aktiv entfernt wird. |
| Android – Benachrichtigungskanal | Android | Verwendet den angegebenen Notification Channel. Leer verwendet den Standardkanal. |
| iOS – Unterbrechungsstufe | iOS | `active`, `time-sensitive`, `critical` oder `passive`. |
| iOS – Ton | iOS | Optionaler vollständiger Dateiname eines verfügbaren Notification Sounds. |

Zeitkritische oder kritische iOS-Benachrichtigungen benötigen passende Berechtigungen und Einstellungen auf dem iPhone. Die Auswahl im Blueprint allein kann Betriebssystembeschränkungen nicht umgehen.

### Erweitert

| Einstellung | Standard | Beschreibung |
| --- | ---: | --- |
| Abweichende Notify-Dienste | `{}` | Manuelle Zuordnung von Geräte-ID oder Gerätename zu `notify.mobile_app_*`. |
| Maximale parallele Aufrufe | `20` | Maximale Anzahl gleichzeitig laufender Skriptaufrufe. Zulässig sind 1 bis 100. |

## Verwendung in Automationen

### Normale Benachrichtigung

Ohne `buttons` oder mit `buttons: false` wird die Nachricht versendet und das Skript sofort mit dem Ergebnis `sent` beendet.

```yaml
action: script.benachrichtigung_haushalt
data:
  title: "Waschmaschine"
  message: "Die Wäsche ist fertig."
```

### Benachrichtigung mit Bestätigung

Bei `buttons: true` wartet der Skriptlauf auf die erste Reaktion eines Empfängergeräts.

```yaml
action: script.benachrichtigung_haushalt
data:
  title: "Müllabfuhr"
  message: "Bitte den Restmüll rausstellen."
  buttons: true
  confirm_text: "Ist erledigt"
  dismiss_text: "Noch nicht"
  timeout_minutes: 180
  notification_tag: "restmuell"
response_variable: notification_result
```

Nach einer Bestätigung kann die aufrufende Automation das Ergebnis auswerten:

```yaml
- action: script.benachrichtigung_haushalt
  data:
    title: "Müllabfuhr"
    message: "Bitte den Restmüll rausstellen."
    buttons: true
    confirm_text: "Ist erledigt"
    dismiss_text: "Noch nicht"
    timeout_minutes: 180
    notification_tag: "restmuell"
  response_variable: notification_result

- condition: template
  value_template: "{{ notification_result.confirmed | default(false) }}"

- action: logbook.log
  data:
    name: "Müllabfuhr"
    message: >-
      {{ notification_result.confirmed_by }} hat die Aufgabe bestätigt.
```

### Benachrichtigung mit Ziel-URL und Android-Darstellung

```yaml
action: script.benachrichtigung_haushalt
data:
  title: "Haustür"
  message: "Die Haustür ist noch offen."
  url: "/lovelace/sicherheit"
  icon: "mdi:door-open"
  color: "#e53935"
```

Die Ziel-URL kann ein relativer Home-Assistant-Pfad wie `/lovelace/home` oder eine vollständige externe URL sein.

## Aufruffelder

| Feld | Erforderlich | Standard | Beschreibung |
| --- | --- | --- | --- |
| `title` | Ja | – | Titel der Benachrichtigung. |
| `message` | Ja | – | Nachrichtentext. Templates können von der aufrufenden Automation gerendert werden. |
| `buttons` | Nein | `false` | Aktiviert den Bestätigen- und Ablehnen-Button. |
| `confirm_text` | Nein | `Erledigt` | Beschriftung des Bestätigen-Buttons. |
| `dismiss_text` | Nein | `Noch nicht` | Beschriftung des Ablehnen-Buttons. |
| `timeout_minutes` | Nein | `360` | Wartezeit auf eine Reaktion; nur mit Buttons relevant. Bereich: 1–1440 Minuten. |
| `notification_tag` | Nein | leer | Gleiche Tags ersetzen ältere Meldungen. Bei Buttons wird ohne Angabe automatisch ein eindeutiger Tag erzeugt. |
| `url` | Nein | leer | Ziel beim Antippen der Benachrichtigung. |
| `icon` | Nein | leer | Android-Icon, beispielsweise `mdi:trash-can`. |
| `color` | Nein | leer | Android-Iconfarbe, beispielsweise `red` oder `#ff0000`. |

## Rückgabewerte

Damit eine Automation die Rückgabe erhält, muss der Skriptaufruf `response_variable` verwenden.

```yaml
response_variable: notification_result
```

Beispiel nach einer Bestätigung:

```yaml
result: confirmed
confirmed: true
confirmed_by: Corinna Handy
device_id: 0123456789abcdef0123456789abcdef
tag: restmuell
recipients:
  - 11111111111111111111111111111111
  - 22222222222222222222222222222222
```

| Schlüssel | Beschreibung |
| --- | --- |
| `result` | Ergebnisart: `sent`, `confirmed`, `dismissed`, `cleared`, `timeout` oder im unerwarteten Fall `unknown`. |
| `confirmed` | `true` ausschließlich bei Betätigung des Bestätigen-Buttons. |
| `confirmed_by` | Name des bestätigenden Mobile-App-Geräts; bei anderen Ergebnissen leer. |
| `device_id` | Home-Assistant-Geräte-ID des reagierenden Geräts, soweit von der App übermittelt. |
| `tag` | Verwendeter Notification-Tag. |
| `recipients` | Liste der Geräte-IDs dieser Skriptinstanz. |

### Bedeutung der Ergebnisse

| Ergebnis | Bedeutung |
| --- | --- |
| `sent` | Normale Benachrichtigung ohne Buttons wurde versendet. |
| `confirmed` | Der Bestätigen-Button wurde betätigt. |
| `dismissed` | Der Ablehnen-Button wurde betätigt. |
| `cleared` | Die Meldung wurde unter Android weggewischt. |
| `timeout` | Innerhalb des Zeitlimits wurde keine unterstützte Reaktion empfangen. |
| `unknown` | Es wurde ein unerwartetes Ereignis empfangen. Dieser Wert sollte im Normalbetrieb nicht auftreten. |

## Verhalten bei mehreren Empfängern

Jedes Gerät der Gruppe erhält dieselbe Nachricht. Bei einer bestätigbaren Benachrichtigung gilt die **erste Reaktion**:

1. Ein Gerät bestätigt, lehnt ab oder wischt die Meldung unter Android weg.
2. Der Skriptlauf bestimmt das Ergebnis.
3. Die Benachrichtigung wird anhand ihres Tags von allen Geräten der Gruppe entfernt.
4. Die Rückgabe wird an die aufrufende Automation übergeben.

Parallele Skriptaufrufe verwenden eindeutige Action-IDs. Eine Antwort kann dadurch nicht versehentlich einen anderen offenen Aufruf abschließen.

## Notification-Tags

Ein stabiler `notification_tag` sorgt dafür, dass eine neue Meldung mit demselben Tag eine ältere Meldung ersetzt. Das eignet sich beispielsweise für wiederkehrende Erinnerungen:

```yaml
notification_tag: "restmuell"
```

Bei Bestätigungsbuttons bleiben die Action-IDs trotz eines stabilen Tags pro Skriptlauf eindeutig.

Beachte: Werden zwei bestätigbare Aufrufe mit demselben Tag gleichzeitig gestartet, ist auf den Geräten nur die zuletzt zugestellte Meldung sichtbar. Der ältere Skriptlauf bleibt bis zu einer passenden Reaktion oder seinem Timeout aktiv. Für gleichzeitig sichtbare Aufgaben sollten daher unterschiedliche Tags verwendet werden.

## Android und iOS

Nicht jede Companion-App-Funktion verhält sich auf beiden Plattformen identisch:

- `cleared` wird nur unter Android ausgelöst, wenn eine Benachrichtigung weggewischt wird.
- Unter iOS führt das Entfernen einer Benachrichtigung nicht zu diesem Ereignis; der Skriptlauf endet dann über den Timeout.
- Notification Channels, Sticky, Icon und Farbe sind Android-spezifisch.
- Unterbrechungsstufe und der konfigurierte iOS-Sound sind iOS-spezifisch.
- Betriebssysteme können Benachrichtigungen abhängig von Energiesparmodus, Fokus, Berechtigungen und App-Einstellungen verzögern oder unterdrücken.

## Troubleshooting

### Nachricht wird an ein Gerät nicht gesendet

1. Unter **Entwicklerwerkzeuge → Aktionen** prüfen, ob die passende Aktion `notify.mobile_app_*` existiert.
2. Eine einfache Testnachricht direkt mit dieser Aktion senden.
3. Prüfen, ob der aus dem Gerätenamen abgeleitete Aktionsname stimmt.
4. Bei einer Abweichung den tatsächlichen Dienst unter **Abweichende Notify-Dienste** eintragen.
5. In den Companion-App-Einstellungen kontrollieren, ob die App korrekt mit dem Home-Assistant-Server verbunden ist und Benachrichtigungsrechte besitzt.

### Das Skript meldet „Action not found“

Der automatisch abgeleitete Notify-Dienst stimmt sehr wahrscheinlich nicht mit dem tatsächlichen Dienst überein. Hinterlege eine Ausnahme, beispielsweise:

```yaml
Mauricios Phone: notify.mobile_app_moro
```

### Eine weggewischte Meldung wartet weiter

Das ist unter iOS erwartbar. Das Ereignis `mobile_app_notification_cleared` wird nur von Android übermittelt. Der Lauf endet unter iOS nach `timeout_minutes`.

Unter Android sollte geprüft werden, ob die Companion App aktuell ist und das Ereignis in **Entwicklerwerkzeuge → Ereignisse** als `mobile_app_notification_cleared` ankommt.

### Die falsche oder keine Person wird zurückgegeben

`confirmed_by` basiert auf dem Mobile-App-Gerät, nicht auf einer sicheren Personenidentifikation. Bestätigt jemand auf dem Handy einer anderen Person, wird der Name dieses Handys zurückgegeben. Kann keine Geräte-ID zugeordnet werden, verwendet der Blueprint einen neutralen Ersatzwert.

### Ein Skriptlauf bleibt lange aktiv

Mit Buttons wartet jeder Aufruf bis zu einer Reaktion oder bis zum Timeout. Das Skript läuft im Modus `parallel`, blockiert dadurch aber keine späteren Aufrufe. Passe `timeout_minutes` und bei Bedarf die maximale Zahl paralleler Aufrufe an dein Einsatzszenario an.

### Gleicher Tag, mehrere offene Aufrufe

Ein gleicher Tag ersetzt die vorherige sichtbare Benachrichtigung, beendet aber nicht automatisch deren wartenden Skriptlauf. Verwende für unabhängige parallele Aufgaben unterschiedliche Tags.

## Sicherheit und Datenschutz

- Nachrichtentexte können auf dem Sperrbildschirm sichtbar sein. Versende keine sensiblen Inhalte, wenn die Geräteeinstellungen sie dort anzeigen.
- URLs in Benachrichtigungen sollten nur auf vertrauenswürdige Ziele verweisen.
- `confirmed_by` identifiziert das reagierende Gerät, nicht zweifelsfrei die Person.
- Kritische Benachrichtigungen sollten nicht als Ersatz für zertifizierte Alarmierungs-, Brandmelde- oder Sicherheitssysteme verwendet werden.

## Updates

Nach einer Aktualisierung der Blueprint-Datei:

1. Den Blueprint in Home Assistant neu importieren beziehungsweise die Datei ersetzen.
2. Die Blueprints neu laden.
3. Die aus dem Blueprint erstellten Skriptinstanzen öffnen, prüfen und erneut speichern, damit Änderungen übernommen werden.
4. Vor produktiver Verwendung je eine normale und eine bestätigbare Testnachricht ausführen.

## Credits

Die Geräteauswahl und einige plattformspezifische Zustellungsoptionen wurden von Blackys Community-Blueprint [Notifications & Announcements](https://community.home-assistant.io/t/notifications-announcements/728100) inspiriert. Die Empfänger- und Bestätigungslogik dieses Projekts wurde als eigenständiger, wiederverwendbarer Skript-Blueprint umgesetzt.

Dokumentation:

- [Home Assistant: Script Blueprint Schema](https://www.home-assistant.io/docs/blueprint/schema/)
- [Companion App: Actionable Notifications](https://companion.home-assistant.io/docs/notifications/actionable-notifications/)
- [Companion App: Notification Cleared](https://companion.home-assistant.io/docs/notifications/notification-cleared/)

## Lizenz

Füge vor der Veröffentlichung eine passende Open-Source-Lizenz zum Repository hinzu. Für einen unkompliziert wiederverwendbaren Home-Assistant-Blueprint eignet sich beispielsweise die MIT-Lizenz. Ohne Lizenz dürfen andere den öffentlich sichtbaren Code nicht automatisch verändern und weiterverbreiten.
