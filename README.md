# Centralina di irrigazione ESP32

Centralina embedded per la gestione automatica e manuale di un impianto di irrigazione, sviluppata su **ESP32** e controllabile tramite **Bluetooth** da un'applicazione realizzata con **Virtuino**.

Il codice originale, di Emiliano Pacenti, è stato ri-scritto per l'80%, la parte bluetooth di routing dei messaggi dall'app di Virtuino sono rimasti invariati.

Il sistema permette di configurare i parametri di irrigazione, gestire manualmente l'impianto e programmare cicli automatici con intervalli e durata configurabili.

## Caratteristiche

* Controllo automatico dell'irrigazione
* Controllo manuale tramite app e pulsanti fisici
* Programmazione dell'intervallo tra i cicli di irrigazione
* Durata dell'irrigazione configurabile
* Ritardo programmabile per la prima irrigazione
* Selezione dell'unità di tempo: ore o minuti
* Comunicazione Bluetooth con l'applicazione Virtuino
* Memorizzazione persistente della configurazione tramite EEPROM
* Monitoraggio del livello della batteria
* Monitoraggio del pannello solare
* Indicazione dello stato tramite LED
* Gestione dei timer tramite una libreria dedicata

## Architettura

Il sistema è composto da tre elementi principali:

```text
┌──────────────────────┐
│      Virtuino App    │
│   Android / Mobile   │
└──────────┬───────────┘
           │
        Bluetooth
           │
           ▼
┌──────────────────────┐
│        ESP32         │
│                      │
│  Firmware C/C++      │
│  ├─ VirtuinoCM       │
│  ├─ Timer            │
│  ├─ EEPROM           │
│  └─ Irrigation FSM   │
└───────┬───────┬──────┘
        │       │
        ▼       ▼
     Relays   Sensors
        │       │
        ▼       ├─ Battery
    Irrigation  └─ Solar panel
```

L'applicazione Virtuino comunica con il firmware attraverso una memoria virtuale `V[]`. I parametri configurabili e gli stati del sistema vengono sincronizzati tra ESP32 e applicazione tramite queste variabili.

## Modalità operative

### Automatico

In modalità automatica il firmware gestisce autonomamente il ciclo di irrigazione.

L'intervallo e la durata vengono configurati tramite Virtuino. Al raggiungimento dell'intervallo programmato viene avviato il ciclo di irrigazione; al termine della durata impostata il sistema spegne l'irrigazione e riparte con il conteggio del ciclo successivo.

È inoltre possibile configurare un ritardo specifico per la prima irrigazione.

### Manuale

La modalità manuale permette di comandare direttamente l'impianto senza attendere il ciclo automatico.

L'irrigazione può essere avviata e arrestata tramite i comandi disponibili nell'applicazione e tramite i controlli fisici presenti sulla centralina.

## Gestione degli stati

Il firmware utilizza una macchina a stati per separare le diverse condizioni operative:

```text
             ┌──────────────┐
             │   AUTOMATICO │
             │    stato 1   │
             └──────┬───────┘
                    │
             intervallo raggiunto
                    │
                    ▼
             ┌──────────────┐
             │  IRRIGAZIONE │
             │    stato 3   │
             └──────┬───────┘
                    │
              durata raggiunta
                    │
                    ▼
             ┌──────────────┐
             │   AUTOMATICO │
             └──────────────┘

             ┌──────────────┐
             │   MANUALE    │
             │    stato 2   │
             └──────────────┘
```

La gestione separata degli stati permette di distinguere il normale funzionamento automatico, il controllo manuale e la fase effettiva di irrigazione.

## Persistenza della configurazione

I principali parametri di funzionamento vengono salvati nella EEPROM dell'ESP32:

* intervallo di irrigazione
* durata dell'irrigazione
* modalità operativa
* ritardo della prima irrigazione
* unità di misura del tempo

In questo modo la configurazione viene mantenuta anche dopo il riavvio della centralina.

## Comunicazione Bluetooth

La centralina espone un dispositivo Bluetooth denominato:

```text
Melograni1
```

Il protocollo di comunicazione con l'applicazione è gestito dalla libreria **VirtuinoCM**.

Il firmware implementa le callback per la lettura e la modifica delle variabili virtuali utilizzate dall'applicazione:

```cpp
onReceived()
onRequested()
```

La funzione `virtuinoRun()` gestisce inoltre le richieste ricevute durante il normale funzionamento del firmware.

Per evitare di bloccare completamente la comunicazione durante le temporizzazioni, il progetto utilizza una funzione `vDelay()` che continua a elaborare le richieste Bluetooth durante l'attesa.

## Hardware

Il firmware utilizza, tra gli altri:

| Funzione                |    GPIO |
| ----------------------- | ------: |
| Relè irrigazione ON     | GPIO 32 |
| Relè irrigazione OFF    | GPIO 33 |
| LED di stato            |  GPIO 5 |
| Pulsante                | GPIO 23 |
| Pulsante                | GPIO 19 |
| Lettura batteria        | GPIO 35 |
| Lettura pannello solare | GPIO 34 |

La configurazione dei GPIO può essere adattata all'hardware utilizzato.

## Software e tecnologie

* **C/C++**
* **ESP32**
* **Arduino Framework**
* **PlatformIO**
* **Bluetooth Classic**
* **VirtuinoCM**
* **EEPROM**
* gestione temporale tramite `millis()`
* libreria `Timer` personalizzata

## Struttura del progetto

```text
Centralina_agricoltura/
│
├── lib/
│   └── Timer/
│       └── ...
│
├── src/
│   └── main.cpp
│
├── Melograni-2024luglio08.vrt6
├── platformio.ini
├── README.md
└── .gitignore
```

Il file `.vrt6` contiene il progetto dell'interfaccia realizzata con Virtuino.

## Build

Il progetto utilizza **PlatformIO**.

Per compilare il firmware:

```bash
pio run
```

Per caricarlo sulla scheda:

```bash
pio run --target upload
```

Il monitor seriale utilizza una velocità di:

```text
115200 baud
```

## Note

Questo progetto nasce come sistema reale per la gestione di un impianto di irrigazione e non come semplice esercizio didattico.

Il firmware integra comunicazione wireless, gestione dello stato, temporizzazione, controllo di uscite hardware, acquisizione di ingressi analogici e persistenza dei parametri di configurazione all'interno di un unico sistema embedded.

## Autore

**Fabio Crivellaro**

Embedded / Firmware / Electronics

[GitHub](https://github.com/crivellator)
