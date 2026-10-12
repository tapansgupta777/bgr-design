# CMOS Bandgap Reference (BGR) — LTspice Design, Calculations and Temperature Analysis

An educational analog-design project exploring a self-biased bandgap-reference core, PTAT/CTAT behavior, MOSFET current mirrors, SPICE operating-point debugging, and temperature-sweep analysis.

> **Status:** The final documented simulation uses `R1 = 3.604 kΩ` and `R2 = 117 kΩ`. The plotted output is approximately **1.0833–1.0901 V** over −50 °C to 125 °C. This is a useful topology-level simulation result, but it does **not** meet the original ~1.189 V target. The BJT devices are generic LTspice `2N3904` models, not verified TSMC 180 nm parasitic-BJT models. The design has not been validated in silicon or across process corners.

## 1. Project goals and setup

| Parameter | Design intent / configuration |
|---|---:|
| Supply voltage | 1.8 V |
| Original reference target | ~1.189 V |
| Nominal core-current target | ~5 µA |
| BJT area ratio | `N = 2` (approximated using two parallel 2N3904 instances for the larger-area branch) |
| Temperature sweep | −50 °C to 125 °C |
| Simulator | LTspice 24.1.10 |
| MOSFET models | `CMOSN` and `CMOSP` Level-49 BSIM3 models from the supplied model file |
| BJT models | Generic LTspice `2N3904` NPN devices used as placeholders |
| Final plotted resistors | `R1 = 3.604 kΩ`, `R2 = 117 kΩ` |
| Initial NMOS sizing | `W = 5.42 µm`, `L = 5 µm` |
| Initial PMOS sizing | `W = 12.83 µm`, `L = 5 µm` |

The file supplied as `tsmc180nm.lib` contains MOS model declarations but no process-specific BJT models. Its filename alone does not establish that it is an official TSMC PDK; the file header should be checked for provenance and redistribution permissions before making that claim or publishing it.

## 2. Circuit concept

The circuit combines:

- A BJT pair with an intended effective area ratio of two to generate a base-emitter voltage difference, `ΔVBE`, with a positive temperature coefficient (PTAT).
- `R1` to convert the PTAT-related voltage into a current.
- NMOS bias devices and a PMOS current mirror to establish and replicate the branch currents.
- A diode-connected BJT (`Q4`) and `R2` in the output branch to add a resistor-generated PTAT contribution to the BJT CTAT voltage.

The first-order BGR relationship is

\[
V_{REF} \approx V_{BE,Q4} + I_{R2}R_2.
\]

If the output current approximately follows the PTAT core current, and the voltage across `R1` is approximately `ΔVBE`, then

\[
I_{PTAT} \approx \frac{\Delta V_{BE}}{R_1},
\qquad
V_{REF} \approx V_{BE,Q4} + \frac{R_2}{R_1}\Delta V_{BE}.
\]

These are **first-order equations**, not exact equations for the complete circuit. The actual operating point also depends on the MOSFET bias network, resistor terminal voltages, body effect, current-mirror error, BJT model parameters, and transistor operating regions.

## 3. Hand calculations

### 3.1 Thermal voltage at 27 °C

Thermal voltage is

\[
V_T = \frac{kT}{q},
\]

where:

- Boltzmann constant: `k = 1.380649 × 10⁻²³ J/K`
- Elementary charge: `q = 1.602176634 × 10⁻¹⁹ C`
- Nominal calculation temperature: `T = 27 °C = 300.15 K`

Substitution gives

\[
V_T = \frac{(1.380649\times10^{-23})(300.15)}{1.602176634\times10^{-19}}
\approx 25.865\text{ mV}.
\]

### 3.2 Expected PTAT voltage for area ratio `N = 2`

For a BJT at a given temperature,

\[
V_{BE} \approx V_T\ln\left(\frac{I_C}{I_S}\right).
\]

At equal current density, an effective emitter-area ratio `N` gives

\[
\Delta V_{BE}=V_T\ln(N).
\]

For `N = 2` at 27 °C,

\[
\Delta V_{BE} = 25.865\text{ mV}\times\ln(2)
\approx 17.928\text{ mV}.
\]

The ideal first-order expectation is therefore **about 17.93 mV at 27 °C**. Two parallel discrete-transistor model instances approximate a doubled-area device only to the extent that the model instances share current as assumed; they are not a substitute for a characterized on-chip BJT layout/model.

The ideal slope of this PTAT voltage is

\[
\frac{d(\Delta V_{BE})}{dT}=\frac{k}{q}\ln(2)
\approx 59.7\text{ µV/K}.
\]

This slope applies to the ideal `ΔVBE` term, not directly to the whole output reference.

### 3.3 First-pass calculation of `R1`

For a nominal PTAT current of `5 µA`,

\[
R_1 \approx \frac{\Delta V_{BE}}{I_{PTAT}}
=\frac{17.928\text{ mV}}{5\text{ µA}}
\approx 3.586\text{ kΩ}.
\]

The selected value was `R1 = 3.604 kΩ`. Under the ideal assumption that exactly `17.928 mV` appears across `R1`,

\[
I_{PTAT,ideal}=\frac{17.928\text{ mV}}{3.604\text{ kΩ}}
\approx 4.975\text{ µA}.
\]

**Important:** `R1` is connected between circuit nodes influenced by the NMOS bias network and the BJT core. Its actual terminal voltage is not guaranteed to equal the ideal `ΔVBE`; therefore `4.975 µA` is a sizing estimate, not a promised SPICE result.

### 3.4 First-pass MOSFET sizing

The initial dimensions were estimated using the long-channel square-law saturation relation

\[
I_D\approx\frac{1}{2}\mu C_{ox}\frac{W}{L}V_{OV}^{2}.
\]

Rearranging,

\[
\frac{W}{L}\approx\frac{2I_D}{\mu C_{ox}V_{OV}^{2}}.
\]

Assumptions used for the initial estimate:

- `ID = 5 µA`
- `VOV = 0.2 V`
- `μnCox = 230.6 µA/V²`
- `μpCox = 97.4 µA/V²`
- `L = 5 µm`

**NMOS:**

\[
\frac{W}{L}=\frac{2(5\text{ µA})}{(230.6\text{ µA/V}^2)(0.2\text{ V})^2}
=\frac{10}{9.224}\approx1.084.
\]

\[
W\approx1.084(5\text{ µm})=5.42\text{ µm}.
\]

**PMOS:**

\[
\frac{W}{L}=\frac{2(5\text{ µA})}{(97.4\text{ µA/V}^2)(0.2\text{ V})^2}
=\frac{10}{3.896}\approx2.567.
\]

\[
W\approx2.567(5\text{ µm})=12.83\text{ µm}.
\]

| Device type | `μCox` assumption | Calculated `W/L` | Selected starting size |
|---|---:|---:|---:|
| NMOS | 230.6 µA/V² | 1.084 | `W = 5.42 µm`, `L = 5 µm` |
| PMOS | 97.4 µA/V² | 2.567 | `W = 12.83 µm`, `L = 5 µm` |

These dimensions are starting estimates only. BSIM3 model behavior includes effects not represented in the square-law equation; use the model's operating-point parameters to check actual current, headroom, and operating region.

### 3.5 First-order output-voltage and resistor-ratio calculation

The branch-level output equation is

\[
V_{REF}=V_{BE,Q4}+I_{R2}R_2.
\]

For the final plotted values,

\[
\frac{R_2}{R_1}=\frac{117\text{ kΩ}}{3.604\text{ kΩ}}\approx32.464.
\]

Using the ideal `ΔVBE` from Section 3.2, the scaled PTAT term would be

\[
V_{PTAT,scaled}\approx32.464(17.928\text{ mV})\approx582.0\text{ mV}.
\]

This demonstrates the intended design mechanism: scale a small PTAT voltage and add it to the CTAT `VBE` term. It is **not** a precise prediction for the simulated output because the actual voltage across `R1` differs from the ideal `ΔVBE`, and the currents and device models are nonideal.

### 3.6 Resistor estimate for the original target (fixed-current approximation)

At a fixed operating point, the output equation can be rearranged to estimate `R2`:

\[
R_2\approx\frac{V_{REF,target}-V_{BE,Q4}}{I_{R2}}.
\]

An **intermediate operating-point snapshot**, taken with `R2 = 137 kΩ` before the final PMOS-bulk correction, reported:

- `V(BGR) = 1.18391 V`
- `V(Q4 base/collector) ≈ 0.51728964 V`
- `I(R2) = 4.86584 µA`

Using those intermediate values solely as a fixed-current estimate for a `1.189 V` target:

\[
R_{2,target}\approx\frac{1.189-0.51728964}{4.86584\times10^{-6}}
\approx138.05\text{ kΩ}.
\]

This calculation explains the approximate nominal-voltage effect in that intermediate circuit state. It **must not be interpreted as the required resistor for the final circuit**, because the PMOS bulk wiring was subsequently corrected and the operating point changed. The calculation also does not address temperature compensation.

## 4. Final documented simulation result

The final displayed temperature-sweep plot uses `R1 = 3.604 kΩ` and `R2 = 117 kΩ`. Approximate values read from the plot are:

| Metric | Final plotted result (approx.) |
|---|---:|
| Temperature range | −50 °C to 125 °C |
| Minimum `V(BGR)` | 1.0833 V |
| Maximum `V(BGR)` | 1.0901 V |
| Peak-to-peak spread | 6.8 mV |
| Output near room temperature | ~1.090 V |
| Original target | 1.189 V |
| Approximate room-temperature shortfall | 99 mV, or ~8.3% |

The min/max are approximate readings from the screenshot, not precision measurements exported from the raw data. For a final report, use LTspice waveform cursors or export the waveform data and calculate the extrema numerically.

### 4.1 Peak-to-peak spread

\[
\Delta V_{pp}=V_{max}-V_{min}
\]

\[
\Delta V_{pp}\approx1.0901-1.0833=0.0068\text{ V}=6.8\text{ mV}.
\]

### 4.2 Relative error from the target

Using an approximate room-temperature output of `1.090 V`,

\[
E_{abs}=1.189-1.090=0.099\text{ V}=99\text{ mV}.
\]

\[
E_{rel}=\frac{1.189-1.090}{1.189}\times100\%
\approx8.3\%.
\]

The final plotted circuit therefore **does not meet the original 1.189 V target**.

### 4.3 Descriptive range-normalized drift

Because the curve peaks in the interior of the temperature range, a single linear temperature coefficient would not fully describe its curvature. A descriptive range-normalized metric is

\[
TC_{range}=\frac{V_{max}-V_{min}}{V_{nom}(T_{max}-T_{min})}\times10^6\quad\text{ppm/°C}.
\]

With a representative `Vnom = 1.087 V`, a `6.8 mV` spread, and a `175 °C` span,

\[
TC_{range}\approx\frac{0.0068}{1.087(175)}\times10^6
\approx35.7\text{ ppm/°C}.
\]

This is a range-normalized summary metric, **not** a guaranteed linear or datasheet-style temperature coefficient. Quote the peak-to-peak spread and temperature span alongside it.

## 5. Simulation debugging record

The following sequence captures the main issues found during debugging. Early measurements below belong to intermediate schematic states and are not the final results.

### 5.1 Initial output near the supply rail

An early `.op` run at `R2 = 97.8 kΩ` reported:

- `V(BGR) = 1.73677 V`
- `V(Q4 base/collector) = 0.54112 V`
- `I(R2) = 12.2255 µA`
- `I(R1) = 5.77518 µA`

From the resistor equation,

\[
I_{R2}=\frac{V_{BGR}-V_{Q4}}{R_2}
=\frac{1.73677-0.54112}{97.8\text{ kΩ}}
\approx12.23\text{ µA}.
\]

Likewise, the displayed R1 terminals were approximately `524.606 mV` and `503.792 mV`, giving

\[
I_{R1}\approx\frac{524.606-503.792\text{ mV}}{3.604\text{ kΩ}}
\approx5.77\text{ µA}.
\]

The output current was therefore more than twice the core current in that early state.

### 5.2 PMOS mirror gate and terminal connections

The early generated netlist included a line equivalent to

```spice
M2 Vdd N005 BGR N003 CMOSP l=5u w=12.83u
```

The four-terminal MOS instance order is `D G S B`. `N005` was an otherwise unused gate node, so the output PMOS was not receiving the intended shared mirror gate bias. The gate was connected to the mirror-bias net during correction.

For a conventional high-side PMOS mirror, verify from the generated netlist that:

- the source terminals connect to `VDD`;
- the bulk terminals connect to `VDD`;
- the mirror gates share the intended bias node;
- the diode-connected reference PMOS has its gate tied to its drain.

Do not judge pin connectivity by visual symbol orientation alone. Inspect the generated SPICE netlist (`D G S B`) after each significant wiring change.

### 5.3 MOSFET bulk corrections

Two additional connection errors were found during the process:

- NMOS bulk terminals had been connected to their source nodes rather than ground; they were corrected to ground.
- PMOS bulk terminals had been connected to drain nodes rather than `VDD`; they were corrected to `VDD`.

These are important because body junctions and body effect influence the bias point. After correcting a MOSFET's gate/source/drain/bulk wiring, rerun `.op` and the temperature sweep; earlier plots are not interchangeable with later ones.

### 5.4 Branch-current comparison

The current traces `I(R1)` and `I(R2)` were plotted together after the mirror wiring fixes. Their curves overlapped closely in the shown simulation, supporting the conclusion that the output branch followed the core current over the sweep. This is a useful topology-level check, but it does not prove that the PTAT/CTAT weighting gives the required output voltage or temperature stability.

### 5.5 Resistor exploration

Several `R2` values were explored during debugging, including `97.8 kΩ`, approximately `115 kΩ`, `137 kΩ`, and finally `117 kΩ`. Some of the earlier measurements were made before both MOS bulk connections were corrected. For that reason, do not compare every earlier curve as though only R2 had changed. The final documented result is the curve for `R2 = 117 kΩ` after the connection corrections.

## 6. How to reproduce and verify

1. Open the final LTspice schematic. Ensure the PMOS sources and bulks are connected to `VDD`, NMOS bulks are connected to ground, and the PMOS mirror gates are connected as intended.
2. Ensure the MOS model include path resolves locally. Do not publish the model file unless its origin and redistribution license allow it.
3. Run operating-point analysis (`.op`). Record `V(BGR)`, `I(R1)`, `I(R2)`, the BJT `VBE` nodes, and the MOSFET operating-region/headroom values.
4. Run the same temperature sweep from −50 °C to 125 °C used to generate the documented curve. Keep the actual sweep directive and step setting with the project.
5. Use waveform cursors or export the data to calculate the exact minimum, maximum, peak-to-peak spread, and temperature of the peak. Update the approximate values in this README if precise data are collected.
6. Save the schematic and final waveform screenshot/data. Record any future simulation only after saving the corresponding resistor values and netlist.

## 7. Model caveats and limitations

- The supplied MOS file contains `CMOSN` and `CMOSP` model declarations but no suitable process-specific BJT model. Generic LTspice `2N3904` devices were used as placeholders.
- The effective `N = 2` area ratio is approximated by two parallel generic BJT instances. This does not substitute for a characterized parasitic-BJT model and real layout.
- The final simulated output is ~1.09 V rather than 1.189 V. The relatively small plotted temperature spread does not mean both requirements are satisfied.
- This simulation does not include validated process corners, mismatch/Monte Carlo, supply sensitivity, startup robustness, extracted parasitics, layout, or silicon measurement.
- The MOS model file's provenance should be verified before describing it as an official TSMC 180 nm PDK. Its filename alone is not sufficient evidence.

## 8. Skills and concepts demonstrated

- Analog circuit implementation and debugging
- Bandgap-reference fundamentals: PTAT and CTAT contributions
- BJT `ΔVBE` calculations and current-density/area-ratio concepts
- MOSFET square-law sizing estimates and BSIM-model-aware verification
- PMOS current-mirror biasing and MOS terminal/bulk connectivity
- LTspice operating-point analysis and temperature sweeps
- Branch-current comparison, resistor tuning, error calculation, and model limitation analysis

## 9. GitHub usage

Save this file as `README.md` in the project root. Add your final LTspice schematic separately if it may be shared. Keep proprietary or uncertain-license model files out of the public repository.

Example commands from the repository directory:

```bash
git status
git add README.md Draft10.asc
git commit -m "Document BGR simulation calculations and temperature analysis"
git push
```

If the schematic has a different filename, substitute that filename. Review `git status` before committing, and do not add `tsmc180nm.lib` unless redistribution is permitted.
