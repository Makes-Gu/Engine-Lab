# Engine Lab: model and references

This is an educational, steady-operating-point model of a four-stroke reciprocating engine. It connects geometry, timing, heat release, heat loss, and useful work so that changes can be explored consistently. Its fuel presets and loss coefficients are representative teaching assumptions, not a calibration to a particular engine or fuel batch.

## Geometry and angle convention

All timing angles are crankshaft degrees. Compression top dead center is 0°; negative angles are before TDC, and positive angles are after TDC. The displayed four-stroke cycle runs from −360° to +360°. A cylinder completes one cycle in two crankshaft revolutions.

For bore `B`, stroke `S`, crank radius `a = S/2`, rod length `l`, and cylinder count `n`:

```text
piston area A = π B² / 4
swept volume per cylinder Vd = A S
total displacement = n Vd
clearance volume Vc = Vd / (compression ratio − 1)
piston travel from TDC x(θ) = a(1 − cos θ) + l − √(l² − a² sin² θ)
cylinder volume V(θ) = Vc + A x(θ)
```

The rod-ratio control is **connecting-rod length divided by crank radius**, not rod length divided by stroke. Fuel selection does not alter geometric compression ratio. Adding cylinders at fixed bore and stroke increases displacement; changing displacement while holding other dimensions fixed must change bore or stroke.

These relations agree with the engine-geometry formulation in the [Ansys Chemkin theory manual, Internal Combustion Engine Model](https://ansyshelp.ansys.com/public/Views/Secured/corp/v242/en/chemkin_th/i38654.html).

## Heat release and pressure

The cylinder is represented as one spatially uniform ideal-gas zone. During the closed portion of the cycle, pressure follows the ideal-gas relation and the first law:

```text
p V = m R T
dU = dQcombustion − dQwall − p dV
```

The implementation uses `R = 287 J/(kg K)`, constant `γ = 1.35`, and 0.5° integration steps. It releases 98% of the burned fuel's lower heating value into this modeled gas. These constants intentionally simplify changing gas composition, specific heats, and combustion completeness. The reported closed-cycle heat-balance residual is a numerical consistency check, not validation against a real engine.

The burn profile uses a prescribed Wiebe-type burned-fraction curve. The user supplies its start through timing and delay, and its duration through the burn-duration control. This is a convenient shape for distributing heat release over crank angle; it does not simulate individual spray droplets, flame fronts, or detailed chemical kinetics. The [Ansys Wiebe model description](https://ansyshelp.ansys.com/public/Views/Secured/corp/v242/en/chemkin_th/pgfId-1352744.html) explains that its shape parameters are normally fitted to measured engine data. No such engine-specific fit is claimed here.

Cooling is represented by a lumped gas-to-wall heat-loss model with an effective boundary temperature and a strength factor. The air/liquid/insulated choices represent different heat-loss assumptions. They are not a radiator, water-pump, thermostat, coolant-boiling, or transient warm-up simulation. A reported gas temperature is a bulk single-zone temperature, not a local flame temperature.

In **predicted pressure** mode, pressure is an output of geometry, heat input, timing, and cooling. In **target pressure** mode, the model searches over 0–100% fuel input to approach the requested peak pressure. It does not independently force the pressure curve to a chosen number. An unreachable target remains marked as unreachable and the nearest boundary run is shown.

Fuel-input percentage scales fuel mass at fixed trapped air. Its upper bound corresponds to equivalence ratio 0.8 for compression ignition and 1.0 for spark ignition. Therefore, **the same percentage with a different fuel need not supply the same fuel mass or chemical energy**: the fuel's stoichiometric air requirement changes the permitted mass. The energy-input and fuel-flow values help interpret these comparisons. Volumetric efficiency is implemented as an effective cylinder filling-pressure multiplier, rather than a full intake-flow calculation.

## Timing and fuels

For diesel, B20, and B100, the timing setting represents actual start of injection (SOI). Start of combustion (SOC) follows an estimated or manually specified ignition delay. For gasoline and ethanol blends, timing represents the spark event and the delay represents an illustrative spark-to-combustion interval. It is not a diesel cetane calculation.

```text
delay angle [°CA] = 0.006 × rpm × delay [ms]
SOC angle = injection or spark angle + delay angle
```

SOI, SOC, CA50, peak-pressure angle, and peak-torque angle are distinct events. CA10, CA50, and CA90 locate 10%, 50%, and 90% of the model's cumulative heat release. CA90−CA10 describes the main burn interval. A change in SOC need not produce an identical shift in pressure peak or CA50 if the burn shape also changes.

The automatic delay is a teaching correlation. Real biodiesel can have slower evaporation but faster ignition chemistry, and it does not always ignite earlier than petroleum diesel. A primary experiment discusses these competing contributions: [Pham et al., ignition delays of biodiesel–diesel blends](https://www.sciencedirect.com/science/article/pii/S0016236121011303). [MIT's diesel injection and ignition lecture](https://ocw.mit.edu/courses/2-61-internal-combustion-engines-spring-2017/1f6c5374f82f59c50c7f50a8eb5d6b82_MIT2_61S17_lec16.pdf) provides the physical background.

**Octane and cetane are different measures.** Octane characterizes resistance to knock in spark-ignition operation; cetane characterizes compression-ignition quality. Neither is simply an energy-content rating. Switching fuel changes the modeled ignition regime and property assumptions; it does not establish that a real engine could use that fuel without hardware and control changes.

### Petrol grade selection in the native app

For gasoline/E0 and E10, the native app records a separate petrol grade: **92, 95 or 98 RON**, or **87, 89, 91 or 93 AKI**. Grade and ethanol content are independent selections. Older configurations and new defaults use **Unspecified grade**. Diesel and the nominal E85 preset do not use this field; switching back to gasoline/E10 restores the retained grade.

RON is the research octane rating; U.S. pump AKI is `(RON + MON) / 2`, so a RON value cannot be converted to AKI without the motor octane rating. [EIA's octane explanation](https://www.eia.gov/energyexplained/gasoline/octane-in-depth.php) defines the U.S. scale; [Exxon lists 87/89/91–93 pump grades](https://www.exxonmobilfuels.com/en/exxon/exxongas/octane-rating). The [Beijing market regulator explains the RON basis of Chinese 92/95/98 grades](https://scjgj.beijing.gov.cn/zwxx/gs/cpzlgs/201909/t20190904_267505.html). The options are common examples, not an exhaustive list of regional products.

**Grade is recorded metadata in this model.** It does not alter lower heating value, spark-to-SOC delay, prescribed burn duration, pressure or power. Predicting a grade-dependent result requires a knock model and engine-specific control behavior, neither of which is implemented. A higher octane number is not an automatic power or energy-content multiplier. Current and pinned configurations store grades independently; session JSON and named preferences preserve them.

## Fuel-property references

Use the constants in `native/EngineModel.swift` as the exact values used by the native app (`model.mjs` contains the earlier browser model). The following references establish approximate scale and definitions, not a certificate for the presets:

| Reference fuel or blend | Published reference value | Basis |
| --- | ---: | --- |
| Typical No. 2 diesel | 42.6 MJ/kg | Lower heating value; NREL guide |
| Typical B100 biodiesel | 37.8 MJ/kg | Lower heating value; NREL guide |
| Gasoline | 42.7 MJ/kg; stoichiometric air/fuel ≈14.7 | ORNL report, Table 4.1 |
| Anhydrous ethanol | 26.8 MJ/kg; stoichiometric air/fuel ≈9.0 | ORNL report, Table 4.1 |

[NREL, Biodiesel Handling and Use Guide, Sixth Edition](https://afdc.energy.gov/uploads/publication/biodiesel_handling_use_guide.pdf), printed pages 12–13, supplies the mass-based diesel and biodiesel energy values. It also explains B20 as a volume blend and distinguishes biodiesel from renewable diesel.

[Oak Ridge National Laboratory, ORNL/TM-2007/37](https://digital.library.unt.edu/ark:/67531/metadc897317/m2/1/high_res_d/936789.pdf), printed page 15, Table 4.1, provides the gasoline and ethanol property comparison. Commercial gasoline composition varies, so its values are representative.

[DOE AFDC Fuel Properties Comparison](https://afdc.energy.gov/fuels/properties) provides volume-based LHV values: diesel 128,488 Btu/US gal, B100 119,550, B20 126,700, and ethanol/E100 76,330. Its gasoline column combines gasoline and E10; it should not be treated as an assay of a single pure gasoline. Do not compare MJ/kg directly with MJ/L or Btu/gal without density and unit conversion. For a nominal volume blend, convert component volumes to masses before mass-weighting LHV and stoichiometric air/fuel ratio.

[DOE AFDC E85](https://afdc.energy.gov/fuels/ethanol-e85) explains that commercial U.S. E85 contains 51–83% ethanol depending on season and location. Any fixed E85-like preset is one representative composition, not a model of every pump's fuel.

## Work, torque, and pressure ratios

Indicated work is the signed area inside the pressure–volume loop, `W = ∮ p dV`. At `rpm` revolutions per minute, each four-stroke cylinder has `rpm/120` cycles per second, so total indicated power is `n × W × rpm/120`. Brake estimates additionally use the app's simplified mechanical-loss assumptions. Torque follows average power divided by angular speed. The single-cylinder gas-torque trace is relative to an assumed 1 bar crankcase pressure and omits reciprocating inertia and mechanical friction.

Peak pressure alone does not determine power: high pressure before TDC increases compression work; very late combustion leaves less expansion available. [SAE's combustion-phasing analysis](https://saemobilus.sae.org/papers/investigation-fuel-engine-parameters-affect-half-mass-fraction-burned-ca50-optimum-crank-angle-2012-36-0498) discusses this tradeoff. The app is not a universal optimum-timing finder.

A pressure ratio must name its denominator and crank-angle reference. This app reports **fired peak pressure divided by motored peak pressure**; the two peaks need not occur at the same angle. It is different from the volume-based compression ratio. A motored pressure trace is the model's no-combustion reference under the same geometry and boundary assumptions. SOC is absent when no combustion heat is released; the internal scheduled-SOC value only describes the prospective event.

## Scope and interpretation

### Gearbox and road-speed display

The native app adds an engaged-transmission, steady-state calculation downstream of the combustion model. Single-clutch and dual-clutch configurations use the selected entry from an editable forward-gear ratio list. CVT uses a manually specified continuous ratio. Volkswagen describes its dual-clutch layout as two gear-train halves, each with a clutch ([Volkswagen](https://www.volkswagen-newsroom.com/en/the-new-golf-r-estate-media-drive-7510/the-drive-train-7512)); Toyota describes the variable-ratio pulley mechanism in its CVT ([Toyota](https://global.toyota/en/mobility/tnga/powertrain2018/cvt/)). Those architecture labels do not add shift transients to this model.

For gearbox ratio `g`, final drive `f`, engine speed `N` in rpm and tyre rolling diameter `D` in metres:

```
total ratio = g × f
wheel rpm = N / (g × f)
road speed (km/h) = wheel rpm × π × D × 60 / 1000
```

At positive engine torque, combined wheel torque is `engine torque × g × f × efficiency`. Negative engine torque represents a motoring demand, so the model divides by efficiency before multiplying by ratio to account for the wheel input needed to overcome transmission loss. Efficiency affects wheel torque, not the imposed RPM-to-speed relationship or calculated engine brake output.

Clutch slip, tyre slip, launch, shift duration, automatic CVT control, vehicle mass, drag and rolling resistance are omitted. The displayed speed is a kinematic operating point and does not establish an achievable top speed. Single- and dual-clutch configurations give identical steady results at identical ratios and efficiency. Power/torque gauges show engine brake output; wheel torque is shown in the Gearbox panel. Metric/US unit changes are display conversions only. Gauge scales adapt to the readings and are not redlines or mechanical limits.

### Combustion-model limits

- Read trends within the model; do not present estimated power or pressure as verified dynamometer or pressure-transducer measurements.
- This is four-stroke operation; the stroke control changes piston travel in millimeters, not two-stroke versus four-stroke architecture.
- The load/heat-input control is a simplified fueling input, not a complete throttle, governor, turbocharger, or ECU model.
- Gas exchange, residual gases, injection hardware, detailed temperature-dependent gas chemistry, multiple injections, cylinder-to-cylinder differences, and mechanical friction are simplified or omitted.
- The model does not predict knock, preignition, misfire, combustion stability, emissions, aftertreatment, durability, or a manufacturer's safe operating limits.
- Matching a target pressure does not validate the chosen operating point. Similarly, an apparently favorable high-compression gasoline result does not imply knock-free operation.
- The cylinder-bank animation uses an illustrative evenly spaced sequence. It does not identify a manufacturer's crankshaft layout or firing order.
