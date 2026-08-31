# 🏎️ F24 Motorsturing: ICW Project

Dit project beschrijft de essentiële sensoren en protocollen die gebruikt worden voor de datacollectie en uiteindelijke motorsturing van ons F24-voertuig.

Hier vind je alvast ons schema:

![Schema](https://github.com/Viktorvanderaerschot1/ICWProjecten_auto/blob/main/Schema_ESP32%202025-11-18%20141505.png)
---

# 🛠 Hardware Component: ADS1115 (16-bit ADC)

De **ADS1115** is een cruciaal onderdeel van ons telemetriesysteem in de kart. Deze module fungeert als de brug tussen de analoge sensoren (zoals batterijspanning en stroomshunts) en de digitale verwerking op de ESP32.

### 📋 Kernfunctionaliteit
* **Type:** 16-bit Analoge-naar-Digitale Converter (ADC).
* **Communicatie:** I2C-bus (SDA op Pin 25, SCL op Pin 26).
* **Doel:** Het nauwkeurig meten van batterijvoltages en stroomverbruik via de analoge poorten (AIN0-AIN3).

---

### 🧠 Waarom hebben we gekozen voor 16-bit?

Voor een race-applicatie zoals de F24 kart is standaard resolutie vaak onvoldoende. Wij hebben specifiek voor de 16-bit ADS1115 gekozen op basis van de volgende argumenten:

1. **Superieure Resolutie (Stapgrootte):**
   Een standaard 10-bit ADC verdeelt een signaal in slechts 1.024 stapjes. De ADS1115 gebruikt **65.536 stapjes**. Dit stelt ons in staat om zeer kleine spanningsvallen te detecteren wanneer de motor wordt belast, wat essentieel is voor een nauwkeurige berekening van de resterende accucapaciteit.

2. **Nauwkeurige Stroommeting (Shunt):**
   Stroomsensoren en shunts geven vaak een heel klein analoog signaal af (millivolts). Dankzij de 16-bit resolutie kunnen we dit zwakke signaal versterken en meten zonder dat het verdrinkt in de ruis.

3. **Ruisonderdrukking :**
   In een kart zorgt de motor voor veel elektromagnetische storing. Hoewel we 16 bits tot onze beschikking hebben, gebruiken we de extra resolutie vooral om een stabiel "schoon" signaal over te houden na filtering. Zelfs met ruis houden we een nauwkeurigheid over die vele malen hoger ligt dan die van de ingebouwde ADC's van de ESP32.

4. **Differentiële Metingen:**
   De ADS1115 kan het verschil tussen twee ingangen meten. Dit is cruciaal voor het elimineren van "ground noise" die ontstaat door de hoge stromen die door het frame van de kart lopen.

---


### 📚 Documentatie
Voor meer details over de registers en elektrische limieten, zie de [officiële ADS1115 Datasheet](https://www.ti.com/lit/ds/symlink/ads1115.pdf).
### 💻 Python Code

| Bestand | Beschrijving |
| :--- | :--- |
| **`Adressing.py`** | Behandelt de $I^2C$ adressering voor communicatie met de ADS1115. |
| **`ads1x15 file.py`** | De basisbibliotheek (`library`) voor het aansturen van de ADS-chip. |
| **`ADS1115.py`** | Testcode voor het uitlezen en valideren van de metingen. |

* [ADS1115-Adressing](https://github.com/Viktorvanderaerschot1/ICWProjecten_auto/blob/main/Adressing.py)
* [ADS1x15_lib](https://github.com/Viktorvanderaerschot1/ICWProjecten_auto/blob/main/ads1x15%20file.py)
* [ADS115_test](https://github.com/Viktorvanderaerschot1/ICWProjecten_auto/blob/main/ADS1115.py)

---

## 2. MicropyGPS: GPS Dataverwerking

Deze module zorgt voor de verwerking van GPS-locatiedata, datalogging en eventuele snelheidsregelingen.

### 🗺️ De Rol van het NMEA-Protocol

De GPS-module zendt ruwe data uit via het **NMEA-protocol** (National Marine Electronics Association) in de vorm van leesbare tekstzinnen (NMEA-zinnen).

De **MicropyGPS-library** functioneert als een **parser** die deze ruwe NMEA-zinnen leest, decodeert en omzet in direct bruikbare variabelen. De NMEA-zinnen worden in de **`Test gps micropyGPS.py`** code gebruikt als de **input data** die de MicropyGPS-library verwerkt.

### 📊 Geëxtraheerde GPS-Informatie

De parser haalt de volgende vitale gegevens uit de NMEA-data:

* **Locatie:** Latitude / Longitude
* **Tijd & Datum**
* **Hoogte**
* **Satellietinformatie**

### 📄 Verwerkte NMEA Zinnen

De code leest en verwerkt zinnen zoals:

* `$GPGGA` – **Global Positioning System Fix Data** (Positie, tijd, en fix-kwaliteit).
* `$GPRMC` – **Recommended Minimum navigation info** (Positie, snelheid, koers, en datum).
* `$GPGSV` – **Satellites in view** (Informatie over de waargenomen satellieten). 

### 💻 Python Code

| Bestand | Beschrijving |
| :--- | :--- |
| **`MicropyGPS_NMEA.py`** | Bevat de basisstructuur, de initialisatie van de MicropyGPS-parser en de logica voor het verwerken van inkomende bytes. |
| **`Test gps micropyGPS.py`** | Testbestand dat de MicropyGPS-library gebruikt om de NMEA-data te verwerken en de GPS-waarden te valideren. |

* [GPS_NMEA](https://github.com/Viktorvanderaerschot1/ICWProjecten_auto/blob/main/MicropyGPS_NMEA.py)
* [TEST_GPS](https://github.com/Viktorvanderaerschot1/ICWProjecten_auto/blob/main/Test%20gps%20micropyGPS.py)

---
## 3. MLX90614: Infrarood Temperatuursensor

De **MLX90614** is een digitale infraroodsensor die ontworpen is om **contactloos de temperatuur** te meten. Dit is ideaal voor het monitoren van de temperatuur van de motor of andere kritieke componenten van het voertuig zonder direct fysiek contact.



### 🌡️ Functionele Details en Protocol

* **Meetprincipe:** De sensor meet de thermische straling (infrarood) die door een object wordt uitgezonden, en berekent op basis daarvan de temperatuur in $\degree C$ of $\degree F$. Dit biedt een veilige en nauwkeurige manier om de temperatuur op afstand te bepalen.
* **Communicatie:** De MLX90614 communiceert via de **$I^2C$ bus** (Twee-draads Interface), waardoor hij met slechts twee datalijnen ($\text{SDA}$ en $\text{SCL}$) eenvoudig is aan te sluiten op de ESP32 NodeMCU.
* **Uitvoerwaarden:** De sensor levert twee kritieke temperatuurwaarden:
  **Omgevingstemperatuur (Ambient):** De temperatuur van de sensorchip zelf.

### 💻 Python Implementatie voor de ESP32

De volgende Python-bestanden zijn noodzakelijk voor het uitlezen van de sensor op de ESP32. 
| Bestand | Beschrijving | URL voor Code |
| :--- | :--- | :--- |
| **Library (`lib`)** | De basisbibliotheek die de $I^2C$ communicatie en het uitlezen van de temperatuurregisters van de MLX90614 regelt. | **[mlx90614_lib](https://github.com/Viktorvanderaerschot1/ICWProjecten_auto/blob/main/mlx90614.py)** |
| **Testcode** | Het hoofdscript dat de library importeert, de sensor initialiseert en de omgeving- en objecttemperatuur periodiek uitleest en valideert. | **[mlx90614](https://github.com/Viktorvanderaerschot1/ICWProjecten_auto/blob/main/Testcode-mlx90614.py)** |

|Omgevingstemperatuur  | Min | Max |
| :--- | :--- | :--- |
| Object | -70°C | 382.2°C |
| Ambient | -40°C | 125°C |

---
## 4. OT3686: Optical Speed Sensor Module

De **OT3686** is een optische snelheidsmodulesensor (encoder-module) die de snelheid van motoren meet en gebruikt wordt voor toepassingen zoals pulstelling en positionering.  
Deze sensor werkt met een infrarood lichtzender en een fototransistor om de snelheid te detecteren en is ontworpen om externe lichtbronnen te weerstaan.


### ⚙️ Functionele Details en Principe

#### **Meetprincipe**
- De sensor werkt met een U-vormige constructie bestaande uit een infrarood (IR) LED-zender en een fototransistor-ontvanger.
- Een roterende schijf (encoder-schijf) met openingen onderbreekt de IR-straal periodiek.
- Elke onderbreking genereert een digitaal puls-signaal (HIGH/LOW) op de uitgangspen van de sensor.

#### **Signaal**
- De uitgang is een **digitaal signaal**.
- Het aantal pulsen per tijdseenheid is direct evenredig met de rotatiesnelheid.

#### **Datacollectie**
- De microcontroller (ESP32) telt deze pulsen met een **interrupt** om nauwkeurig de rotatiesnelheid in **RPM** (Rotaties per Minuut) te bepalen.


### 💻 Python Implementatie voor de ESP32

| Bestand | Beschrijving |
|--------|--------------|
| **Encoder_Handler.py** | Logica voor het opzetten van de GPIO-interrupt en de ISR (Interrupt-Service-Routine) die elke puls telt. |
| **Speed_Calculation.py** | Het hoofdscript dat de totale puls-telling periodiek uitleest en de **RPM** berekent op basis van het aantal pulsen per rotatie. |

**Links:**

- [Encoder_Handler_URL](https://github.com/Viktorvanderaerschot1/ICWProjecten_auto/blob/main/hall.py)  
- [Speed_Calculation_URL](https://github.com/Viktorvanderaerschot1/ICWProjecten_auto/blob/main/pulse.py)

  ---
  ## 5. Visueel Dashboard & Live Telemetrie (MQTT)

Om de prestaties van de kart in realtime te volgen, worden alle verzamelde sensorgegevens via een radiomodule verzonden naar een centraal dashboard.  
Dit dashboard is gebouwd met **Pygame** en maakt gebruik van de **Yandex Maps API** voor navigatie en circuitvisualisatie.


## 🎮 Waarom Pygame?

Hoewel Pygame vaak wordt geassocieerd met gameontwikkeling, is dit framework bewust gekozen voor het dashboard om de volgende technische redenen:

- **Lage latency**  
  Pygame biedt directe controle over de frame rate en grafische rendering, wat essentieel is voor het vloeiend weergeven van snel veranderende sensordata (zoals RPM-naalden en snelheidsmeters).

- **Hardware-integratie**  
  Pygame verwerkt gelijktijdig keyboard-inputs en inkomende datastreams efficiënt, zonder dat de interface blokkeert of bevriest.


## 🗺️ Live Circuit Tracking met Yandex Maps

Voor de visuele weergave van de kart op het circuit gebruiken we de **Yandex Maps Static API**.

### Waarom Yandex Maps i.p.v. Google Maps of OpenStreetMap?

- **Hoge resolutie satellietbeelden**  
  Op veel locaties, waaronder specifieke racecircuits, biedt Yandex scherpere en actuelere satellietbeelden dan OpenStreetMap (OSM).

- **Eenvoudige integratie**  
  De Yandex API maakt het mogelijk om via eenvoudige HTTP-requests statische kaarten op te halen, die direct als `Surface` in Pygame kunnen worden ingeladen en gerenderd.

- **Geen kosten / minder restricties**  
  Voor dit type project biedt Yandex een toegankelijker gratis quotum en minder complexe authenticatie-eisen dan de Google Maps API, wat de ontwikkelsnelheid verhoogt.


## 📡 Datastroom via MQTT & Radio

Het systeem maakt gebruik van een draadloze datastroom waarbij de kart fungeert als **Publisher** en het dashboard als **Subscriber**.

- **Radio-link**  
  Een RF-module in de kart verzendt de ruwe sensordata naar een grondstation.

- **MQTT Broker**  
  Het grondstation vertaalt de inkomende data naar MQTT-berichten.

- **Dashboard**  
  In de interface kan geselecteerd worden welke kart gevolgd wordt door te subscriben op het specifieke ID-topic van de kart.


## 📊 Dashboard Overzicht
Hier vind je alvast ons dashboard:

![Dashboard](https://github.com/Viktorvanderaerschot1/ICWProjecten_auto/blob/main/Schermafbeelding%202026-02-02%20134754.png)
Het dashboard toont de volgende kritieke waarden:

| Gegevenstype   | Bron        | Weergave |
|---------------|-------------|----------|
| Snelheid      | OT3686      | Realtime weergave in km/u |
| Batterijen    | ADS1115     | Spanning ($V$) van beide accu's (balance monitoring) |
| Temperatuur   | MLX90614    | Motor- en omgevingstemperatuur |
| GPS-positie   | MicropyGPS  | Live marker op de Yandex satellietkaart |

---

## 💻 Python Code

| Bestand | Beschrijving |
|-------|--------------|
| `Dashboard_Main.py` | De Pygame main loop die de interface tekent en de MQTT-data verwerkt |
| `Map_Handler.py` | Haalt kaartsegmenten op via de Yandex Maps API op basis van GPS-coördinaten |

🔗 **[Bekijk Dashboard Code](https://github.com/Viktorvanderaerschot1/ICWProjecten_auto/blob/main/MQTT_data(1).py)**


## 📡 6. Draadloze Datatransmissie (LoRa & MQTT)

Om de data van de bewegende kart betrouwbaar in de pits te krijgen, maken we gebruik van een hybride communicatienetwerk dat radiofrequenties combineert met cloud-protocollen.

### 📶 LoRa Radioverbinding (Long Range)
De kart is uitgerust met een LoRa-radiomodule die via **UART** verbonden is met de ESP32. 
* **Protocol:** De sensordata wordt op de ESP32 ingepakt als een binair pakket (`struct`). Dit is vele malen efficiënter dan het versturen van tekst, waardoor de verbinding minder gevoelig is voor storingen en de reikwijdte wordt vergroot.
* **Transmissie:** De module zendt de data uit op de 433/868 MHz band, wat een stabiel signaal geeft over het volledige circuit.

### 🌐 MQTT Gateway & Broker
De ontvanger aan de kant van de baan fungeert als een bridge (gateway) tussen de radiogolven en het internet.
* **Decodering:** Het grondstation ontvangt de binaire HEX-string en vertaalt deze naar een gestructureerd **JSON-object**.
* **Publicatie:** Deze JSON wordt via het MQTT-protocol gepubliceerd naar de broker `mqtt.2-wire.xyz`.
* **Topic Hiërarchie:** Door de data te versturen naar `f24/kart/[ID]`, kan het dashboard zich gericht abonneren op de data van de gewenste kart.

### 💻 Python Code

| Bestand | Beschrijving | URL voor Code |
| :--- | :--- | :--- |
| **Receiver (PC)** | Vertaalt radio-pakketten naar MQTT JSON-data. | **[radio2mqtt.py](https://github.com/Viktorvanderaerschot1/ICWProjecten_auto/blob/main/Send_data.py)** |
| **Sender (ESP32)** | Verzamelt sensordata en stuurt deze naar de LoRa-module. | **[main.py](https://github.com/Viktorvanderaerschot1/ICWProjecten_auto/blob/main/Send_data.py)** |

---



