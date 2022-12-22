# Opzetten van de home assistant

Ten eerste plug je de home assitant en CDEM in. Hierbij moet de CDEM verbonden zijn met de digitale meter.
Indien de CDEM nog meer setup nodig heeft zie de [documentatie Dag_Van_De_Wetenschap](./Dag_Van_De_Wetenschap.md).

Hierna kan je de schermen instellen, hiervoor heb je een extra pi nodig. Op deze pi verbind je met de home assistant en kan je dan de verschillende dashboards tonen. De mogelijke gebruikers zijn iot met wachtwoord iot_lab1234 en watcher met wachtwoord WatcherHomeAssistant.
Op deze pi wordt ook het gescript gebruikt die de voeding aanstuurt om de dag te simuleren. Hiervoor moeten eerst de voeding en de inverter aanstaan. Steek de voeding in, maar niet in het net van de digitale meter en verbind hem vervolgens met de inverter. Steek de inverter in in het net van de digitale meter en verbind de USB van de voeding met de pi. Voer vervolgens op de pi het powersupply script uit met python3 mqtt.py. Het script is te vinden in de directory /Documents/powersupply_final/Iot_Lab_Powersupply_Script/ Bij het runnen van dit script moet je zeker maken dat het ip address van de mqtt broker juist ingesteld is en de poort waarop de voeding aangesloten is ook juist staat. Voor meer info zie [Powersupply repo documentatie](https://github.com/vives-project-xp/Iot-lab_powersupply_script).

Voor de opstelling moet je vervolgens de verschillende bouwlampen/toestellen verbinden die we gebruiken om te simuleren. In onze opstelling zijn de verschillende plugs gelabeled met het toestel waarmee verbonden wordt. De koelkast is het toestel dat het minste verbruikt, dan de oven en dan de boiler. Om de oven en boiler aan te sturen worden de buttons gebruikt waarop oven en boiler staan.

Vervolgens moet je de twee cameras aan zetten waarop a306 en a308 staat, de fibaro motion sensor aanzetten waarop motion staat, de door sensor aanzetten waarop door staat en de twee weather sensors waarop lumi 1 en lumi 2 staat.
