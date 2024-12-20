# Installation
## (Optional) Installation einer neueren Version von Node.js
Falls man eine neuere Version von Node.js verwenden möchte/muss, als im Repository von Raspbian vorhanden, kann mann das wie folgt machen ([Quelle](https://pimylifeup.com/raspberry-pi-nodejs/)):

    sudo apt update
    sudo apt upgrade
    sudo apt install -y ca-certificates curl gnupg
    curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/nodesource.gpg
    NODE_MAJOR=22
    echo "deb [signed-by=/usr/share/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list
    sudo apt update
    sudo apt install nodejs

## Installation von Node-RED
Für Node-RED sollte ein eigener User verwendet werden, der zur sudo-Gruppe hinzugefügt wird und dessen Passwort auch gesetzt ist:

    sudo useradd -d /opt/nodered -m -s /bin/bash nodered
    sudo usermod -a -G sudo nodered
    sudo passwd nodered

Ab jetzt sollte mit diesem User gearbeitet werden.

Zunächst muss [Node-RED installiert](https://nodered.org/docs/getting-started/) werden, falls noch nicht vorhanden.

## Installation benötigter Bibliotheken
Folgende Module müssen über `Palette verwalten -> Installation` installiert werden:
- `node-red-contrib-cron-plus`
- `node-red-dashboard`
- `node-red-contrib-ui-led`

Einige weitere Bibliotheken müssen manuell in der Shell installiert werden, während man sich im Verzeichnis `~/.node-red` befindet. Dazu muss der Befehl 

    npm i <name>

für jeden Namen aus der nachfolgenden Liste einmal ausgeführt werden, wobei `<name>` jeweils durch den Listeneintrag ersetzt wird:

- `@camueller/json-converter`
- `@date-fns/tz`
- `date-fns`

Außerdem müssen diese Bibliotheken noch in der Datei `~/.node-red/settings.js` eingetragen werden. Dazu in der Datei nach `functionGlobalContext`) suchen und wie folgt ändern:

    functionGlobalContext: {
        jsonconverter:require('@camueller/json-converter'),
        datefnstz:require('@date-fns/tz'),
        datefns:require('date-fns')
    },

## Zeitzone
Damit die Berechnungen funktionieren, muss die Zeitzone in Node-RED richtig gesetzt sein.
Dazu muss in der Datei `~/.node-red/settings.js` oberhalb von `module.exports = {` eingetragen werden:

    process.env.TZ = "Europe/Berlin"

## Logging
### Node-RED Service-Log in Datei schreiben (optional)
Das Service-Log von Node-RED kann mit dem Befehl `node-red-log` angezeigt werden. Standardmäßig wird es nicht in eine Datei geschrieben.

Zur besseren Nachvollziehbarkeit erzeugt der Flow einige Einträge im Service-Log. Wenn diese in eine Datei geschrieben werden sollen, muss zunächst `resyslog` installiert werden:

    sudo apt install rsyslog

Dafür ist die Installation folgender Konfigurationsdatei erforderlich:


    sudo wget https://github.com/camueller/node-red-contrib-heatpump-dynamic-tariff/raw/master/nodered-logging.conf -P /etc/rsyslog.d

Abschliesssend muß `resyslog` neu gestartet werden:

    sudo systemctl restart rsyslog

### Log-Level
Zur besseren Nachvollziehbarkeit erzeugt der Flow einige Einträge im Service-Log. Diese sind mit dem Log-Level `debug` zugeordnet. Nach der Installation von Node-RED ist der Log-Level `info` aktiv, d.h. Log-Einträge mit Log-Level `debug` werden nicht ausgegeben.

Die Änderung der Log-Levels auf `debug` erfolgt in der Datei `~/.node-red/settings.js`:

     logging: {
         console: {
             level: "debug",
             ...
         }
     },

## Flow-Import
Der Import der Flows in Node-RED erfolgt über das Menü `Import`. Nach Klick in den zentralen, roten Bereich des Import-Dialoges kann dort das [Flow-JSON für Heatpump-Tibber-PV](https://raw.githubusercontent.com/camueller/node-red-contrib-dynamic-tariff/main/flow.json) eingefügt werden. Durch Klicken des `Import`-Buttons werden die Flows in Node-RED importiert. 

## Deploy
Wenn zuvor alle benötigten Bibliotheken installiert wurden, sollten dabei keine Fehler auftreten. Für die fehlerfreie Ausführung des Flows ist zusätzlich die [Konfiguration der Parameter](configuration.md) erforderlich.
