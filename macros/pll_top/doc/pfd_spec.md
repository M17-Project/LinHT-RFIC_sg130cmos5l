# LinHT-RFIC_sg130cmos5l
## N-Fractional PLL: Phase and Frequency Detector Specification   

# PFD Specification

| # | Specification / Design Parameter | Value |
|---:|---|---|
| 1 | Process technology | IHP SG13CMOS5L, 130 nm SiGe BiCMOS |
| 2 | PFD supply voltage | 1.2 V |
| 3 | Operating temperature | −40 to 125 °C |
| 4 | Architecture | 2 × DFF + NAND reset |
| 5 | Inputs | REF, FB |
| 6 | Outputs | UP, DN |
| 7 | Logic voltage range | 0–1.2 V |
| 8 | Reference frequency range | 10–50 MHz |
| 9 | Nominal reference frequency | 32 MHz |
| 10 | Minimum PFD operating frequency | 10 MHz |
| 11 | Maximum PFD operating frequency | 50 MHz |
| 12 | Feedback frequency | TBD |
| 13 | VCO frequency | 2.2–2.6 GHz |
| 14 | Feedback divider | ÷64–127 |
| 15 | PFD input edge | Rising edge |
| 16 | Input VIH | TBD |
| 17 | Input VIL | TBD |
| 18 | Input rise time | TBD |
| 19 | Input fall time | TBD |
| 20 | Input capacitance | TBD |
| 21 | Output VOH | TBD |
| 22 | Output VOL | TBD |
| 23 | Output load capacitance | TBD |
| 24 | Output rise time | TBD |
| 25 | Output fall time | TBD |
| 26 | Output drive current | TBD |
| 27 | Minimum detectable phase difference | TBD |
| 28 | Maximum phase-error range | TBD |
| 29 | Minimum UP pulse width | TBD |
| 30 | Minimum DN pulse width | TBD |
| 31 | Maximum UP pulse width | TBD |
| 32 | Maximum DN pulse width | TBD |
| 33 | DFF clock-to-Q delay | TBD |
| 34 | NAND reset propagation delay | TBD |
| 35 | Maximum allowable reset delay | TBD |
| 36 | PFD dead zone | TBD |
| 37 | UP/DN propagation-delay mismatch | TBD |
| 38 | UP/DN pulse-width mismatch | TBD |
| 39 | UP/DN symmetry | TBD |
| 40 | Charge-pump input voltage requirement | TBD |
| 41 | Charge-pump input capacitance | TBD |
| 42 | Charge-pump input load | TBD |
| 43 | Required UP drive strength | TBD |
| 44 | Required DN drive strength | TBD |
| 45 | Maximum PFD power | TBD |
| 46 | Static power | TBD |
| 47 | Dynamic power at 32 MHz | TBD |
| 48 | Dynamic power at 50 MHz | TBD |
| 49 | Maximum PFD area | TBD |
| 50 | Target PFD area | TBD |
| 51 | NMOS device dimensions | TBD |
| 52 | PMOS device dimensions | TBD |
| 53 | DFF transistor sizing | TBD |
| 54 | NAND transistor sizing | TBD |
| 55 | Reset-path transistor sizing | TBD |
| 56 | Transistor minimum channel length | TBD |
| 57 | PVT process corners | TBD |
| 58 | Supply-voltage tolerance | TBD |
| 59 | Monte-Carlo / mismatch requirement | TBD |
| 60 | Post-layout parasitic extraction | Required |
| 61 | Post-layout functional verification | Required |
| 62 | DRC | Clean |
| 63 | LVS | Clean |
| 64 | PFD-to-charge-pump interface | TBD |
| 65 | PLL loop bandwidth | 75–300 kHz |
| 66 | Nominal PLL loop bandwidth | 150 kHz |
| 67 | Dead-zone mitigation | Required |
| 68 | PFD integration | `pll_top` submodule |
| 69 | PFD standalone verification | Required |
| 70 | PFD integrated PLL verification | Required |