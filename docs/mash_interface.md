# MASH 1-1-1 Interface Specification

**Project:** LinHT-RFIC PLL  
**Technology:** IHP SG13CMOS5L  
**Block:** MASH 1-1-1 Delta-Sigma Modulator  
**Document Status:** Draft – Pending Architecture Review  
**Purpose:** Define the MASH digital interface and its integration requirements with the PLL Multi-Modulus Divider (MMD).

---

## 1. Scope

This document specifies the proposed interface between the MASH 1-1-1 delta-sigma modulator and the PLL divider.

The MASH modulator generates a digital sequence that controls the instantaneous division ratio of the MMD. This allows the PLL to achieve a fractional average division ratio while using an integer multi-modulus divider.

The scope of this task includes:

- Defining the MASH input and output signals.
- Establishing the proposed input widths and numerical formats.
- Defining clock and reset requirements.
- Describing the MASH-to-MMD interface.
- Identifying unresolved architectural decisions.

This document is an interface proposal. Parameters that have not been confirmed by the PLL architect are explicitly marked TBD.

## 2. PLL and MMD Context

The LinHT-RFIC PLL uses a fractional-N architecture with an LC-VCO and a MASH 1-1-1 delta-sigma modulator.

The current PLL proposal specifies:

| Parameter | Proposed value |
|---|---|
| PLL architecture | Fractional-N |
| Delta-sigma architecture | MASH 1-1-1 |
| MMD division range | 64–127 |
| Fractional input resolution | 20 bits |
| Nominal PFD/reference frequency | 32 MHz |
| MASH output correction range | −3 to +4 (proposed) |

The MASH output controls the MMD division modulus over successive cycles. The resulting average division ratio provides the required fractional frequency synthesis.

The correction range and divider operating limits must be verified together before the interface is finalized.

## 3. Functional Description

The MASH 1-1-1 consists of three cascaded first-order accumulator stages with digital noise-cancellation logic.

The fractional input is accumulated to generate a digital output sequence. This sequence represents the fractional contribution to the average division ratio.

The intended average division ratio is:

```text
N_avg = N_integer + F
```

For a 20-bit fractional input:

```text
F = frac_word / 2^20
```

Therefore:

```text
N_avg = N_integer + (frac_word / 1048576)
```

Where:

- `N_integer` is the integer portion of the division ratio.
- `frac_word` is the unsigned fractional input.
- `F` is the fractional portion of the division ratio.
- `N_avg` is the long-term average division ratio.

The MASH output sequence determines the instantaneous correction applied to the integer division value.

## 4. MASH Interface Definition

### 4.1 Proposed Top-Level Signals

| Signal | Direction | Width | Description |
|---|---|---:|---|
| `clk` | Input | 1 | MASH operating clock |
| `rst_n` | Input | 1 | Active-low reset |
| `frac_word` | Input | 20 | Unsigned fractional control word |
| `N_integer` | Input | TBD | Integer division value |
| `mash_out` | Output | TBD | MMD correction/control output |

The interface intentionally does not include `enable`, `valid`, or configuration-update signals at this stage. Their necessity depends on the agreed PLL integration protocol.

### 4.2 Fractional Input

The proposed fractional input is:

```text
frac_word[19:0]
```

Format:

```text
Unsigned Q0.20
```

Its numerical range is:

```text
0 <= F <= 0.9999990463256836
```

Representative input values:

| `frac_word` | Fractional value |
|---|---:|
| `20'h00000` | 0 |
| `20'h00001` | 1 / 2^20 |
| `20'h40000` | 0.25 |
| `20'h80000` | 0.5 |
| `20'hC0000` | 0.75 |
| `20'hFFFFF` | 1 − 2^-20 |

The fractional input is expected to remain stable during normal modulation unless dynamic updates are explicitly supported.

**TBD:** Confirm whether the fractional input can be changed during operation and how such updates should be synchronized.

### 4.3 Integer Division Input

`N_integer` provides the integer portion of the PLL division ratio.

Its width and valid input range depend on the MMD implementation and the required fractional operating range.

**TBD:**

- Confirm the integer input width.
- Confirm the permitted integer division values.
- Confirm whether the integer and fractional inputs are sampled continuously or latched during configuration.

### 4.4 MASH Output

The output `mash_out` provides the correction information used by the MMD.

The current proposed correction range is:

```text
-3 <= C_MASH <= +4
```

The conceptual divider relationship is:

```text
N_instantaneous = N_integer + C_MASH
```

This describes the intended correction behavior, not a finalized RTL interface equation. The actual relationship depends on the output encoding and the MMD implementation.

The output must represent all eight possible correction values.

Possible representation choices include:

- A 3-bit unsigned offset encoding, with a defined mapping between codes and corrections.
- A signed representation, requiring sufficient width to represent both −3 and +4.

A conventional 3-bit two's-complement value cannot represent +4. Therefore, the output width and encoding must be explicitly agreed upon.

**TBD:**

- Confirm whether the correction range of −3 to +4 is final.
- Confirm the output width.
- Confirm signed, unsigned-offset, or other encoding.
- Confirm the mapping from output code to divider correction.
- Confirm whether the MMD applies the correction directly or requires additional decoding.

## 5. MASH-to-MMD Integration

The MASH output drives the MMD control logic.

At each relevant divider update, the MMD uses the integer division value and the MASH correction to select its instantaneous modulus.

The proposed MMD division range is:

```text
64 <= N_instantaneous <= 127
```

The integration must ensure that every applied modulus remains within the supported range.

### 5.1 Divider Range Compatibility

The proposed MASH correction range is −3 to +4.

The relationship between the nominal integer division range and the instantaneous MMD range must be checked.

For example, if the integer division value were 67:

```text
Minimum modulus = 67 - 3 = 64
Maximum modulus = 67 + 4 = 71
```

However, applying the full correction range to an integer value of 124 would produce:

```text
Maximum modulus = 124 + 4 = 128
```

This exceeds the proposed MMD maximum of 127.

Therefore, the permitted integer input range may need to be restricted, or the correction behavior must be adjusted near the boundaries.

**TBD:** Confirm the valid integer division range and how boundary conditions are handled.

### 5.2 MMD Sampling

The MMD must sample the MASH correction at a well-defined point in its divider cycle.

The interface must establish:

- Whether the MASH output is registered.
- When the MMD samples the output.
- Whether the output is updated every MASH clock cycle.
- Whether an additional synchronization or holding register is required.
- Whether the MASH and MMD operate in the same clock domain.

These decisions are necessary to avoid applying a changing divider value at an unsafe point in the MMD cycle.

## 6. Clock Requirements

The MASH modulator requires a clock that is synchronous with the divider-control operation.

The project discussion has raised the reference/PFD clock and `F_XOSC` as clock-related considerations. The exact source must be agreed upon.

The proposed PLL reference/PFD frequency is 32 MHz.

The following points require confirmation:

1. Is the MASH clock directly driven by `F_XOSC`?
2. Is the MASH clock the PFD/reference clock?
3. Does the MMD use the same clock as the MASH?
4. Is the clock frequency fixed during operation?
5. What is the permitted clock-frequency range?

**TBD:** Confirm the clock source and its relationship to the MMD and PFD.

## 7. Reset Requirements

The proposed reset signal is:

```text
rst_n
```

It is active low.

The proposed reset strategy is:

- Asynchronous reset assertion.
- Synchronized reset deassertion.
- Initialization of all accumulator states to zero.
- Defined MASH output behavior during reset.

After reset release, the MASH must begin operating from a known internal state.

The MMD must not interpret an undefined MASH output as a valid divider correction.

**TBD:**

- Confirm whether asynchronous assertion and synchronized deassertion are required.
- Confirm the initial output value.
- Confirm whether the MASH needs an initialization period.
- Confirm whether the MMD must remain in a fixed-modulus mode during initialization.
- Confirm whether a separate enable or lock-related control is required.

## 8. LSB Dithering

The project discussion identifies LSB dithering as a consideration for the MASH implementation.

Dithering can be used to reduce periodic patterns and associated fractional spurs in the output sequence.

A possible implementation uses a pseudo-random sequence generated by an LFSR to perturb the least significant bit of the fractional input or an agreed internal MASH path.

The exact injection point must be selected carefully because different dither architectures can affect noise shaping and spur performance differently.

Dithering should not be assumed to be part of the finalized interface until its implementation is agreed upon.

**TBD:**

- Is LSB dithering required in the initial MASH implementation?
- Is an LFSR required?
- What is the LFSR width and polynomial?
- Where is the dither injected?
- Is the dither seed configurable or fixed?
- Does dithering affect the external interface or remain entirely internal?

## 9. Verification Requirements

The following verification cases are proposed for the MASH RTL and its MMD integration.

### 9.1 Reset Verification

- Assert and release reset.
- Verify all accumulator states initialize correctly.
- Check deterministic output behavior after reset.
- Verify that the MMD does not receive an invalid correction during initialization.

### 9.2 Fractional Input Verification

Test the following fractional input values:

| Test case | Input | Expected fraction |
|---|---|---:|
| Zero | `20'h00000` | 0 |
| Smallest | `20'h00001` | 0.000000953674 |
| Quarter | `20'h40000` | 0.25 |
| Half | `20'h80000` | 0.5 |
| Three-quarter | `20'hC0000` | 0.75 |
| Maximum | `20'hFFFFF` | 0.999999046326 |

### 9.3 Average Division Ratio

For each fractional input, evaluate the long-term average divider behavior.

The measured average should approach:

```text
N_avg = N_integer + (frac_word / 1048576)
```

The observation window must be sufficiently long to measure the average accurately.

### 9.4 MMD Integration

- Verify that every MASH output code is decoded correctly.
- Verify the resulting instantaneous divider modulus.
- Confirm that the MMD never receives a modulus outside its supported range.
- Test operation near the minimum and maximum divider limits.
- Verify correct synchronization between the MASH and MMD.

### 9.5 Dither Verification

If dithering is implemented:

- Verify the LFSR sequence.
- Verify the selected injection point.
- Compare output sequences with and without dithering.
- Evaluate the output spectrum for periodic spurs.
- Confirm that the average fractional division ratio is maintained.

## 10. Open Questions and Required Decisions

The following table summarizes the decisions required before freezing the interface.

| No. | Topic | Required decision |
|---:|---|---|
| 1 | Clock source | Confirm `F_XOSC` or PFD/reference clock |
| 2 | Clock frequency | Confirm the supported MASH clock frequency |
| 3 | Integer input | Confirm `N_integer` width and valid range |
| 4 | Fractional input | Confirm unsigned 20-bit Q0.20 format |
| 5 | Output correction | Confirm the proposed −3 to +4 range |
| 6 | Output width | Determine the required `mash_out` width |
| 7 | Output encoding | Define signed or offset encoding |
| 8 | MMD interface | Define the exact correction-to-modulus mapping |
| 9 | Divider boundaries | Resolve compatibility with the 64–127 range |
| 10 | Sampling | Define when the MMD samples `mash_out` |
| 11 | Reset | Confirm reset assertion and deassertion behavior |
| 12 | Initialization | Define startup behavior and initialization cycles |
| 13 | Input updates | Define whether runtime updates are supported |
| 14 | Dithering | Confirm whether LSB dithering is required |
| 15 | LFSR | Define the dither generator and injection point |
| 16 | Control signals | Confirm whether additional enable or valid signals are needed |

## 11. References

1. **LinHT-RFIC Project Repository**  
   https://github.com/M17-Project/LinHT-RFIC_sg130cmos5l

2. **PLL Proposal**  
   `macros/pll_top/doc/chipalooza_pll_proposal.md`

3. **PLL Macro Documentation**  
   `macros/pll_top/README.md`

4. **Project Design Plan**  
   `doc/design_plan.md`

5. **MASH 1-1-1 Background and Reference Implementation**  
   https://iic-jku.github.io/radio-frequency-integrated-circuits/content/pll/delta_sigma_modulator-preview.html

---

## 12. Deliverable Status

This document establishes the proposed MASH 1-1-1 interface and identifies the remaining integration decisions.

The interface should be frozen only after the PLL architect confirms the clock source, input formats, output encoding, MMD sampling behavior, divider boundaries, reset sequence, and dithering requirements.

**Next step:** Review the open questions with the PLL architecture team and update this specification based on the agreed decisions.
