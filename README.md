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
  !!!! HIER MOET NOG KOMEN HOE JE EEN MQTT BROKER OPZET BINNEN HASS !!!!!
2. publish vanuit home assistant  
   switch: om een switch toe te voegen die zijn state published naar mqtt moet je volgende code toevoegen aan configuration.yaml

```txt
  name: "{naam van de entity}"
  state_topic: "{topic om naar te publishen}"
  command_topic: "{topic om op te subscriben om updates te ontvangen}"
  qos: {te gebruiken mqqt qos, wij gebruiken 1}
  payload_on: "{wat te versturen wanneer de swith aan is}"
  payload_off: "{wat te versturen wanneer de swith uit is}"
  retain: {wanneer retain true is wordt na subscription onmiddelijk ontvangen wat de huidige state van de switch moet zijn}

mqtt:
    broker: 127.0.0.1
    port: 1883
    client_id: home-assistant
    keepalive: 60
    switch:
      - unique_id: powersupply_switch
        name: "powersupply switch"
        state_topic: "state"
        command_topic: "state"
        payload_on: "on"
        payload_off: "off"
        qos: 1
        retain: true
      - unique_id: current_effect_switch
        name: "current effect switch"
        state_topic: "currentEffect"
        command_topic: "currentEffect"
        qos: 1
        payload_on: "on"
        payload_off: "off"
        retain: true
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
  !!!! HIER MOET NOG KOMEN HOE JE SUBSCRIBED OP EEN TOPIC BINNEN HASS !!!!!

### Home assistant dashboards

Voor het maken van onze dashboards hebben we gebruik gemaakt van een aantal add-ons.
Om de add-ons te installeren maken we gebruik van HACS.

1. Ga naar de Add-on store
2. Installeer een SSH add-on, bijvoorbeeld "Terminal & SSH" (Om deze add-ons te zien moet je advanced mode aanzetten in je user profile)
3. Configureer je gekozen add-on volgens de documentatie van de add-on
4. Start de SSH add-on
5. Connecteer met de SSH add-on
6. Run het volgende command

```txt
wget -O - https://get.hacs.xyz | bash -
```

Wanneer HACS geïnstalleerd is komt het in je sidebar. Selecteer het in je sidebar en klik op Frontend voor dashboard/styling add-ons.
Voor onze dashboards hebben we gebruik gemaakt van Mushroom cards, layout-card, card-mod en apexcharts-card.

#### Mushroom cards

Van Mushroom cards hebben we gebruik gemaakt van de grid cards om het plaatsen van cards in de juiste volgorde en in bepaalde kolommen makkelijker te maken.
Hieronder voorbeeld code van een grid card met het resultaat eronder. In dit voorbeeld hebben we twee kolommen waarin telkens cards zitten. In de eerste kolom zit er daarnaast nog een grid card met 3 kolommen om de status van de knoppen van onze verschillende verbruikers te tonen.

```text
square: false
columns: 2
type: grid
cards:
  - square: false
    columns: 1
    type: grid
    cards:
      - type: custom:mushroom-title-card
        title: Dag van de wetenschap
      - type: entities
        entities:
          - entity: switch.powersupply_switch
          - entity: switch.current_effect_switch
          - entity: input_number.voltage_slider
          - entity: input_number.current_slider
        state_color: true
      - type: grid
        cards:
          - type: button
            tap_action:
              action: toggle
            entity: switch.scherm_mat
          - type: button
            tap_action:
              action: toggle
            entity: switch.scherm_mat
          - type: button
            tap_action:
              action: toggle
            entity: switch.scherm_mat
      - type: custom:apexcharts-card
        graph_span: 5m
        update_interval: 2s
        span:
          start: minute
          offset: '-2m'
        header:
          show: true
          title: Stroom
          show_states: true
          colorize_states: true
        series:
          - entity: sensor.currunt
  - square: false
    columns: 1
    type: grid
    cards:
      - type: custom:mushroom-title-card
        title: Iot lab
      - type: entities
        entities:
          - entity: sensor.lumi_lumi_weather_humidity
          - entity: sensor.lumi_lumi_weather_temperature
          - entity: sensor.lumi_lumi_weather_pressure
      - type: history-graph
        entities:
          - entity: binary_sensor.google_test
        title: Google
      - type: history-graph
        entities:
          - entity: binary_sensor.youtube_test
        title: Youtube
```

![example grid card](/img/grid_mushroom.png)

#### layout-card

Layout-card bied 3 nieuwe view types aan voor ons dashboard: Horizontal, Vertical en Grid. Wij hebben hiervan gebruik gemaakt van grid. Hieronder het verschil tussen de default view type en grid view type van layout-card.

![grid view](/img/grid_mushroom.png)
![default view](/img/default_layout.png)

#### apexcharts-card

Apexcharts staat ons toe om mooiere grafieken toe te voegen aan ons dashboard. Persoonlijk hebben wij het gebruikt om een grafiek van de stroom van onze voeding te tonen.

![apexcharts grafiek](/img/clean_graph.png)
