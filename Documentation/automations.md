# Onze gebruikte automaties

## POST command automation

1. 's Ochtends moet een foto op het scherm geplaatst worden, dit gebeurd met een post command. 's Avonds moet er een zwarte foto op het scherm geplaatst worden.

```yaml
alias: POST to screen
description: ""
trigger:
  - platform: time
    at: "08:15:00"
    id: morning
    enabled: false
  - platform: time
    at: "19:15:00"
    id: evening
    enabled: false
  - platform: state
    entity_id:
      - schedule.school_open
    from: "off"
    to: "on"
    id: school open
  - platform: state
    entity_id:
      - schedule.school_open
    id: school closed
    from: "on"
    to: "off"
action:
  - choose:
      - conditions:
          - condition: trigger
            id: school open
        sequence:
          - service: rest_command.post_spongebob
            data: {}
      - conditions:
          - condition: trigger
            id: school closed
        sequence:
          - service: rest_command.post_black
            data: {}
mode: single

```

In deze automation zijn de triggers twee tijdstippen. Als het kwart na acht is wordt de post spongebob rest command uitgevoerd en als het avond is wordt de post black command uitgevoerd.

- trigger: time ochtend en avond of een schedule helper -> als de schedule helper aan gaat mag alles aan anders moet het uit
- action: als er school is post een meme, als er geen school is post het een zwarte foto.

## Security

1. Snapshot nemen wanneer er niemand in een lokaal hoort te zijn, maar er toch motion is.

Voor deze automatie wordt gebruik gemaakt van een google calendar en een motion sensor om een snapshot te sturen via een melding. We doen dit voor security, zodat er niemand in het lokaal kan zijn wanneer er geen les is. Een mogelijke uitbreiding hierbij is om ervoor te zorgen dat dit niet gebeurt wanneer ons "alarm" uitgezet is.

```yaml
alias: send snapshot when motion and calendar event
description: ""
trigger:
  - platform: state
    entity_id:
      - binary_sensor.motion_sensor_home_security_motion_detection
    from: "off"
    to: "on"
    enabled: true
  - platform: calendar
    event: start
    offset: "0:0:0"
    entity_id: calendar.timeedit_bxh_a308_labo_elektronica_24p_cbru_0_20221101
    enabled: false
condition:
  - condition: state
    entity_id: calendar.timeedit_bxh_a308_labo_elektronica_24p_cbru_0_20221101
    state: "on"
action:
  - service: camera.snapshot
    target:
      entity_id:
        - camera.192_168_1_117
    data:
      filename: /media/Camera_1/last_motion.jpg
  - service: camera.snapshot
    target:
      entity_id:
        - camera.192_168_1_117
    data:
      filename: /media/Camera_1//archive/motion_{{ now().strftime("%Y%m%d-%H%M%S")}}.jpg
  - delay:
      hours: 0
      minutes: 0
      seconds: 2
      milliseconds: 0
  - service: notify.all_devices
    data:
      message: Camera 1
      title: Camera 1
      data:
        image: /media/local/Camera_1/last_motion.jpg

```

- trigger: In het trigger gedeelte kijken we of de state van de fibaro motion sensor van off naar on gaat. Hier staat off voor clear/geen motion detected en on voor motion detected.
- condition: In de condition controlleert of er op dit moment geen les is in het lokaal. We willen namelijk enkel een snapshot nemen en versturen wanneer er geen les hoort te zijn en dus ook geen motion.
- action: In het action gedeelte voeren we meerdere actions uit. Ten eerste wordt er een foto genomen door de geselecteerde camera en wordt deze opgeslagen. Hierna wachten we enkele seconden, om daarna de genomen foto te versturen in een melding naar de home assistant app.

2. Aan en uitzetten van alarm aan de hand van NFC tag.

Aan de hand van de volgende automaties wordt het alarm aan en uitgezet bij het scannen van een NFC tag. Wanneer deze tag gescanned wordt, zal ook een snapshot genomen worden die opgestuurd wordt naar de home assistant app. Deze automatisatie cycled door de helper die aangemaakt is met 2 options.

![helper](./img/helper_color.png)

```yaml
alias: Cycle RGB lights
description: ""
trigger:
  - platform: tag
    tag_id: 41094dc2-da31-4a49-8f0e-50ad6d08a20e
condition: []
action:
  - service: input_select.select_next
    data: {}
    target:
      entity_id: input_select.tag_color_toggle
  - service: camera.snapshot
    data:
      filename: /media/Camera_1/last_motion.jpg
    target:
      entity_id: camera.192_168_1_117
  - service: camera.snapshot
    target:
      entity_id:
        - camera.192_168_1_117
    data:
      filename: /media/Camera_1//archive/motion_{{ now().strftime("%Y%m%d-%H%M%S")}}.jpg
  - delay:
      hours: 0
      minutes: 0
      seconds: 2
      milliseconds: 0
  - service: notify.mobile_app_sm_a528b
    data:
      title: Camera 1
      message: Camera 1
      data:
        image: /media/local/Camera_1/last_motion.jpg
```

- trigger: De trigger van deze automatie is een gespecifieerde NFC tag.
- actions: Ten eerste cyclen we door een input_select helper, deze helper triggered onderstaande automaties die ervoor zorgen dat de lichten van kleur veranderen. Vervolgens voeren we dezelfde actions uit zoals hierboven uitgelegd om een snapshot te nemen en te versturen.

```yaml
alias: lights red
description: ""
trigger:
  - platform: state
    entity_id:
      - input_select.tag_color_toggle
    to: red_light
condition: []
action:
  - service: input_boolean.turn_on
    data: {}
    target:
      entity_id: input_boolean.alarm_state
  - service: script.light_go_red
    entity_id: light.lamp1
```

```yaml
alias: lights green
description: ""
trigger:
  - platform: state
    entity_id:
      - input_select.tag_color_toggle
    to: green_light
condition: []
action:
  - service: input_boolean.turn_off
    data: {}
    target:
      entity_id: input_boolean.alarm_state
  - service: script.cycle_rgb_lights
    data: {}
```

- trigger: De trigger van deze automatie is wanneer de tag_color_toggle en input_select helper verandert naar red_light.
- actions: Ten eerste zetten we de input_boolean die de state van ons alarm aantoont op on. Hierna wordt een script gerunt die de lamp op rood zet. Het script wordt hieronder uitgelegd.

Dit script zal ervoor zorgen dat de lamp rood wordt (phillips hue lamp). Hiervoor wordt er een service opgemaakt die de lamp aansteekt met het bepaald kleur (bv: rood zoals hieronder). Ook wordt er gestuurd via een whatsapp notificatie als het alarm aan of uit is. Vervolgens wordt de google hub aangestuurd en wordt er bijvoorbeeld gezegd: "The alarm is on". Hiervoor wordt er een media player service opgesteld, een delay aan toegevoegd van 4 seconden (zodat de text-to-speech niet blokkeert) en dan de media player uitzetten als laatste. Hetzelfde bij de "light go green"-script.

```yaml
alias: light go red
sequence:
  - service: light.turn_on
    data:
      rgb_color:
        - 255
        - 0
        - 0
    target:
      entity_id: light.ph1_alarm
  - service: input_select.select_first
    data: {}
    target:
      entity_id: input_select.tag_color_toggle
  - service: notify.whatsapp_message_phone1
    data:
      message: The alarm is turned on.
      title: ALARM
  - service: media_player.volume_set
    data:
      volume_level: 0.6
    target:
      device_id: 0300c8ca212177a3923003f6cceb9dbd
  - service: media_player.play_media
    target:
      entity_id: media_player.nesthub0c4d
    data:
      media_content_id: media-source://tts/google_translate?message=The+alarm+is+on
      media_content_type: provider
    metadata:
      title: The alarm is on
      thumbnail: https://brands.home-assistant.io/_/google_translate/logo.png
      media_class: app
      children_media_class: null
      navigateIds:
        - {}
        - media_content_type: app
          media_content_id: media-source://tts
        - media_content_type: provider
          media_content_id: media-source://tts/google_translate?message=The+alarm+is+on
  - delay:
      hours: 0
      minutes: 0
      seconds: 4
      milliseconds: 0
  - service: media_player.turn_off
    data: {}
    target:
      device_id: 0300c8ca212177a3923003f6cceb9dbd
mode: single
```

```yaml
alias: light go green
sequence:
  - service: light.turn_on
    data:
      rgb_color:
        - 0
        - 255
        - 0
    target:
      entity_id: light.ph1_alarm
  - service: input_select.select_last
    entity_id: input_select.tag_color_toggle
  - service: notify.whatsapp_message_phone1
    data:
      message: The alarm is turned off.
      title: "ALARM "
  - service: media_player.volume_set
    data:
      volume_level: 0.6
    target:
      device_id: 0300c8ca212177a3923003f6cceb9dbd
  - service: media_player.play_media
    target:
      entity_id: media_player.nesthub0c4d
    data:
      media_content_id: media-source://tts/google_translate?message=The+alarm+is+off
      media_content_type: provider
    metadata:
      title: The alarm is off
      thumbnail: https://brands.home-assistant.io/_/google_translate/logo.png
      media_class: app
      children_media_class: null
      navigateIds:
        - {}
        - media_content_type: app
          media_content_id: media-source://tts
        - media_content_type: provider
          media_content_id: media-source://tts/google_translate?message=The+alarm+is+off
  - delay:
      hours: 0
      minutes: 0
      seconds: 4
      milliseconds: 0
  - service: media_player.turn_off
    data: {}
    target:
      device_id: 0300c8ca212177a3923003f6cceb9dbd
mode: single
```

### Alarm creation

Dit script is voor als het alarm aan is en de deur gaat open. (Alarm_Automation_on)

-Triggers: Deze automation zal getriggered worden als de deur opengaat en als de condition **Alarm_state** aanstaat.

-Actions: Er lopen 3 acions in parallel => het volume wordt aangezet en dan speelt er een geluid voor 10 seconden.

Daarna komt het in een options terecht:

1. Als het alarm nog steeds aanstaat, dan zal het alarm afgaan totdat het alarm state wordt uitgezet en ook een notificatie.

2. Als het alarm uit is door de NFC-tag, dan zal er een notificatie komen die zal zeggen "Alarm is off".

```yaml
alias: Alarm_Automation_on
description: Als de deur open gaat en alarm staat aan
trigger:
  - type: opened
    platform: device
    device_id: dd68653e7801c9978c9305e192d66903
    entity_id: binary_sensor.lumi_lumi_sensor_magnet_aq2_opening
    domain: binary_sensor
condition:
  - condition: state
    entity_id: input_boolean.alarm_state
    state: "on"
action:
  - parallel:
      - service: media_player.volume_set
        data:
          volume_level: 0.6
        target:
          device_id: 0300c8ca212177a3923003f6cceb9dbd
      - service: media_player.play_media
        target:
          entity_id: media_player.nesthub0c4d
        data:
          media_content_id: >-
            media-source://media_source/local/Countdown_Clock_-_Bombe_Timer_SOUND.mp3
          media_content_type: audio/mpeg
        metadata:
          title: Countdown_Clock_-_Bombe_Timer_SOUND.mp3
          thumbnail: null
          media_class: music
          children_media_class: null
          navigateIds:
            - {}
            - media_content_type: app
              media_content_id: media-source://media_source
      - delay:
          hours: 0
          minutes: 0
          seconds: 10
          milliseconds: 0
      - service: media_player.media_stop
        data: {}
        target:
          device_id: 0300c8ca212177a3923003f6cceb9dbd
  - choose:
      - conditions:
          - condition: state
            entity_id: input_boolean.alarm_state
            state: "on"
        sequence:
          - repeat:
              until:
                - condition: state
                  entity_id: input_boolean.alarm_state
                  state: "off"
              sequence:
                - service: notify.whatsapp_message_phone1
                  data:
                    message: SOMEONE ENTERED WITHOUT SCANNING
                    title: THE ALARM IS GOING OFF
                - service: media_player.volume_set
                  data:
                    volume_level: 0.5
                  target:
                    device_id: 0300c8ca212177a3923003f6cceb9dbd
                - service: media_player.play_media
                  target:
                    entity_id: media_player.nesthub0c4d
                  data:
                    media_content_id: >-
                      media-source://media_source/local/salamisound-1344064-siren-howling-even-from-very.mp3
                    media_content_type: audio/mpeg
                  metadata:
                    title: salamisound-1344064-siren-howling-even-from-very.mp3
                    thumbnail: null
                    media_class: music
                    children_media_class: null
                    navigateIds:
                      - {}
                      - media_content_type: app
                        media_content_id: media-source://media_source
                - delay:
                    hours: 0
                    minutes: 0
                    seconds: 7
                    milliseconds: 0
      - conditions:
          - condition: state
            entity_id: input_boolean.alarm_state
            state: "off"
        sequence:
          - service: notify.whatsapp_message_phone1
            data:
              message: alarm disarmed
              title: Thanks for disarming the alarm
          - service: media_player.media_stop
            data: {}
            target:
              device_id: 0300c8ca212177a3923003f6cceb9dbd
mode: single
```

Het volgende script dient ervoor als het alarm terug wordt aangezet.

-Triggers: Als het alarm veranderd van state (off=>on) en als de deur open is.

-Actions: Er wordt een tijd gegeven van 10 seconden en dan komt het in options terecht.

1. Als de deur nog open is, zal het een melding sturen dat de deur nog openstaat en zal de google hub steeds afspelen dat je de deur moet sluiten.

2. Als de deur gesloten is, zal je een bericht krijgen dat de deur dicht is en zal de media van de google hub niet meer afspelen.

```yaml
alias: Alarm_On_When_Door_Open
description: ""
trigger:
  - platform: state
    entity_id:
      - input_boolean.alarm_state
    from: "off"
    to: "on"
condition:
  - type: is_open
    condition: device
    device_id: dd68653e7801c9978c9305e192d66903
    entity_id: binary_sensor.lumi_lumi_sensor_magnet_aq2_opening
    domain: binary_sensor
action:
  - delay:
      hours: 0
      minutes: 0
      seconds: 10
      milliseconds: 0
  - choose:
      - conditions:
          - type: is_open
            condition: device
            device_id: dd68653e7801c9978c9305e192d66903
            entity_id: binary_sensor.lumi_lumi_sensor_magnet_aq2_opening
            domain: binary_sensor
        sequence:
          - service: notify.whatsapp_message_phone1
            data:
              title: DOOR IS STILL OPEN, PLEASE CLOSE IT
              message: "Shut the door or the alarm won't work properly "
          - repeat:
              until:
                - type: is_not_open
                  condition: device
                  device_id: dd68653e7801c9978c9305e192d66903
                  entity_id: binary_sensor.lumi_lumi_sensor_magnet_aq2_opening
                  domain: binary_sensor
              sequence:
                - service: media_player.volume_set
                  data:
                    volume_level: 0.6
                  target:
                    device_id: 0300c8ca212177a3923003f6cceb9dbd
                - service: media_player.play_media
                  target:
                    entity_id: media_player.nesthub0c4d
                  data:
                    media_content_id: >-
                      media-source://tts/google_translate?message=Please+shut+the+door+or+i+will+do+selfdestruct.
                    media_content_type: provider
                  metadata:
                    title: Please shut the door or i will do selfdestruct.
                    thumbnail: >-
                      https://brands.home-assistant.io/_/google_translate/logo.png
                    media_class: app
                    children_media_class: null
                    navigateIds:
                      - {}
                      - media_content_type: app
                        media_content_id: media-source://tts
                      - media_content_type: provider
                        media_content_id: >-
                          media-source://tts/google_translate?message=Please+shut+the+door+or+i+will+do+selfdestruct.
                - delay:
                    hours: 0
                    minutes: 0
                    seconds: 5
                    milliseconds: 0
          - service: media_player.media_stop
            data: {}
            target:
              device_id: 0300c8ca212177a3923003f6cceb9dbd
      - conditions:
          - type: is_not_open
            condition: device
            device_id: dd68653e7801c9978c9305e192d66903
            entity_id: binary_sensor.lumi_lumi_sensor_magnet_aq2_opening
            domain: binary_sensor
        sequence:
          - service: notify.whatsapp_message_phone1
            data:
              message: Thanks for closing the door
              title: Door closed
          - service: media_player.media_stop
            data: {}
            target:
              device_id: 0300c8ca212177a3923003f6cceb9dbd
mode: single
```

## DVDW automations

1. Lichten van kleur veranderen afhankelijk van of het nacht of dag is in de simulatie van een dag.

```yaml
alias: Night/Day toggle
description: ""
trigger:
  - platform: state
    entity_id:
      - switch.powersupply_switch
    from: "on"
    to: "off"
    id: night
  - platform: state
    entity_id:
      - switch.powersupply_switch
    id: day
    from: "off"
    to: "on"
condition: []
action:
  - service: light.turn_on
    data:
      rgb_color:
        - 139
        - 215
        - 253
    target:
      entity_id:
        - light.dashboard_lamp_3
  - choose:
      - conditions:
          - condition: trigger
            id: night
        sequence:
          - service: light.turn_on
            data:
              rgb_color:
                - 139
                - 215
                - 253
            target:
              entity_id:
                - light.dashboard_lamp_3
      - conditions:
          - condition: trigger
            id: day
        sequence:
          - service: light.turn_on
            data:
              rgb_color:
                - 255
                - 174
                - 0
            target:
              entity_id:
                - light.dashboard_lamp_3
mode: single
```

- triggers: In deze automatie maken we gebruik van trigger ID's om een onderscheiding te maken tussen verschillende triggers en dan andere actions uit te gaan voeren. We hebben hier twee triggers wanneer de switch van off naar on gaat en het dus dag wordt en wanneer de switch van on naar off gaat en het dus nacht wordt.
- action: In onze action zetten we eerst de lamp aan om vervolgens een choose action te gebruiken. Hierin kan aan de hand van ons entity ID gekozen worden welke action uitgevoerd moet worden. Wanneer de id=night, zetten we de lamp op een wit/lichtblauwe kleur en wanneer de ID=day, zetten we de lamp op een oranje kleur.

2. Aan en uitzetten van koelkast afhankelijk van state van simulatie dag

```yaml
alias: koelkast(on/off)
description: ""
trigger:
  - platform: state
    entity_id:
      - switch.powersupply_switch
    id: psu_off
    from: "on"
    to: "off"
  - platform: state
    entity_id:
      - switch.powersupply_switch
    from: "off"
    to: "on"
    id: psu_on
condition: []
action:
  - choose:
      - conditions:
          - condition: trigger
            id: psu_off
        sequence:
          - type: turn_off
            device_id: 810484b76179e181971fced662893273
            entity_id: switch.lumi_lumi_plug_maeu01_switch
            domain: switch
      - conditions:
          - condition: trigger
            id: psu_on
        sequence:
          - type: turn_on
            device_id: 810484b76179e181971fced662893273
            entity_id: switch.lumi_lumi_plug_maeu01_switch
            domain: switch
mode: single
```

- trigger: We hebben voor deze automatie opnieuw twee triggers met elk een trigger ID. Het is hier opnieuw of de switch van de dag simulatie van off naar on gaat en van on naar off.
- action: In de action wordt afhankelijk van de trigger ID's van de triggers gekozen om de switch van de koelkast aan of uit te zetten.

3. Apparaat aanzetten met een actionable notification

```yaml
alias: Actionable bouwlamp 2_test
description: ""
trigger:
  - platform: device
    device_id: 694d768c2536b86d13ea2abe505c6313
    domain: zwave_js
    type: event.value_notification.central_scene
    property: scene
    property_key: "001"
    endpoint: 0
    command_class: 91
    subtype: Endpoint 0 Scene 001
condition: []
action:
  - service: script.actionable_notification_turn_on_device
    data:
      deviceName: boiler
mode: single
```

- trigger: De trigger van deze automation is de push van een button.
- action: Als action wordt een script uitgevoerd die een actionable notification stuurt en wordt als data de naam van het device meegestuurd.

```yaml
alias: Actionable notification (turn on device ??)
sequence:
  - service: notify.ALL_DEVICES
    data:
      message: >-
        Do you want to turn on or off the {{ deviceName }} 

        production/consumption = ( {{ states('sensor.l1_power_production') }} kW
        / {{ states('sensor.l1_power_consumption') }} kW) 

        gemiddeld verbruik = 500W
      title: "{{deviceName}}"
      data:
        actions:
          - action: TURN_ON_{{deviceName}}
            title: Turn on the device
          - action: TURN_OFF_{{deviceName}}
            title: Turn off device
mode: single
```

Het script dat door de automatie gerunt wordt, stuurt een actionable notification naar de home assistant app met twee buttons. Wanneer op deze buttons gedrukt wordt, zal de action met de naam TURN_ON_deviceName (bv. boiler) of TURN_OFF_deviceName uitgevoerd worden. Momenteel is hier nog het verbruik hardcoded, maar het zou de bedoeling zijn om ook dit als argument mee te geven aan het script.

Vervolgens hebben we nog een automation die de verschillende actions van onze actionable notifcation handled.

```yaml
alias: actions from actionable notification
description: ""
trigger:
  - platform: event
    event_type: mobile_app_notification_action
    event_data:
      action: TURN_ON_boiler
    id: turn_on_device
  - platform: event
    event_type: mobile_app_notification_action
    id: turn_off_device
    event_data:
      action: TURN_OFF_boiler
condition: []
action:
  - type: toggle
    device_id: db6d7e69c52288cebd3673f3a21be0cf
    entity_id: switch.bouwlamp2_plug_3
    domain: switch
    enabled: false
  - choose:
      - conditions:
          - condition: trigger
            id: turn_on_device
        sequence:
          - type: turn_on
            device_id: db6d7e69c52288cebd3673f3a21be0cf
            entity_id: switch.bouwlamp2_plug_3
            domain: switch
      - conditions:
          - condition: trigger
            id: turn_off_device
        sequence:
          - type: turn_off
            device_id: db6d7e69c52288cebd3673f3a21be0cf
            entity_id: switch.bouwlamp2_plug_3
            domain: switch
mode: single
```

- trigger: De triggers van deze automatie gebeuren wanneer een mobile_app_notification_action ontvangen wordt. Deze triggers hebben een trigger ID om bij action een geschikte action te kunnen kiezen.
- action: Als actions wordt afhankelijk van het trigger ID een switch aan of uit gezet.

4. De boiler uitzetten wanneer de oven aangezet wordt

```yaml
alias: Oven_Boiler_Off
description: ""
trigger:
  - platform: state
    entity_id:
      - binary_sensor.oven
    from: "off"
    to: "on"
condition: []
action:
  - type: turn_off
    device_id: db6d7e69c52288cebd3673f3a21be0cf
    entity_id: switch.bouwlamp2_plug_3
    domain: switch
mode: single
```

- trigger: De oven wordt aangezet.
- action: De boiler wordt uitgezet.

5. Waarde van een slider publishen naar mqtt (stroom)

```yaml
alias: powersupply current
description: ""
trigger:
  - entity_id: input_number.current_slider
    platform: state
action:
  - service: mqtt.publish
    data:
      payload_template: "{{ states('input_number.current_slider') | float }}"
      topic: current
      retain: true
```

- trigger: De waarde van de slider verandert.
- action: Haal de waarde van de slider op en publish het naar de current topic als een float.

6. Waarde van een slider aanpassen met een button

```yaml
alias: zon onder
description: ""
trigger:
  - device_id: 58b656184167d7fe2d0c4ce54c4e9dba
    domain: zha
    platform: device
    type: remote_button_short_press
    subtype: dim_down
  - device_id: 58b656184167d7fe2d0c4ce54c4e9dba
    domain: zha
    platform: device
    type: remote_button_long_press
    subtype: dim_down
condition: []
action:
  - service: input_number.decrement
    data: {}
    target:
      entity_id: input_number.current_slider
mode: single
```

- trigger: Er wordt op een button gedrukt.
- action: Verminder de waarde van de button met 1, indien increment zou gebruikt worden vermeerderd de waarde met 1.

## Iot Lab automations

1. Stuur een melding wanneer de humiditeit te hoog is.

```yaml
alias: humidity too high
description: ""
trigger:
  - platform: numeric_state
    entity_id: sensor.lumi_lumi_weather_humidity
    above: 65
condition: []
action:
  - service: notify.whatsapp_message_phone1
    data:
      message: the humidity is high, consider opening a window
mode: single
```

- trigger: De humiditeit is boven de 65%.
- action: Stuur een bericht naar de gebruiker.
