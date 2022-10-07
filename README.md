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

## powersupply script

Een script om aan de hand van mqtt de waarden van een voeding aan te passen. Dit gebeurt met USB aan de hand van SCPI commands.

[powersupply repo](https://github.com/vives-project-xp/Iot-lab_powersupply_script)