# Iot-Lab

## Wat

Het hoofddoel van dit project is om op een interactieve manier mensen toe te lichten over verbruik.
Een subdoel hierbij is om een lokaal smart te maken, hierin kan het hoofddoel dan verwerkt worden in een home assistant dashboard.
Onder dit subdoel horen presence detection, status monitoring van sites, het tonen van verbruik met leds en tonen van waarden die sensoren uitlezen.
Daarnaast hoort ook het maken van automations bij dit subdoel.

## Hoe

Aan de hand van home assistant zullen we de uitgelezen waarden van de digitale meter weergeven. Deze worden uitgelezen van een mqtt broker.
De presence detection zal gebeuren aan de hand van MAC adressen en ubiquiti access points, de status monitoring aan de hand van pings binnen home assistant en het aansturen van de leds met WLED.
Het maken van de automations gebeurt ook binnen home assistant.

## Dashboard schets (idee)

![dashboard schets](/img/idea_dashboard.png)

## Documentatie

### powersupply script

Een script om aan de hand van mqtt de waarden van een voeding aan te passen. Dit gebeurt met USB aan de hand van SCPI commands.

[powersupply repo documentatie](https://github.com/vives-project-xp/Iot-lab_powersupply_script)

### Home assistant

#### status monitoring

Status monitoring van services/sites kan binnen home assistant uitgevoerd worden aan de hand van de ping service.
Om een site/service te monitoren voeg je in configuration.yaml een site volgens onderstaande template toe onder binary_sensor.
Om in configuration.yaml te kunnen bewerken moet je de file editor add-on installeren. Ga hiervoor naar instellingen -> add-ons -> add-on store -> official add-ons -> file editor. Selecteer bij het installeren show in sidebar, na het installeren verschijnt de add on nu in de sidebar en kan je configuration.yaml bewerken.
Als er nog geen binary_sensor onderdeel te vinden is in configuration.yaml moet deze nog toegevoegd worden.

```txt
binary_sensor:
  - platform: ping
    host: '{site of service}'
    name: '{naam (zo kan deze entity later makkelijk teruggevonden worden)}'
    count: {aantal pings die verstuurd moeten worden}
    scan_interval: {om de hoeveel seconden worden pings gestuurd}
```

![state monitoring](/img/state_monitoring.png)
![state monitoring code example](/img/state_monitoring_example.png)

Om nu deze monitoring op je dashboard te tonen ga je naar je dashboard en klik je rechtsboven op de drie bolletjes -> edit dashboard.
Klik op add card en kies entities voeg hier onderaan de te monitoren sites/services hun entities toe
(te vinden aan de hand van de gegeven naam) en zet color icons based on state aan. Indien je ook wil zien wanneer de
sites/services down waren kan je nog een history graph card toevoegen. Voeg hiervoor zoals voor de vorige card de verschillende entities toe. 

