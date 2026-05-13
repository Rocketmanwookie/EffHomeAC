# Multi-Zone HVAC System — BMS Package

**Project:** 16-Zone Commercial Building HVAC  
**Revision:** A | **Date:** 2026-05-13 | **Author:** Copilot BMS Generator  
**BACnet Protocol:** BACnet/IP + 4× MSTP segments  

---

## Package Contents

| File | Description |
|------|-------------|
| `hvac_system.xml` | Master HVAC document — all zones, plant, supervisor, GlobalBOM |
| `hvac_editor.py` | Python CLI + importable editor for the XML |
| `hvac_bom.csv` | Standalone flattened Bill of Materials (all sections) |
| `hvac_functional_chart.png` | System architecture + BOM cost breakdown charts |
| `INSTALL.md` | Environment setup and dependency installation |
| `TESTING.md` | Full test plan with unit, integration, and CLI test cases |
| `README.md` | This file |

---

## System Overview

```
┌─────────────────────────────────────────────────────────┐
│              EC-Net 4 BAS Supervisor  (192.168.1.10)     │
│           BACnet/IP | BACnet MSTP ×4 | Port 47808        │
└────────┬──────────────┬──────────────┬──────────────────┘
         │              │              │
    MSTP-1/2       MSTP-3/4       Plant Network
   Z01–Z08        Z09–Z16       AHU-01 + HP-01

:Distech - EC-Net 4 BAS Supervisor:
https://docs-be.distech-controls.com/bundle/EC-Net-4_SS/raw/resource/enus/EC-Net%204_SS.pdf
```

### Zone Types
| Type | Zones | Controller | Damper Actuator |
|------|-------|-----------|-----------------|
| VAV  | Z01–Z05, Z07–Z10, Z15–Z16 | Distech ECB-VAV | Belimo LMCB24-SR |
| CAV  | Z06, Z11–Z12 | Distech ECB-CAV | Belimo GMB24-MFT |
| EXH  | Z13–Z14 | Distech ECB-PTU | Belimo FSNF230-S |

### Plant Equipment
| ID | Description | Design Capacity |
|----|-------------|----------------|
| AHU-01 | Carrier 39MN060 Air Handler | 20,000 CFM, 2.5 in-WC |
| HP-01  | Carrier 30XWH150 Heat Pump  | 40 Tons, COP 3.8 |

### Global Setpoints (defaults)
| Parameter | Value |
|-----------|-------|
| Occupied Cooling SP | 74.0 °F |
| Occupied Heating SP | 70.0 °F |
| Supply Air Temp SP | 55.0 °F |
| Duct Static Pressure | 1.5 in-WC |
| CHW Supply Temp | 44.0 °F |
| HW Supply Temp | 140.0 °F |
| CO₂ Setpoint | 1,000 ppm |
| Economizer Lockout OAT | 65.0 °F |

:Distech - ECB-VAV:
https://docs-be.distech-controls.com/bundle/ECB-VAV_SP/raw/resource/enus/ECB-VAV_SP.pdf

:Distech - ECB-CAV:
https://www.bing.com/ck/a?!&&p=eaf4d8ac7dea220aa84e149c0d42784969a49e5224106d72c459e41af0e4b702JmltdHM9MTc3ODYzMDQwMA&ptn=3&ver=2&hsh=4&fclid=14a2a4eb-fc94-6ec8-06b3-b3bbfdaa6f1d&psq=Distech+ECB-CAV&u=a1aHR0cHM6Ly9kb2NzLWJlLmRpc3RlY2gtY29udHJvbHMuY29tL2J1bmRsZS9FQ0ItU2VyaWVzX0lHL3Jhdy9yZXNvdXJjZS9lbnVzL0VDQiUyMFNlcmllc19JRy5wZGY_c2F2ZV9sb2NhbD10cnVl

:Distech - ECB-PTU:
https://www.distech-controls.com/products/detail/947846/distech-controls/ecb-ptu-series

:Belimo - LMCB24-SR:
https://www.belimo.com/us/shop/en_US/p?code=LMCB24-SR

:Belimo - GMB24-MFT:
https://www.belimo.com/us/shop/en_US/p?code=GMB24-MFT

:Belimo - FSNF230-S:
https://www.belimo.com/us/shop/en_US/p?code=FSNF230-S+US

:Carrier - 39MN060:
https://www.carrier.com/marine-offshore/en/worldwide/products/air-treatment-ventilation/air-handling-units/39m/

:Carrier - 30XWH150:
---

## Quick Start

```bash
# List all 16 zones
python hvac_editor.py --file hvac_system.xml --list-zones

# Adjust Conference Room 1 (Z04) setpoints
python hvac_editor.py --file hvac_system.xml --zone Z04 \
  --set-htg-sp 71.0 --set-clg-sp 73.5 --save-as hvac_updated.xml

# Change AHU supply air setpoint
python hvac_editor.py --file hvac_system.xml \
  --plant-ahu --set-tag AHU-SP-SAT --value 54.0 --save-as hvac_updated.xml

# Rebuild the Global BOM after any edits
python hvac_editor.py --file hvac_system.xml --rebuild-bom --save-as hvac_updated.xml

# Export BOM to CSV
python hvac_editor.py --file hvac_system.xml --export-bom-csv hvac_bom_export.csv
```

---

## BACnet Point Addressing

| Section | BACnet ID Range |
|---------|----------------|
| Zone Z01 | 1101–1127 |
| Zone Z02 | 1201–1227 |
| … | … (base + zone index × 100) |
| Zone Z16 | 2601–2627 |
| AHU-01 Tags | 3001–3030 |
| HP-01 Tags | 4001–4025 |
| Supervisor | Device ID 1000 |

---

## Occupancy Schedule (default)

| Day | Occupied Window |
|-----|----------------|
| Monday–Thursday | 06:00–20:00 |
| Friday | 06:00–18:00 |
| Saturday | 08:00–14:00 |
| Sunday | Unoccupied |

---

## Alarm Priority Matrix

| Alarm | Priority |
|-------|----------|
| Freeze Stat / Smoke Detector | Critical |
| HP Alarm / AHU Fan Fault | High |
| Filter Dirty / Zone Controller Alarm | Medium |
| High CO₂ / Schedule Override | Low / Info |

---

## License
Internal use only — not for redistribution.  
© 2026 Grant Engineering.
