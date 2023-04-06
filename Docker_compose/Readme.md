# Installation Docker

The purpose of the install script is for a convenience for quickly
installing the latest Docker-CE releases on the supported linux
distros. It is not recommended to depend on this script for deployment
to production systems. For more thorough instructions for installing
on the supported distros, see the [install
instructions](https://docs.docker.com/engine/install/).

## Usage:

From `https://get.docker.com`:
```shell
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

From `https://test.docker.com`:
```shell
curl -fsSL https://test.docker.com -o test-docker.sh
sh test-docker.sh
```
Gebruiker toevoegen aan docker group

```sh
sudo usermod -aG docker gebruikersnaam

```

# Running Docker

Het bestand bevat de definities van vijf services:

- Home Assistant
- Mosquitto
- InfluxDB
- Grafana
- Node-RED

## Home Assistant

Home Assistant is een open-source platform voor home automation. Met de container_name, image, ports, volumes, restart, en privileged eigenschappen definieer je hoe de container moet worden opgestart en welke configuratie-instellingen moeten worden gebruikt. Home Assistant draait op poort 8123 en heeft toegang nodig tot de lokale tijdzone

### Waarom docker

Deze configuratie stelt ons in staat om ons zonnepaneelscript uit te voeren en alle Docker-images op één Raspberry Pi te laten draaien. Bovendien kunnen we het dashboard eenvoudig vanaf deze enkele Raspberry Pi openen, waardoor het gemakkelijker wordt om het systeem te monitoren en te beheren.

## Mosquitto

Mosquitto is een open-source message broker dat het MQTT (Message Queuing Telemetry Transport) protocol ondersteunt. Het laat applicaties en apparaten toe om berichten te versturen en te ontvangen in een gedistribueerd systeem.

Er worden volumes gemount voor het opslaan van data en logs. Verder wordt ook de poort 1883 (MQTT) en 9001 (MQTT over WebSockets) geëxposeerd, zodat het systeem berichten kan ontvangen en verzenden via het MQTT-protocol.

## InfluxDB

InfluxDB is een open-source time series database dat gebruikt kan worden voor het opslaan en analyseren van tijdsgebonden data, zoals sensordata.

In de Docker Compose file wordt een container gemaakt met de image "influxdb:1.8". Er wordt ook een naam aan de container gegeven ("influxdb") en een volume wordt gemount voor het opslaan van de InfluxDB data. Verder wordt ook de poort 8086 geëxposeerd voor de toegang tot de InfluxDB HTTP API.


## Grafana 
Grafana is een open-source platform voor data-analyse en visualisatie. Het kan worden gebruikt om data uit verschillende bronnen te visualiseren en om dashboards te maken voor het monitoren van verschillende systemen.

Er worden volumes gemount voor het opslaan van de data en de configuratie. Verder wordt ook de poort 3000 geëxposeerd, zodat het Grafana dashboard toegankelijk is via een webbrowser. Afhankelijk van de configuratie kan het dashboard data visualiseren die opgeslagen is in InfluxDB.

## Node-RED
Node-RED is een open-source visuele programmeeromgeving die gebruikt kan worden om Internet of Things (IoT) toepassingen te ontwikkelen. Het laat toe om eenvoudig flows te bouwen door middel van een visuele interface en biedt de mogelijkheid om data te integreren uit verschillende bronnen en protocollen.

We gebruiken Node-Red om data van mqtt in onze influx te plaatsen.