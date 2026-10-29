# Engine Lab release notes

## 1.1 — 2026-10-29

Native app version **1.1**, build **2**. The changes below compare this release with the retained version 1.0 app, build 1.

### Added

- An instrument-cluster display with four gauges: estimated engine power, estimated engine torque, RPM, and road speed. The digital display remains available.
- Metric and US output units: kW/hp, N·m/lb-ft, and km/h/mph.
- Single-clutch, dual-clutch, and CVT gearbox options.
- One to eight forward gears with editable ratios and active-gear selection for single- and dual-clutch setups.
- A continuous ratio control for CVT, plus final drive, tyre rolling diameter, and transmission efficiency controls for all gearbox types.
- Calculated road speed and wheel torque. Manual gear ratios are retained when switching to CVT and back.

### Changed

- Added a Settings menu under the gear icon and placed display style, the existing language selector, and output units together.
- Made Gearbox sidebar section **04** and moved Cooling & pressure to section **05**.
- Extended autosave, named preferences, pinned comparisons, and exported JSON sessions to include gearbox, display style, and output-unit settings.
- Added defaults when loading older files without the new fields. Existing language and engine settings are retained.
- Made the About window read the app version from its metadata, keeping the displayed version consistent with the built app.
- Updated the GitHub README with a native-app screenshot, build instructions, controls, saving/exporting guidance, and model limitations.

### Features carried over from 1.0

The retained 1.0 build already includes the native engine simulator, fuel and combustion controls, charts and animation, named preferences and autosave, JSON/CSV export, English and Simplified Chinese, and petrol-grade selection. These are continuing features, not new additions in 1.1.

### Keeping both versions

You can retain separate app bundles named **Engine Lab V1.0.app** and **Engine Lab V1.1.app**. Renaming the app bundle does not change its internal version; use **About Engine Lab** to check which version is open.

Both versions use the same application identifier and local preference storage. Separate app names therefore do not create independent save libraries. Use one version at a time and export important setups as JSON before switching back to 1.0. The older app does not retain the new gearbox, display-style, and output-unit fields when it saves a session.

Version 1.1 supports older session files. The JSON session format still uses its existing schema version; that number is separate from the app's 1.1 release number.

### Scope of this update

- Main power and torque readings remain estimates of engine output. Wheel torque is a separate drivetrain result.
- Road speed is calculated from RPM and gearing with no slip. This release does not add acceleration, road-load, shift-time, clutch-slip, or automatic CVT-control simulation.
- Single- and dual-clutch types have identical steady results at the same ratios and efficiency.
- Geometry and combustion inputs remain in mm, L, bar, and °C. Output-unit selection changes power, torque, and speed displays.
- Petrol grades remain saved metadata; changing octane grade alone does not change simulated power or combustion timing.

See the [README](README.md) for usage and [MODEL.md](MODEL.md) for the model assumptions.
