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

# Dag van de wetenschap

[DVDW](Dag_Van_De_Wetenschap.md)

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

Het is binnen home assistant ook mogelijk om een notificatie te sturen naar een gsm die verbonden is met de home assistant. Hiervoor moet het apparaat echter wel op
hetzelfde netwerk zitten als de home assistant.
Om een notificatie te sturen wanneer een service down is download je eerst de home assistant app en verbind je met jouw home assistant.
Vervolgens ga je naar instellingen -> automations -> create automation. Klik op add triger -> state en zoek naar de naam van de entity die je eerder aangemaakt hebt.
Selecteer bij From "connected" en bij To "disconnected". Vervolgens klik je op add action -> call service en zoek naar send a notification via mobile app. Hier kan
dan het juiste apparaat geselecteerd worden en een title en message voor de notification gekozen worden. Druk ten laatste op save.

Indien je je niet altijd op het netwerk van de home assistant bevindt kan je ook meldingen sturen via e-mail of whatsapp.
Voor whatsapp kan je de [callmebot](https://www.callmebot.com/blog/whatsapp-text-messages-from-homeassistant/) service gebruiken en voor e-mail de  
[SMTP service](https://www.home-assistant.io/integrations/smtp/).

#### toevoegen van sensoren, switches, ... aan home assistant

#### mqtt

1. mqtt broker binnen home assistant
2. publish vanuit home assistant  
   switch: om een switch toe te voegen die zijn state published naar mqtt moet je volgende code toevoegen aan configuration.yaml

```txt
switch:
  - platform: mqtt
    name: "{naam van de entity}"
    state_topic: "{topic om naar te publishen}"
    command_topic: "{topic om op te subscriben om updates te ontvangen}"
    qos: {te gebruiken mqqt qos, wij gebruiken 1}
    payload_on: "{wat te versturen wanneer de swith aan is}"
    payload_off: "{wat te versturen wanneer de swith uit is}"
    retain: {wanneer retain true is wordt na subscription onmiddelijk ontvangen wat de huidige state van de switch moet zijn}
```

input: om een input te publishen naar mqtt maken we gebruik van de mqtt.publish service, hiervoor moet in automations.yaml een automation toegevoegd worden.
voorbeeld slider

```txt
- id: '1665648904981'
  alias: powersupply current
  description: ''
  trigger:
  - entity_id: input_number.current_slider (de slider waarvan de waarde moet gepublished worden)
    platform: state (wanneer de state veranderd publishen wa de waarde)
  action:
  - service: mqtt.publish
    data:
      payload_template: '{{ states(''input_number.current_slider'') | float }}' (wat we zullen publishen en het type ervan)
      topic: current (naar welke topic het moet gepublished worden)
      retain: false
```

3. subscribe vanuit home assistant
