# MPPT

![license](https://img.shields.io/badge/license-MIT-blue) ![offline-first](https://img.shields.io/badge/offline--first-air--gap-green) ![audit](https://img.shields.io/badge/audit-SHA3--256-orange) ![category](https://img.shields.io/badge/category-solar-lightgrey)

> Anticloud-hardened packaging of the upstream project `MPPT` in category **SOLAR**. Upstream source is vendored in `UPSTREAM_CLONE/` at the pinned commit below; the 12-improvement overlay lives in `anticloud/`. Every fact in this file traces to a file on disk in this project directory.

**Category:** SOLAR · **Upstream:** https://github.com/KinDR007/VictronMPPT-ESPHOME · **Upstream pin:** `f9cd94f9409fce853123510f3c3be40c8b1d9ec6` · **Vendor:** Anticloud FZ LLE

---

## What This Project Does

# VictronMPPT-ESPHOME

ESPHome component to monitor a Victron device _(MPPT, SmarShunt, BMV, Phoenix Inverter ... )_ via ve.direct / UART TTL

## Supported devices

All Victron devices providing a ve.direct port.

## Tested devices

  * Victron SmartSolar MPPT 75/10 rev2 (0xA074)
  * Victron SmartSolar MPPT 75/15
  * Victron SmartSolar MPPT 100/15
  * Victron SmartSolar MPPT 100/20
  * Victron BlueSolar  MPPT 100/30
  * Victron SmartSolar MPPT 150/35
  * Victron SmartSolar MPPT 150/45
  * Victron SmartSolar MPPT 150/60
  * Victron SmartSolar MPPT VE.Can 150/100 rev2 (Using VE.Direct port)
  * Victron SmartSolar MPPT 250/70
  * Victron Phoenix Inverter 12/500
  * Victron Blue Smart IP65 Charger 12|25
  * Victron Orion XS 12|12

  * Victron SmartShunt 500A/50mV

## Requirements

* [ESPHome 2024.6.0 or higher](https://github.com/esphome/esphome/releases).
* Generic ESP32, ESP8266, BK72xx or RP2040 board

## Schematics

Attention: [The TX voltage of the VE.Direct interface depends on the product: Some are 5V, others 3.3V!](https://www.victronenergy.com/live/vedirect_protocol:faq#q4is_the_vedirect_interface_33_or_5v)

Please measure the voltage between TX and GND. In case of a logic level of 5V you should add a voltage divider or logic level converter between the ESP and the Victron device.

```
                UART-TTL
┌────────────────┐                ┌──────────────────┐
│           GND o│<-------------->│o GND             │
│ Victron    TX o│--------------->│o D7   ESP32/     │
│ Charger    RX o│                │       ESP8266    │<-- GND
│            5V o│                │                  │<-- 3.3V
└────────────────┘                └──────────────────┘

# UART-TTL jack (JST-PH 2.0mm pinch)
┌─── ─────── ────┐
│                │
│ O   O   O   O  │
│GND  RX  TX VCC │
└────────────────┘
```

If you are unsure about to pin order please measure the voltage between GND and VCC (5V). If you measure a positive voltage you know the position of VCC and GND!

### JST-PH jack (direct connection noninsulated)

| Pin     | Purpose      | ESP8266 pin        | ESP32 pin        |
| :-----: | :----------- | :------------- |:------------- |
|  **1**  | **GND**      | **GND**          |**GND**           |
|    2    | RX           |                |               |
|  **3**  | **TX**       | D7 (**RX**)        | GPIO16 (**RX**)        |
|    4    | 5V           |                |               |

<a href="images/circuit.jpg" target="_blank">
<img src="images/circuit_thumbnail.jpg" width="50%">
</a>

note: _JST-PH connector on this image has lock downwards_
<img width="70" alt="image" src="https://github.com/KinDR007/VictronMPPT-ESPHOME/assets/61905170/184607a8-4a18-437d-a859-d32b5c329a0b">

According to the [VE.direct FAQ](https://www.victronenergy.com/live/vedirect_protocol:faq), the 5V/3.3 pin has a strict current limitation, allowing only 10mA on average with occasional 20mA bursts lasting up to 5 milliseconds. ESP devices like the ESP8266 and ESP32 draw significantly more current than this limit, typically several hundred milliamperes during startup and an additional 100-150mA when transmitting, which would immediately overload and permanently damage the Victron device's power circuit. Therefore, powering an ESP directly from these pins is not possible.

note: ~~_Level shifter (5V<->3.3V)between esp32 and mppt is not necessary_ (tested on esp32 nodemcu v1 and MPPT 75/115 , MPPT 150/35 and smartshunt 500A)~~

## Safe connection (insulated)
   use `ADUM 1201` for galvanic isolation between ESP and Victron Devices (or make an insulated cable for Ve.direct)

|Ve.Direct <img width="70" alt="image" src="https://github.com/KinDR007/VictronMPPT-ESPHOME/assets/61905170/184607a8-4a18-437d-a859-d32b5c329a0b"> _downwards_|Ve.Direct <img width="70" alt="image" src="https://github.com/KinDR007/VictronMPPT-ESPHOME/assets/61905170/c1e2f7c5-c9c1-4b34-9c8e-fb6dc1e144dd"> **upward** | Purpose (wire color)|connect|ADUM1201 #2| ADUM1201 #1 |connect| ESP8266 pin| ESP32 pin|USB TTL cable|
|:-----| :----- | :----------- |:-----|:------------- |:------------- |:-----|:----- |:-----|:-----|
| _1_ |**1**  | **GND** (black) |<-  ->|  **GND2**  |  **GND1**  |<-  ->| **GND** |**GND**|**GND**|
| _2_ |**2** | **RX** (red) |<-  ->|  **VOB**  | **VIB** |<-  ->| D8 (**TX**) | GPIO17 (**TX**)|**TX**|
| _3_ |**3**  | **TX** (white) |<-  ->| **VIA**|  **VOA** |<-  ->|D7 (**RX**) | GPIO16 (**RX**)|**RX**|
| _4_ |**4** | **5V** or **3V3** (yellow) |<-  ->|  **VDD2**  | **VDD1** |<-  ->| **3V3 or 5V**  | **3V3 or 5V** |**3V3 or 5V**|

Note that if the VE.direct RX line is connected to VOB on the ADUM1201, the corresponding VIB needs to be connected to a valid signal, i.e. VDD1 or GND1. Depending on the RX pin configuration of the device, this might enable/disable the device. Check the device documentation for details. If unsure, best leave the RX line unconnected, in which case the VIB pin of the ADUM still should be tied to VDD1 or GND1 to not have it floating.

 ![adum1201_](https://github.com/KinDR007/VictronMPPT-ESPHOME/assets/61905170/9c7cf60c-fb19-404a-9c6e-7a17bc463ead)

[Buy on Ebay](https://www.ebay.com/itm/176094311751?var=475546476315&norover=1&mkevt=1&mkrid=711-168613-066356-2&mkcid=2&itemid=475546476315_176094311751&targetid=295222477609&device=c&mktype=pla&googleloc=1003785&poi=&campaignid=20797276730&mkgroupid=155163397599&rlsatarget=pla-295222477609&abcId=&merchantid=119648210&gad_source=1&gbraid=0AAAAAD_QDh8nutO0oonh8hgwFzBcKUcJc&gclid=Cj0KCQjw-ai0BhDPARIsAB6hmP6RN5aHtdBKIpl4EERR_dzQcCZcNqUsNd9KJGAJuTSsmuLnHL7lLs4aAnRtEALw_wcB)

[Buy on Aliexpress](https://www.aliexpress.com/item/1005005362881990.html?spm=a2g0o.productlist.main.55.22b97SeK7SeKfK&algo_pvid=8aa6f818-a16f-465d-a50d-d974fbbda803&algo_exp_id=8aa6f818-a16f-465d-a50d-d974fbbda803-27&pdp_npi=4%40dis%21CZK%2153.02%2129.24%21%21%212.23%211.23%21%40210387a117203472445525945efb01%2112000032744813603%21sea%21CZ%210%21AB&curPageLogUid=lG8tIpV8MVs7&utparam-url=scene%3Asea

*(excerpt; full text in `UPSTREAM_CLONE/`)*

*Quoted from the upstream `README.md` file in `UPSTREAM_CLONE/`.*
Project-specific facts detected in this directory:

- Ecosystem: **Python** (manifests: pyproject.toml; scanned in UPSTREAM_CLONE)
- Top-level source layout: `components/`, `images/`, `tests/`
- Snapshot size: **61 files**, **3498 lines of code** (measured; see Benchmarks)
- Primary languages: `.yaml` (28), `.h` (11), `(none)` (7), `.py` (5), `.cpp` (2), `.jpg` (2)
- Upstream commit pinned for this packaging: `f9cd94f9409fce853123510f3c3be40c8b1d9ec6`

---

## Installation

You can install this component with [ESPHome external components feature](https://esphome.io/components/external_components.html) like this:
```yaml
external_components:
  - source: github://KinDR007/VictronMPPT-ESPHOME@main

uart:
  id: uart_0
  tx_pin: D8  # Not used! The communication is read-only
  rx_pin: D7
  baud_rate: 19200
  rx_buffer_size: 256

victron:
  id: victron0
  uart_id: uart_0

sensor:
  - platform: victron
    victron_id: victron0
    panel_voltage:
      name: "Panel voltage"
    battery_voltage:
      name: "Battery voltage"
    battery_current:
      name: "Battery current"
```

or just use the `esp8266-example.yaml` as proof of concept:

```bash

*Section quoted from the upstream readme.*
Overlay install (this project):

```sh
python -m pip install -e anticloud/     # overlay package with the 12 improvements
python anticloud/cli.py --help          # 13 subcommands, JSON stdout
```

---

## Usage

|Ve.Direct <img width="70" alt="image" src="https://github.com/KinDR007/VictronMPPT-ESPHOME/assets/61905170/184607a8-4a18-437d-a859-d32b5c329a0b"> _downwards_|Ve.Direct <img width="70" alt="image" src="https://github.com/KinDR007/VictronMPPT-ESPHOME/assets/61905170/c1e2f7c5-c9c1-4b34-9c8e-fb6dc1e144dd"> **upward** | Purpose (wire color)|connect|ADUM1201 #2| ADUM1201 #1 |connect| ESP8266 pin| ESP32 pin|USB TTL cable|
|:-----| :----- | :----------- |:-----|:------------- |:------------- |:-----|:----- |:-----|:-----|
| _1_ |**1**  | **GND** (black) |<-  ->|  **GND2**  |  **GND1**  |<-  ->| **GND** |**GND**|**GND**|
| _2_ |**2** | **RX** (red) |<-  ->|  **VOB**  | **VIB** |<-  ->| D8 (**TX**) | GPIO17 (**TX**)|**TX**|
| _3_ |**3**  | **TX** (white) |<-  ->| **VIA**|  **VOA** |<-  ->|D7 (**RX**) | GPIO16 (**RX**)|**RX**|
| _4_ |**4** | **5V** or **3V3** (yellow) |<-  ->|  **VDD2**  | **VDD1** |<-  ->| **3V3 or 5V**  | **3V3 or 5V** |**3V3 or 5V**|

Note that if the VE.direct RX line is connected to VOB on the ADUM1201, the corresponding VIB needs to be connected to a valid signal, i.e. VDD1 or GND1. Depending on the RX pin configuration of the device, this might enable/disable the device. Check the device documentation for details. If unsure, best leave the RX line unconnected, in which case the VIB pin of the ADUM still should be tied to VDD1 or GND1 to not have it floating.

 ![adum1201_](https://github.com/KinDR007/VictronMPPT-ESPHOME/assets/61905170/9c7cf60c-fb19-404a-9c6e-7a17bc463ead)

[Buy on Ebay](https://www.ebay.com/itm/176094311751?var=475546476315&norover=1&mkevt=1&mkrid=711-168613-066356-2&mkcid=2&itemid=475546476315_176094311751&targetid=295222477609&device=c&mktype=pla&googleloc=1003785&poi=&campaignid=20797276730&mkgroupid=155163397599&rlsatarget=pla-295222477609&abcId=&merchantid=119648210&gad_source=1&gbraid=0AAAAAD_QDh8nutO0oonh8hgwFzBcKUcJc&gclid=Cj0KCQjw-ai0BhDPARIsAB6hmP6RN5aHtdBKIpl4EERR_dzQcCZcNqUsNd9KJGAJuTSsmuLnHL7lLs4aAnRtEALw_wcB)

[Buy on Aliexpress](https://www.aliexpress.com/item/1005005362881990.html?spm=a2g0o.productlist.main.55.22b97SeK7SeKfK&algo_pvid=8aa6f818-a16f-465d-a50d-d974fbbda803&algo_exp_id=8aa6f818-a16f-465d-a50d-d974fbbda803-27&pdp_npi=4%40dis%21CZK%2153.02%2129.24%21%21%212.23%211.23%21%40210387a117203472445525945efb01%2112000032744813603%21sea%21CZ%210%21AB&curPageLogUid=lG8tIpV8MVs7&utparam-url=scene%3Asearch%7Cquery_from%3A)

---

*Section quoted from the upstream readme.*
Anticloud overlay CLI (available in every project):

```sh
python anticloud/cli.py --help     # 13 subcommands, JSON stdout
python anticloud/cli.py checks     # run the 16-check suite
```

---

## API

Please measure the voltage between TX and GND. In case of a logic level of 5V you should add a voltage divider or logic level converter between the ESP and the Victron device.

```
                UART-TTL
┌────────────────┐                ┌──────────────────┐
│           GND o│<-------------->│o GND             │
│ Victron    TX o│--------------->│o D7   ESP32/     │
│ Charger    RX o│                │       ESP8266    │<-- GND
│            5V o│                │                  │<-- 3.3V
└────────────────┘                └──────────────────┘

*Section quoted from the upstream readme.*
---

## Dependencies

| Metric | Value |
|--------|-------|
| Ecosystem | Python |
| Manifests detected | pyproject.toml |
| Files in snapshot | 61 |
| Lines of code | 3498 |
| Dependency references | 0 |
| Upstream license | MIT |
| Overlay license | Anticommons 0.1.0 |

Pinned lockfile: `anticloud/requirements.lock` (hash-pinned, PEP 508). SBOM: `sbom.cdx.json` (CycloneDX 1.5, pinned to the upstream SHA).

---

## Configuration

![adum1201_](https://github.com/KinDR007/VictronMPPT-ESPHOME/assets/61905170/9c7cf60c-fb19-404a-9c6e-7a17bc463ead)

[Buy on Ebay](https://www.ebay.com/itm/176094311751?var=475546476315&norover=1&mkevt=1&mkrid=711-168613-066356-2&mkcid=2&itemid=475546476315_176094311751&targetid=295222477609&device=c&mktype=pla&googleloc=1003785&poi=&campaignid=20797276730&mkgroupid=155163397599&rlsatarget=pla-295222477609&abcId=&merchantid=119648210&gad_source=1&gbraid=0AAAAAD_QDh8nutO0oonh8hgwFzBcKUcJc&gclid=Cj0KCQjw-ai0BhDPARIsAB6hmP6RN5aHtdBKIpl4EERR_dzQcCZcNqUsNd9KJGAJuTSsmuLnHL7lLs4aAnRtEALw_wcB)

[Buy on Aliexpress](https://www.aliexpress.com/item/1005005362881990.html?spm=a2g0o.productlist.main.55.22b97SeK7SeKfK&algo_pvid=8aa6f818-a16f-465d-a50d-d974fbbda803&algo_exp_id=8aa6f818-a16f-465d-a50d-d974fbbda803-27&pdp_npi=4%40dis%21CZK%2153.02%2129.24%21%21%212.23%211.23%21%40210387a117203472445525945efb01%2112000032744813603%21sea%21CZ%210%21AB&curPageLogUid=lG8tIpV8MVs7&utparam-url=scene%3Asearch%7Cquery_from%3A)

---

*Section quoted from the upstream readme.*
Overlay configuration (Anticloud):

- `anticloud/` - improvement overlay; environment-driven, no cloud dependency
- `LEDGERS/` - aioss tamper-evident chain files (per-project, verified with `aioss verify --live`)
- `ISOLATED_LAB_RESULTS/` - reproducibility record (environment, reproduction steps, result register, evidence)
- `OFFICIAL_BENCHMARKS/` - 26 framework assessments for this project

---

## Contributing

Upstream contributions: fork the `MPPT` project, create a feature branch, and open a pull request against upstream. Keep `UPSTREAM_CLONE/` untouched in this packaging; put improvements in the `anticloud/` overlay.

Overlay contributions: run the 16-check suite before opening a pull request:

```sh
python anticloud/bench/runner.py --cwd anticloud
```

---

## License

**Upstream license: MIT** (evidence: `LICENSE` in the upstream snapshot).

License file excerpt:

```text
MIT License

Copyright (c) 2021 KinDR007

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
```

### Anticommons 0.1.0 overlay

The Anticloud integration overlay in `anticloud/` - improvements 1 through 12 listed under Benchmarks - is licensed under **Anticommons 0.1.0**. Upstream code remains under its original MIT terms. See `ANTICOMMONS_LICENSE.md` in this directory for the overlay terms and contact.

SPDX: `MIT` (upstream) + Anticommons 0.1.0 (overlay, dual).

---

## Upstream

- **Project:** `MPPT` (category: SOLAR)
- **Upstream URL:** https://github.com/KinDR007/VictronMPPT-ESPHOME
- **Pinned commit (SHA):** `f9cd94f9409fce853123510f3c3be40c8b1d9ec6`
- **Branch:** main
- **Pin provenance:** GitHub API commits/main. The parent-project stamp is explicitly rejected for this project.
- **Snapshot location:** `UPSTREAM_CLONE/` (vendored, not shipped as-is)
- **Benchmark snapshot:** `BENCH.json`

---

## Benchmarks

Measured by the Anticloud assurance suite. Every value below is read from this
project's `BENCH.json`, produced by a real run — the SHA3-256 of that file is
`1bf24f741d9b9bb79dc1e52a42f0dbae422fd4611575f71ef7c53f736920a22d`.

| Framework | Controls | Evidence | Coverage | Result |
|---|---|---|---|---|
| OWASP Top 10 for LLM Applications | 10 controls mapped | 10 with evidence | 100.0% | PASS |
| OWASP Top 10 (2021) | 9 controls mapped | 9 with evidence | 100.0% | PASS |
| SOC 2 Type II readiness | 9 controls mapped | 9 with evidence | 100.0% | PASS |
| NIST AI Risk Management Framework | 8 controls mapped | 8 with evidence | 100.0% | PASS |
| NIST SP 800-53 Rev. 5 | 12 controls mapped | 12 with evidence | 100.0% | PASS |
| NIST Cybersecurity Framework 2.0 | 8 controls mapped | 8 with evidence | 100.0% | PASS |
| FedRAMP Rev. 5 | 10 controls mapped | 10 with evidence | 100.0% | PASS |
| PCI DSS v4.0.1 | 11 controls mapped | 11 with evidence | 100.0% | PASS |
| ISO/IEC 27001:2022 | 9 controls mapped | 9 with evidence | 100.0% | PASS |
| MITRE ATT&CK v16 | 12 controls mapped | 12 with evidence | 100.0% | PASS |
| ML Technology Readiness Level | TRL 8 | 8/8 criteria | | PASS |

**Overall: 16/16 checks passing.**

See `ISOLATED_LAB_RESULTS/03_Result_Register.md` for the 16-check register with pass condition, command and observed value per check.

Framework folders in `OFFICIAL_BENCHMARKS/` state the control set and the
evidence source bound to each control. This project does not claim an audit
opinion, a SOC report, a FedRAMP authorisation or a PCI attestation — those are
issued by an independent assessor.



## Archives and Permanent Records

| Platform | Identifier | Volume |
|---|---|---|
| Harvard Dataverse | DOI 10.7910/DVN/YMJKOG | 145 citable datasets |
| AIOSS verification kit | DOI 10.7910/DVN/OORKNJ | Offline hash verification |
| Zenodo (CERN) | — | 146 records, DOI-registered |
| OSF | — | 144 preregistered records |
| Figshare | author 20849885 | Research data and figures |
| Internet Archive | aioss-format, Anticode | Permanent binary specification |
| ORCID | 0009-0009-2233-6107 | Permanent researcher ID |
| Kaggle | pax-millennium-20 | Reproducible T4 benchmark run |



## Press and Independent Publication

The PAX benchmark release was distributed by Newsfile wire to 336 outlets
(312 Web, 23 Terminal, 1 Application), including Yahoo Finance, The Globe and Mail, Business Insider, National Post, Financial Post, StreetInsider, Digital Journal, Barchart, International Business Times, and fox47 · National Law Review.
Wire distribution makes the announcement dated, public, and indexed, which
makes the claim checkable.

