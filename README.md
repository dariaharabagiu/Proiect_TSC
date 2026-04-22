# InkTime - Proiect Smartwatch Open-Source (nRF52840)

**InkTime** este un dispozitiv purtabil de tip smartwatch, optimizat pentru eficienta energetica extrema si conectivitate Bluetooth Low Energy (BLE). Proiectul integreaza un ecosistem complex de senzori si management de putere intr-un format compact.

---

### 1. Diagrama Bloc a Sistemului (InkTime)

          [ Port USB-C ]
                |
                v (5V)
      [ Incarcator LiPo BQ25180 ] <---> [ Acumulator LiPo ]
                |
                v
      [ Regulator DC/DC RT6160 ] ----> ( 3.3V Power Rail )
                |                         |
                |          +--------------+-------------+
                |          |              |             |
                v          v              v             v
        +-------------------------------------------------------+
        |                nRF52840 MCU (Cortex-M4F)              |
        |                   Bluetooth Low Energy                |
        +-------------------------------------------------------+
                ^          |              |             |
          (I2C) |    (SPI) |       (GPIO) |       (IRQ) |
                v          v              v             v
        [ IMU BMA421 ] [ E-Paper ]  [ Haptic Driver ] [ Butoane ]
        [ Fuel Gauge ] [ Display ]  [   DRV2605L    ]
                                          |
                                          v
                                   [ Motor Vibratii ]

---

## 2. Bill of Materials (BOM) - Selectie JLC Parts

| Componenta | Rol | Cod JLC (LCSC) | Datasheet |
| :--- | :--- | :--- | :--- |
| **nRF52840-QIAA** | MCU + BLE | [C190623](https://jlcpcb.com/partdetail/NordicSemiconductor-nRF52840QIAAR/C190623) | [Link](https://infocenter.nordicsemi.com/pdf/nRF52840_PS_v1.0.pdf) |
| **BQ25180** | Li-Po Charger IC | [C5332824](https://jlcpcb.com/partdetail/TexasInstruments-BQ25180YFPR/C5332824) | [Link](https://www.ti.com/lit/ds/symlink/bq25180.pdf) |
| **BMA421** | IMU (Acc/Giro) | [C404552](https://jlcpcb.com/partdetail/BoschSensortec-BMA421/C404552) | [Link](https://www.bosch-sensortec.com/) |
| **DRV2605L** | Haptic Driver | [C136239](https://jlcpcb.com/partdetail/TexasInstruments-DRV2605LDGSR/C136239) | [Link](https://www.ti.com/lit/ds/symlink/drv2605l.pdf) |
| **GDEW0154T8** | 1.54" E-Paper | [C512345](https://jlcpcb.com/parts) | [Link](https://www.good-display.com/) |
| **RT6160** | DC/DC Buck-Boost | [C3005615](https://jlcpcb.com/parts) | [Link](https://www.richtek.com/) |

---

## 3. Functionalitate Hardware si Interfete

### A. Managementul Energiei
* **BQ25180:** Gestioneaza incarcarea bateriei prin USB-C. Este configurat via I2C pentru a seta curentul de terminare si tensiunea de siguranta.
* **RT6160:** Asigura o tensiune stabila de **3.3V** pentru restul componentelor, chiar si atunci cand tensiunea bateriei scade sub 3.3V (operare Buck-Boost).
* **Consum:** S-a optimizat consumul prin utilizarea modului *Deep Sleep* al nRF52840 si oprirea regulatorului haptic cand nu este folosit.

### B. Senzori si Feedback
* **BMA421 (IMU):** Comunica prin **I2C**. Foloseste pinii de intrerupere (INT1/INT2) pentru a "trezi" procesorul doar la detectarea miscarii (economisire energie).
* **DRV2605L:** Driver haptic avansat care ruleaza secvente de vibratie pre-programate pentru a simula click-uri mecanice.

### C. Display E-Paper
* **Interfata:** SPI (pana la 12MHz).
* **Circuit Boost:** Include un circuit extern (inductor + mosfet) necesar generarii tensiunilor de +/-15V pentru reorientarea particulelor de cerneala electronica.

---

## 4. Alocare Pini nRF52840 (Pinout)

| Periferic | Pin MCU | Protocol | Motivatie |
| :--- | :--- | :--- | :--- |
| **I2C SCL / SDA** | P0.27 / P0.26 | I2C | Bus comun pentru IMU, Charger, Fuel Gauge si Haptic. |
| **SPI MOSI / SCK** | P0.20 / P0.19 | SPI | Viteza mare pentru transferul buffer-ului de imagine. |
| **EPD CS / DC** | P0.17 / P0.21 | GPIO | Controlul fluxului de date catre ecran. |
| **IMU INT1** | P0.11 | Input/IRQ | Detectare gest "Lift to Wake". |
| **USB D+ / D-** | D+ / D- | USB | Pini dedicati nRF52840 pentru date USB. |
| **SWDIO / SWDCLK** | Dedicat | Debug | Programare si debugging hardware. |

---

## 5. Design Log si Note de Revizuire

### Proiectare PCB (Layout)
* **Stack-up:** 2 Layere (Top/Bottom). S-a folosit plan de masa continuu pe Bottom pentru integritatea semnalului.
* **Trasee Putere:** Latime de **0.30mm** pentru liniile principale de alimentare (VCC, V_BATT).
* **Antena:** Antena de tip Meander (PCB Trace Antenna), pozitionata la marginea placii. S-a pastrat un *Keep-out area* de 5mm fara cupru sub antena pentru a evita atenuarea semnalului BLE.
