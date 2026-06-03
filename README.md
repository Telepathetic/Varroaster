# Varroaster

**Open-source closed-loop hyperthermia treatment for Varroa destructor.**

Chemical-free. Retrofittable. Dead simple.

Despite the name, the goal is controlled therapeutic hyperthermia — not overheating.

-----

## The Problem

Varroa destructor is the primary threat to managed honeybee colonies. Sustained hyperthermia around 41°C has been shown in multiple studies to substantially reduce Varroa populations while remaining within the tolerance range of bees and brood under controlled conditions. Real-world outcomes vary with hive configuration and thermal control.

Existing solutions aren’t good enough:

- **Solar thermotherapy** — weather dependent, requires babysitting, useless in the Pacific Northwest
- **Commercial hyperthermia units** — $500–1000+, proprietary boxes, not retrofittable
- **Chemical treatments** — effective but increasing resistance pressure and not everyone wants residues

Nobody has shipped a cheap, open-source, weather-independent, retrofittable closed-loop hyperthermia unit. This is that.

-----

## Why Hyperthermia?

- No chemical residues in wax or honey
- No resistance pressure on mite populations
- Weather independent — works on any day, any season
- Targets the brood-safe window when properly controlled
- Compatible with integrated Varroa management approaches

**Why not oxalic acid vaporization?**
OAV is effective and we respect it. Hyperthermia is complementary, not a replacement — particularly useful for beekeepers who want a residue-free option or are managing resistant mite populations.

-----

## How It Works

The Varroaster mounts directly to the base port of the hive. Unlike external warming boxes, heat enters directly at the hive base with essentially zero transport loss, allowing lower heater power and battery operation. Return air is pulled from the roof back to the unit via a flexible hose.

```
[VARROASTER] → hot air → [HIVE BASE PORT]
     ↑                          ↓
     └── return air ── [HIVE ROOF PORT]
  (base-mounted, direct inject)
```

Heat rises naturally through the colony. The hive’s thermal mass slowly soaks up to temperature over the treatment window. Return air arrives cool from the roof hose — this is expected and used diagnostically. On cold days the return hose passively dumps excess humidity as condensate; a low point drain prevents blockage.

Closed-loop recirculation reduces the colony’s ability to reject heat through normal ventilation behaviors, since their primary cooling mechanism assumes an open entrance. Recirculating hive air also maintains humidity and may contribute to mite mortality, though this has not been independently characterized.

**Why 41°C?**
Published hyperthermia studies have reported near-complete mortality of reproducing mites and partial mortality of phoretic adults around this temperature and duration, within the documented tolerance range of bees and brood. Outcomes vary by methodology and colony conditions. Pushing higher risks brood damage. 41°C is the conservative starting target derived from published studies — subject to validation across colony configurations. See [Trial Instrumentation](#trial-instrumentation).

Repeat in 12 days to catch the next emerged wave.

-----

## Treatment Protocol

Place Varroaster on hive and close hive entrance door. Connect 12v battery or 12v adapter. Press Start. Any power fault automatically reopens the entrance. Upon normal completion the entrance opens, a notification is sent, and the fan idles. Remove Varroaster at your convenience.

Repeat in 12 days.

> **Note for trial operators:** The 41°C / 3h parameters are the current best-estimate targets derived from published hyperthermia literature. They are not yet validated across diverse colony configurations. See [Trial Instrumentation](#trial-instrumentation) for the sensor requirements and data collection protocol that will refine these parameters over time.

-----

## Hardware

### Varroaster Unit

One unit treats any compatible hive. Mounts directly to the base port.

**BOM (~$45–65):**

|Component                   |Notes                                                                                                                |~Cost   |
|----------------------------|---------------------------------------------------------------------------------------------------------------------|--------|
|ESP32 dev board             |Main controller                                                                                                      |$3–5    |
|HS50 4R F resistor          |50W chassis mount, 4Ω — 36W at 12v. Two in parallel for larger hives                                                 |$3–6    |
|Aluminum heatsink plate     |Resistor bolts flat, fins extend into airstream parallel to flow                                                     |$3–5    |
|N-channel MOSFET            |PWM switching of resistor                                                                                            |$1–2    |
|SSR or safety relay         |Secondary hardware cutoff independent of MOSFET                                                                      |$3–5    |
|High static pressure blower |5015 or 9733 dual ball bearing — hive comb is a dense labyrinth                                                      |$5–10   |
|DS18B20 sensors ×4          |S0: inside unit (watchdog) · S1: base port output · S2: roof port return · S3: ambient (isolated from heater airflow)|$4–6    |
|Bimetallic thermal fuse     |Passive ~55°C cutoff inline with resistor                                                                            |$3–5    |
|Fail-safe entrance solenoid |Powered-closed, springs open on power loss                                                                           |$4–6    |
|Small buck/boost module     |Battery → 12v logic rail (low current only)                                                                          |$2–3    |
|3.3v LDO                    |12v → ESP32                                                                                                          |$1–2    |
|SSD1306 OLED                |S1 / S2 / elapsed time                                                                                               |$2–3    |
|Printed enclosure + fittings|See `/hardware`                                                                                                      |filament|

### Hive Ports

Two ported fittings — base port mounts directly to the Varroaster unit, roof port connects via flexible return hose. See `/hardware` for STLs and retrofitting guidance.

-----

## Trial Instrumentation

S1 and S2 measure supply and return air temperatures. They do not measure brood or comb temperature directly. The relationship between S2 (return air) and actual brood center temperature is the load-bearing unknown of this project — it varies with colony strength, hive geometry, insulation, ventilation configuration, and season.

**This relationship must be characterized before the treatment protocol can be considered validated.**

### Required for v0.1 / v0.2 Field Trials

All operators running early trial hardware should instrument their hives with additional internal probes during treatment. At minimum:

- **Brood center probe** — DS18B20 on a thin flexible cable, clipped between active brood frames at mid-frame depth
- **Brood edge probe** — same, at the outer brood frame

Recommended additions for thorough characterization:

- **Honey frame probe** — adjacent honey storage frame
- **Upper box probe** — if running a double brood box

These probes do not feed the control loop. They provide ground truth on actual brood temperature versus air temperature across different hive configurations — the dataset that will either validate or refine the S2 proxy and the 41°C target.

**Log everything.** The firmware exports S0/S1/S2 continuously; add your internal probe readings to the same log. See `/firmware/README.md` for logging configuration.

### Companion Hardware: Hive Thermal Mapper

The **[Hive Thermal Mapper](https://github.com/Telepathetic/Hive-Thermal-Mapper)** is a natural companion instrument for Varroaster trial stages. It provides dense multi-point brood-frame temperature data (up to 75 sensors per frame across a full 10-frame Langstroth) via a distributed RS-485 sensor network — exactly the spatial resolution needed to characterize the S2 ↔ brood interior transfer function across colony configurations.

Running a Thermal Mapper alongside Varroaster during treatment cycles will generate the empirical dataset that makes Varroaster’s treatment claims defensible. Both projects are open-source under compatible licenses. Field data from combined deployments is especially welcome.

### Graduating Out of Trial Instrumentation

Once the S2 ↔ brood center transfer function has been characterized across a sufficient range of colony configurations (target: at minimum weak/strong colonies, screened/solid bottom boards, single/double brood boxes, insulated/uninsulated hives), the internal probe requirement can be retired for standard deployments. That dataset will also inform whether the 41°C / 3h parameters need per-configuration adjustment.

Until then: **instrument everything.**

-----

## Power Architecture

```
12v battery
  ├─→ SSR → Bimetallic fuse → Resistor + MOSFET  (high current, PWM)
  └─→ Buck/boost → 12v → Fan + LDO → ESP32 + sensors + OLED + solenoid
```

Resistor runs direct from battery — no high-current buck/boost needed. **Size for 11v** (depleted battery floor) to ensure treatment completes at end of charge. SSR provides a second hardware cutoff fully independent of the MOSFET.

**Recommended power source:** 12v motorcycle battery. Estimated energy budget is 150–250Wh for a typical treatment including warmup and blower overhead. Car battery for cold days or very large hives. NaIon packs at 12v nominal work natively.

-----

## Firmware

Mostly open-loop with safety intelligence. Hives are high thermal-mass systems with long time constants — fixed characterized output with safety ceilings proved simpler and less prone to oscillation than aggressive closed-loop control. A fixed PWM duty cycle, characterized once on the bench for your blower and heater geometry, holds output air near 42.5°C. S1 acts as a hard ceiling; dS2/dt provides early warning if the hive is responding unexpectedly. S3 (ambient) is logged continuously and included in all treatment records — makes cross-colony and cross-season comparison meaningful.

**State machine:**

- **HEATING** — fixed PWM, S1 ceiling at 42.5°C, waiting for S2 to cross 40°C and hold
- **TREATMENT_SOAK** — same PWM, 3 hour timer running, diagnostics active
- **COMPLETE** — entrance solenoid releases, notification sent, fan idles
- **FAULT** — any diagnostic trip: hard shutdown, alert with fault code

There is no ramp phase. The unit turns on at soak duty cycle immediately and waits for the hive to come up to temperature.

**Diagnostics (runs every 5 seconds):**

- **Airflow blocked** — S0 climbs above 55°C while S2 is stagnant or dropping: hard shutdown, alert “Check hose for blockage or water”
- **Circuit open** — S1 hot but S2 doesn’t rise within 15 minutes of start: shutdown, alert “Check hose connections”
- **Hive not reaching temperature** — S1 stable, S2 persistently below 38°C after 45 minutes: warning alert “Hive may be too large, too cold, or leaking — check conditions” (treatment continues)
- **Overtemp** — S1 exceeds 43°C: shutdown, alert
- **dS2/dt too steep** — S2 rising faster than expected: throttle PWM, log event

**S1−S2 delta** is logged continuously and is a useful secondary diagnostic. A large delta indicates poor circulation, a disconnected or collapsed hose, or a stalled blower. A shrinking delta over time indicates the hive approaching thermal equilibrium. Sudden delta changes mid-treatment are worth flagging in logs.

**Other:**

- OLED: S1 / S2 / elapsed time — no phone needed in the bee yard
- WiFi web UI for configuration and treatment logs
- Push notification on completion or fault

See `/firmware/README.md` for build and flash instructions.

-----

## Honest Caveats

- **Air temp ≠ brood temp** — S1 and S2 measure air, not comb or brood interior. The relationship between return air temperature and actual brood temperature is not characterized across hive configurations and must be established before the treatment protocol can be considered validated. This is the primary open question of the project. See [Trial Instrumentation](#trial-instrumentation)
- **41°C / 3h is a starting hypothesis** — derived from published hyperthermia literature, not yet validated across the range of colony configurations this device will encounter in the field
- **Queen tolerance** — queen survival under a single treatment is within documented thermal limits, but tolerance under repeated treatment cycles has not been independently characterized in this system
- **Stacked stressors** — this system combines heat, humidity, and restricted ventilation simultaneously. The combined effect on bee safety margin has not been independently studied; treat early versions with appropriate caution
- **Fixed PWM varies by conditions** — ambient temperature, colony size, hive insulation, and geometry all affect the characterized duty cycle. What works on one hive may need adjustment on another
- **Entrance mechanism durability** — the fail-open solenoid design is sound, but the hive environment is hostile to moving parts. Propolis accumulation, wax contamination, dead bee jams, and winter icing around the entrance mechanism are all real failure modes to characterize during field trials
- **Cold days stress the system** — treatment not recommended below ~5°C ambient
- **Not a silver bullet** — most effective as part of an integrated Varroa management approach
- **Pre-prototype** — contributions and field reports very welcome

-----

## Long-Term Vision

Varroaster and the [Hive Thermal Mapper](https://github.com/Telepathetic/Hive-Thermal-Mapper) are converging projects. The architecture they’re building toward:

```
Hive Thermal Mapper (750-point brood field)
        +
Varroaster (heat source + control)
        =
Target: brood-field temperature, not air temperature
```

At that point the treatment question shifts from:

*Did return air hold 40°C for 3 hours?*

to:

*What percentage of brood comb exceeded 40°C? For how long? Were there cold spots? Did any region exceed safety limits?*

Those are questions existing hyperthermia systems generally cannot answer. The mapper makes them answerable. Near-term the mapper is a validation instrument; long-term it becomes the control instrument.

-----

## License

Hardware: **CERN OHL-S v2** — Docs: **CC BY-SA 4.0** — Firmware: **MIT**
